---
author : "杨承龙"
date : "2017-06-04T21:47:04+08:00"
draft : false
title : "pool"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---

sync.Pool

当多个goroutine都需要创建同一个对象的时候，如果goroutine过多，可能导致对象的创建数目剧增。 而对象又是占用内存的，进而导致的就是内存回收的GC压力徒增。造成“并发大－占用内存大－GC缓慢－处理并发能力降低－并发更大”这样的恶性循环。但如果每个goroutine不再自己单独创建对象，而是从对象池中获取出一个对象（如果池中已经有的话）。 这就是sync.Pool出现的目的了。

sync.Pool的使用非常简单，提供两个方法:Get和Put 和一个初始化回调函数New。

看下面这个例子(取自gomemcache)：

    // keyBufPool returns []byte buffers for use by PickServer's call to
    // crc32.ChecksumIEEE to avoid allocations. (but doesn't avoid the
    // copies, which at least are bounded in size and small)
    var keyBufPool = sync.Pool{
    	New: func() interface{} {
    		b := make([]byte, 256)
    		return &b
    	},
    }
    
    func (ss *ServerList) PickServer(key string) (net.Addr, error) {
    	ss.mu.RLock()
    	defer ss.mu.RUnlock()
    	if len(ss.addrs) == 0 {
    		return nil, ErrNoServers
    	}
    	if len(ss.addrs) == 1 {
    		return ss.addrs[0], nil
    	}
    	bufp := keyBufPool.Get().(*[]byte)
    	n := copy(*bufp, key)
    	cs := crc32.ChecksumIEEE((*bufp)[:n])
    	keyBufPool.Put(bufp)
    
    	return ss.addrs[cs%uint32(len(ss.addrs))], nil
    }

这是实际项目中的一个例子，这里使用keyBufPool的目的是为了让crc32.ChecksumIEEE所使用的[]bytes数组可以重复使用，减少GC的压力。

但是这里可能会有一个问题，我们没有看到Pool的手动回收函数。 那么是不是就意味着，如果我们的并发量不断增加，这个Pool的体积会不断变大，或者一直维持在很大的范围内呢？

答案是不会的，sync.Pool的回收是有的，它是在系统自动GC的时候，触发pool.go中的poolCleanup函数。

    func poolCleanup() {
    	for i, p := range allPools {
    		allPools[i] = nil
    		for i := 0; i < int(p.localSize); i++ {
    			l := indexLocal(p.local, i)
    			l.private = nil
    			for j := range l.shared {
    				l.shared[j] = nil
    			}
    			l.shared = nil
    		}
    		p.local = nil
    		p.localSize = 0
    	}
    	allPools = []*Pool{}
    }

这个函数会把Pool中所有goroutine创建的对象都进行销毁。

那这里另外一个问题也凸显出来了，很可能我上一步刚往pool中PUT一个对象之后，下一步GC触发，导致pool的GET函数获取不到PUT进去的对象。 这个时候，GET函数就会调用New函数，临时创建出一个对象，并存放到pool中。

根据以上结论，sync.Pool其实不适合用来做持久保存的对象池（比如连接池）。它更适合用来做临时对象池，目的是为了降低GC的压力。

连接池性能测试

    package memcache
    
    import (
    	"sync"
    	"testing"
    )
    
    var bytePool = sync.Pool{
    	New: newPool,
    }
    
    func newPool() interface{} {
    	b := make([]byte, 1024)
    	return &b
    }
    func BenchmarkAlloc(b *testing.B) {
    	for i := 0; i < b.N; i++ {
    		obj := make([]byte, 1024)
    		_ = obj
    	}
    }
    
    func BenchmarkPool(b *testing.B) {
    	for i := 0; i < b.N; i++ {
    		obj := bytePool.Get().(*[]byte)
    		_ = obj
    		bytePool.Put(obj)
    	}
    }

文件目录下执行 go test -bench .

    bogon:memcache yang$ go test -bench .
    BenchmarkOnItem-4              	 2000000	       902 ns/op
    BenchmarkPickServer-4          	10000000	       147 ns/op	       0 B/op	       0 allocs/op
    BenchmarkPickServer_Single-4   	20000000	        69.4 ns/op	       0 B/op	       0 allocs/op
    BenchmarkAlloc-4               	50000000	        40.8 ns/op
    BenchmarkPool-4                	50000000	        27.8 ns/op
    PASS
    ok  	gomemcache/memcache	9.318s

通过性能测试可以清楚地看到，使用连接池消耗的CPU时间远远小于每次手动分配内存。



源码分析：

    type Pool struct {
    	noCopy noCopy
    
    	local     unsafe.Pointer
    	localSize uintptr
        //当pool中无可用对象时，调用New函数产生对象值直接返回给调用方，所以其产生的对象值永远不会被放置到池中
    	New func() interface{}
    }
    
    //sync.pool为每个P（golang的调度模型介绍中有介绍）都分配了一个子池。
    //每个子池里面有一个私有对象和共享列表对象，私有对象是只有对应的P能够访问，
    //因为一个P同一时间只能执行一个goroutine，因此对私有对象存取操作是不需要加锁的。
    //共享列表是和其他P分享的，因此操作共享列表是需要加锁的。
    type poolLocal struct {
    	private interface{}
    	shared  []interface{}
    	Mutex
    	pad     [128]byte
    }
    
    
    func fastrand() uint32
    
    var poolRaceHash [128]uint64
    
    func poolRaceAddr(x interface{}) unsafe.Pointer {
    	ptr := uintptr((*[2]unsafe.Pointer)(unsafe.Pointer(&x))[1])
    	h := uint32((uint64(uint32(ptr)) * 0x85ebca6b) >> 16)
    	return unsafe.Pointer(&poolRaceHash[h%uint32(len(poolRaceHash))])
    }
    
    
    //固定到某个P，如果私有对象为空则放到私有对象；
    //否则加入到该P子池的共享列表中（需要加锁）。
    //可以看到一次put操作最少0次加锁，最多1次加锁。
    func (p *Pool) Put(x interface{}) {
    	if x == nil {
    		return
    	}
    	if race.Enabled {
    		if fastrand()%4 == 0 {
    			return
    		}
    		race.ReleaseMerge(poolRaceAddr(x))
    		race.Disable()
    	}
    	l := p.pin()
    	if l.private == nil {
    		l.private = x
    		x = nil
    	}
    	runtime_procUnpin()
    	if x != nil {
    		l.Lock()
    		l.shared = append(l.shared, x)
    		l.Unlock()
    	}
    	if race.Enabled {
    		race.Enable()
    	}
    }
    
    //固定到某个P，尝试从私有对象获取，如果私有对象非空则返回该对象，并把私有对象置空；
    //如果私有对象是空的时候，就去当前子池的共享列表获取（需要加锁）；
    //如果当前子池的共享列表也是空的，那么就尝试去其他P的子池的共享列表偷取一个（需要加锁）；
    //如果其他子池都是空的，最后就用用户指定的New函数产生一个新的对象返回。
    //可以看到一次get操作最少0次加锁，最大N（N等于MAXPROCS）次加锁。
    func (p *Pool) Get() interface{} {
    	if race.Enabled {
    		race.Disable()
    	}
    	l := p.pin()
    	x := l.private
    	l.private = nil
    	runtime_procUnpin()
    	if x == nil {
    		l.Lock()
    		last := len(l.shared) - 1
    		if last >= 0 {
    			x = l.shared[last]
    			l.shared = l.shared[:last]
    		}
    		l.Unlock()
    		if x == nil {
    			x = p.getSlow()
    		}
    	}
    	if race.Enabled {
    		race.Enable()
    		if x != nil {
    			race.Acquire(poolRaceAddr(x))
    		}
    	}
    	if x == nil && p.New != nil {
    		x = p.New()
    	}
    	return x
    }
    
    func (p *Pool) getSlow() (x interface{}) {
    	size := atomic.LoadUintptr(&p.localSize)
    	local := p.local
    
    	pid := runtime_procPin()
    	runtime_procUnpin()
    	for i := 0; i < int(size); i++ {
    		l := indexLocal(local, (pid+i+1)%int(size))
    		l.Lock()
    		last := len(l.shared) - 1
    		if last >= 0 {
    			x = l.shared[last]
    			l.shared = l.shared[:last]
    			l.Unlock()
    			break
    		}
    		l.Unlock()
    	}
    	return x
    }
    
    func (p *Pool) pin() *poolLocal {
    	pid := runtime_procPin()
    
    	s := atomic.LoadUintptr(&p.localSize)
    	l := p.local
    	if uintptr(pid) < s {
    		return indexLocal(l, pid)
    	}
    	return p.pinSlow()
    }
    
    func (p *Pool) pinSlow() *poolLocal {
    	runtime_procUnpin()
    	allPoolsMu.Lock()
    	defer allPoolsMu.Unlock()
    	pid := runtime_procPin()
    
    	s := p.localSize
    	l := p.local
    	if uintptr(pid) < s {
    		return indexLocal(l, pid)
    	}
    	if p.local == nil {
    		allPools = append(allPools, p)
    	}
    
    	size := runtime.GOMAXPROCS(0)
    	local := make([]poolLocal, size)
    	atomic.StorePointer(&p.local, unsafe.Pointer(&local[0]))
    	atomic.StoreUintptr(&p.localSize, uintptr(size))
    	return &local[pid]
    }
    
    //在系统自动GC的时候，触发pool.go中的poolCleanup函数,把Pool中所有goroutine创建的对象都进行销毁
    func poolCleanup() {
    	for i, p := range allPools {
    		allPools[i] = nil
    		for i := 0; i < int(p.localSize); i++ {
    			l := indexLocal(p.local, i)
    			l.private = nil
    			for j := range l.shared {
    				l.shared[j] = nil
    			}
    			l.shared = nil
    		}
    		p.local = nil
    		p.localSize = 0
    	}
    	allPools = []*Pool{}
    }
    
    var (
    	allPoolsMu Mutex
    	allPools   []*Pool
    )
    
    //在init中注册了一个poolCleanup函数，
    //它会清除所有的pool里面的所有缓存的对象，该函数注册进去之后会在每次gc之前都会调用
    func init() {
    	runtime_registerPoolCleanup(poolCleanup)
    }
    
    func indexLocal(l unsafe.Pointer, i int) *poolLocal {
    	return &(*[1000000]poolLocal)(l)[i]
    }
    
    // Implemented in runtime.
    func runtime_registerPoolCleanup(cleanup func())
    func runtime_procPin() int
    func runtime_procUnpin()



示例：

    package main
    
    import (
    	"fmt"
    	"runtime"
    	"runtime/debug"
    	"sync"
    	"sync/atomic"
    )
    
    func main() {
    	// 禁用GC，并保证在main函数执行结束前恢复GC
    	defer debug.SetGCPercent(debug.SetGCPercent(-1))
    	var count int32
    	newFunc := func() interface{} {
    		return atomic.AddInt32(&count, 1)
    	}
    
    	pool := sync.Pool{New: newFunc}
    
    	// New 字段值的作用
    	v1 := pool.Get()
    	fmt.Printf("v1: %v\n", v1)
    
    	// 临时对象池的存取
    	pool.Put(newFunc())
    	pool.Put(newFunc())
    	pool.Put(newFunc())
    	v2 := pool.Get()
    	fmt.Printf("v2: %v\n", v2)
    
    	// 垃圾回收对临时对象池的影响
    	debug.SetGCPercent(100)
    	runtime.GC()
    	v3 := pool.Get()
    	fmt.Printf("v3: %v\n", v3)
    	pool.New = nil
    	v4 := pool.Get()
    	fmt.Printf("v4: %v\n", v4)
    }


