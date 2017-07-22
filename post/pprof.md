---
author : "杨承龙"
date : "2017-06-19T13:20:13+08:00"
draft : false
title : "pprof"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---
net/http/pprof

web 服务器

如果你的go程序是用http包启动的web服务器，你想查看自己的web服务器的状态。这个时候就可以选择net/http/pprof。你只需要引入包_"net/http/pprof"，然后就可以在浏览器中使用http://host:port/debug/pprof/ 直接看到当前web服务的状态，包括CPU占用情况和内存使用情况等。(亲测有效)

服务进程

如果你的go程序不是web服务器，而是一个服务进程，那么你也可以选择使用net/http/pprof包，同样引入包net/http/pprof，然后在开启另外一个goroutine来开启端口监听。

    import (
    	_"net/http/pprof"
    	"log"
    	"net/http"
    )
    go func() {
            log.Println(http.ListenAndServe("localhost:6060", nil)) 
    }()

其中log和net/http是需要额外倒入使goroutine能够正常运行而导入的，localhost:6060是你选择观察其使用情况的地址，在服务运行后，在浏览器输入http://localhost:6060/debug/pprof/ 即可浏览当前服务的状态。页面信息如下：

    /debug/pprof/
    
    profiles:
    0	block
    15	goroutine
    6	heap
    0	mutex
    10	threadcreate
    
    full goroutine stack dump

  名称          	说明                	取样频率             
  go routine  	活跃Goroutine的信息的记录。	仅在获取时取样一次。       
  threadcreate	系统线程创建情况的记录。      	仅在获取时取样一次。       
  heap        	堆内存分配情况的记录。       	默认每分配512K字节时取样一次。
  block       	Goroutine阻塞事件的记录。 	默认每发生一次阻塞事件时取样一次。

block显示

    --- contention:
    cycles/second=1599092497

goroutine显示

    goroutine profile: total 12
    1 @ 0x101010b 0x104207f 0x16093f2 0x105a7a1
    #	0x104207e	os/signal.signal_recv+0xfe	/usr/local/go/src/runtime/sigqueue.go:116
    #	0x16093f1	os/signal.loop+0x21		/usr/local/go/src/os/signal/signal_unix.go:22
    
    1 @ 0x102e05a 0x1028f97 0x10285d9 0x118f098 0x118f104 0x11908c7 0x11a2bc0 0x1295c78 0x105a7a1
    #	0x10285d8	net.runtime_pollWait+0x58			/usr/local/go/src/runtime/netpoll.go:164
    #	0x118f097	net.(*pollDesc).wait+0x37			/usr/local/go/src/net/fd_poll_runtime.go:75
    #	0x118f103	net.(*pollDesc).waitRead+0x33			/usr/local/go/src/net/fd_poll_runtime.go:80
    #	0x11908c6	net.(*netFD).Read+0x1b6				/usr/local/go/src/net/fd_unix.go:250
    #	0x11a2bbf	net.(*conn).Read+0x6f				/usr/local/go/src/net/net.go:181
    #	0x1295c77	net/http.(*connReader).backgroundRead+0x57	/usr/local/go/src/net/http/server.go:656

heap显示

    heap profile: 4: 1792704 [6: 1801536] @ heap/1048576
    1: 1703936 [1: 1703936] @ 0x1009eb6 0x1475f17 0x1542564 0x1542e50 0x15cd9ec 0x15ddbae 0x160a521 0x1658814 0x102dbca 0x105a7a1
    #	0x1475f16	github.com/henrylee2cn/pholcus/common/pinyin.init+0x296	/Users/yang/lib/go/src/github.com/henrylee2cn/pholcus/common/pinyin/pinyin_dict.go:12
    #	0x1542563	github.com/henrylee2cn/pholcus/app/spider.init+0xb3	/Users/yang/lib/go/src/github.com/henrylee2cn/pholcus/app/spider/timer.go:166
    #	0x1542e4f	github.com/henrylee2cn/pholcus/app/downloader.init+0x4f	/Users/yang/lib/go/src/github.com/henrylee2cn/pholcus/app/downloader/downloader_surfer.go:45
    #	0x15cd9eb	github.com/henrylee2cn/pholcus/app/crawler.init+0x5b	/Users/yang/lib/go/src/github.com/henrylee2cn/pholcus/app/crawler/spiderqueue.go:105
    #	0x15ddbad	github.com/henrylee2cn/pholcus/app.init+0x6d		/Users/yang/lib/go/src/github.com/henrylee2cn/pholcus/app/app.go:683
    #	0x160a520	github.com/henrylee2cn/pholcus/exec.init+0x60		/Users/yang/lib/go/src/github.com/henrylee2cn/pholcus/exec/exec_darwin.go:31
    #	0x1658813	main.init+0x43						/Users/yang/lib/go/src/pholcus/example_main.go:20
    #	0x102dbc9	runtime.main+0x1c9					/usr/local/go/src/runtime/proc.go:173

mutex显示

    --- mutex:
    cycles/second=1599092497
    sampling period=0

threadcreate显示

    threadcreate profile: total 11
    11 @
    #	0x0

full goroutine stack dump显示

    goroutine 158 [running]:
    runtime/pprof.writeGoroutineStacks(0x1cbc600, 0xc4205522a0, 0x30, 0xc42060c7b0)
    	/usr/local/go/src/runtime/pprof/pprof.go:603 +0x79
    runtime/pprof.writeGoroutine(0x1cbc600, 0xc4205522a0, 0x2, 0xc420038a90, 0x1011ac8)
    	/usr/local/go/src/runtime/pprof/pprof.go:592 +0x44
    runtime/pprof.(*Profile).WriteTo(0x1e84440, 0x1cbc600, 0xc4205522a0, 0x2, 0xc4205522a0, 0xc420038cc0)
    	/usr/local/go/src/runtime/pprof/pprof.go:302 +0x3b5
    net/http/pprof.handler.ServeHTTP(0xc42060c611, 0x9, 0x1cc34c0, 0xc4205522a0, 0xc4203de800)
    	/usr/local/go/src/net/http/pprof/pprof.go:209 +0x1d1
    net/http/pprof.Index(0x1cc34c0, 0xc4205522a0, 0xc4203de800)
    	/usr/local/go/src/net/http/pprof/pprof.go:221 +0x1e3
    net/http.HandlerFunc.ServeHTTP(0x181e7e8, 0x1cc34c0, 0xc4205522a0, 0xc4203de800)
    	/usr/local/go/src/net/http/server.go:1942 +0x44
    net/http.(*ServeMux).ServeHTTP(0x1ed5160, 0x1cc34c0, 0xc4205522a0, 0xc4203de800)
    	/usr/local/go/src/net/http/server.go:2238 +0x130
    net/http.serverHandler.ServeHTTP(0xc420578000, 0x1cc34c0, 0xc4205522a0, 0xc4203de800)
    	/usr/local/go/src/net/http/server.go:2568 +0x92
    net/http.(*conn).serve(0xc4203d2140, 0x1cc3f40, 0xc420077200)
    	/usr/local/go/src/net/http/server.go:1825 +0x612
    created by net/http.(*Server).Serve
    	/usr/local/go/src/net/http/server.go:2668 +0x2ce

你也可以通过终端命令查看:

//查看堆信息

go tool pprof http://localhost:6060/debug/pprof/heap

    bogon:~ yang$ go tool pprof http://localhost:6060/debug/pprof/heap
    Fetching profile from http://localhost:6060/debug/pprof/heap
    Saved profile in /Users/yang/pprof/pprof.localhost:6060.alloc_objects.alloc_space.inuse_objects.inuse_space.016.pb.gz
    Entering interactive mode (type "help" for commands)
    (pprof) top 10
    4628.68kB of 4628.68kB total (  100%)
    Dropped 27 nodes (cum <= 23.14kB)
    Showing top 10 nodes out of 35 (cum >= 2064.68kB)
          flat  flat%   sum%        cum   cum%
     1027.94kB 22.21% 22.21%  1027.94kB 22.21%  runtime.hashGrow
     1024.08kB 22.12% 44.33%  1024.08kB 22.12%  text/template/parse.(*Tree).pipeline
     1024.03kB 22.12% 66.46%  1024.03kB 22.12%  vendor/golang_org/x/net/http2/hpack.addDecoderNode
      528.17kB 11.41% 77.87%   528.17kB 11.41%  regexp.(*bitState).reset
      512.44kB 11.07% 88.94%   512.44kB 11.07%  runtime.rawstringtmp
      512.02kB 11.06%   100%   512.02kB 11.06%  runtime.makemap
             0     0%   100%  3604.65kB 77.88%  github.com/astaxie/beego.BuildTemplate
             0     0%   100%  3604.65kB 77.88%  github.com/astaxie/beego.Run
             0     0%   100%  3604.65kB 77.88%  github.com/astaxie/beego.getTemplate
             0     0%   100%  2064.68kB 44.61%  github.com/astaxie/beego.getTplDeep
    (pprof) 

//查看30秒内CPU运行情况(是指在你敲下命令后30秒内所检测服务占用cpu情况)

go tool pprof http://localhost:6060/debug/pprof/profile

    bogon:~ yang$ go tool pprof http://localhost:6060/debug/pprof/profile
    Fetching profile from http://localhost:6060/debug/pprof/profile
    Please wait... (30s)
    Saved profile in /Users/yang/pprof/pprof.localhost:6060.samples.cpu.008.pb.gz
    Entering interactive mode (type "help" for commands)
    (pprof) top 10
    280ms of 300ms total (93.33%)
    Showing top 10 nodes out of 189 (cum >= 10ms)
          flat  flat%   sum%        cum   cum%
         100ms 33.33% 33.33%      100ms 33.33%  syscall.Syscall
          50ms 16.67% 50.00%       50ms 16.67%  runtime.mach_semaphore_signal
          30ms 10.00% 60.00%       30ms 10.00%  runtime.usleep
          20ms  6.67% 66.67%       20ms  6.67%  runtime.freedefer
          20ms  6.67% 73.33%       20ms  6.67%  runtime.mach_semaphore_wait
          20ms  6.67% 80.00%       20ms  6.67%  runtime.scanobject
          10ms  3.33% 83.33%       10ms  3.33%  reflect.(*rtype).Name
          10ms  3.33% 86.67%       10ms  3.33%  regexp.(*bitState).push
          10ms  3.33% 90.00%       10ms  3.33%  runtime.evacuate
          10ms  3.33% 93.33%       10ms  3.33%  runtime.kevent
    (pprof) 

//查看goroutine阻塞情况

go tool pprof http://localhost:6060/debug/pprof/block

    bogon:~ yang$ go tool pprof http://localhost:6060/debug/pprof/block
    Fetching profile from http://localhost:6060/debug/pprof/block
    Saved profile in /Users/yang/pprof/pprof.localhost:6060.contentions.delay.004.pb.gz
    Entering interactive mode (type "help" for commands)
    (pprof) top 10
    profile is empty
    (pprof) 

在上述例子中，我们是输入命令，然后显示(pprof) ，表示进入模式，然后选取前10条记录，我们也可以不使用top命令也可以获取到数据。即加上一个 --text参数，如下：

    bogon:~ yang$ go tool pprof --text http://localhost:6060/debug/pprof/heap
    Fetching profile from http://localhost:6060/debug/pprof/heap
    Saved profile in /Users/yang/pprof/pprof.localhost:6060.alloc_objects.alloc_space.inuse_objects.inuse_space.017.pb.gz
    5638.13kB of 5638.13kB total (  100%)
    Dropped 145 nodes (cum <= 28.19kB)
          flat  flat%   sum%        cum   cum%
     1027.94kB 18.23% 18.23%  1027.94kB 18.23%  runtime.hashGrow
     1025.75kB 18.19% 36.43%  1025.75kB 18.19%  runtime.rawstringtmp
     1024.08kB 18.16% 54.59%  1024.08kB 18.16%  text/template/parse.(*Tree).pipeline
     1024.03kB 18.16% 72.75%  1024.03kB 18.16%  vendor/golang_org/x/net/http2/hpack.addDecoderNode
      512.20kB  9.08% 81.84%   512.20kB  9.08%  runtime.malg
      512.10kB  9.08% 90.92%   512.10kB  9.08%  runtime.stringtoslicebyte
      512.02kB  9.08%   100%   512.02kB  9.08%  runtime.makemap
             0     0%   100%  4101.90kB 72.75%  github.com/astaxie/beego.BuildTemplate
             0     0%   100%  4101.90kB 72.75%  github.com/astaxie/beego.Run
        ....
    bogon:~ yang$ 
