---
author : "杨承龙"
date : "2017-05-15T19:59:59+08:00"
draft : false
title : "set实现"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---
练习利用go语言的标准数据类型map实现python语言中的set数据结构

其他语言中，set的底层都是由哈希表(Hash Table)来实现的，go语言拥有作为Hash Table实现的字典Map类型。我们在对Set和Map进行比较之后会发现他们在一些主要特性上是及其相似的如下：

- 他们中的元素都是不可重复的。
- 他们都只能用迭代的方式取出其中的所有元素。
- 对他们中的元素进行迭代的顺序都是与元素插入顺序无关的，同时也不保证任何有序性，虽然有一些区别。
- Set的元素是一个单一的值，而map的元素则是一个键值对。
- Set的元素不可重复指的是不能存在任意两个单一值相等的情况。map的元素不可重复指的是任意两个键值对中的键的值不能相等。

我们可以发现Set更像是Map的一种简化版本，我们可不可以利用Map来编写一个Set的实现呢？答案当然是肯定的。

基本定义

    首先，我们创建一个名为hash_set.go的源码文件，把它放在项目的代码包set中。我们需要首先在这个源码文件的第一行上写入这样一行代码：

    package set

     这是为了声明源码文件hash_set.go是代码包set中的一员。然后我们声明一个包含了一个字典类型的字段的结构体类型。声明如下：

    type HashSet struct {
        m map[interface{}]bool
    }

    这个类型声明中的唯一的字段是map[interface{}]bool。之所以选择这样的一个字典类型是有原因的。因为我们希望HashSet类型的元素可以是任意类型的，所以我们将字典m的键类型设置为了interface{}。又由于我们只需要用到m的值中的键来储存HashSet类型的元素值，所以就应该选用值占用空间最小的类型来作为m的值的元素类型，这里使用bool类型有3个好处：

- 从值的储存形式的角度看，bool类型值的占用空间是最小的之一，只占用一个字节。
- 从值的表示形式的角度看，bool类型的值只有两个，这两个值都是预定义常量。
- 把bool类型作为值类型更有利于判断字典类型中是否存在某个键。例如，我们可以用m["a"]的结果值体现m的值中是否包含键为"a"键值对，但是，如果m的类型是map[interface{}]byte的话，那么我们只有通过v, ok := m["a"],才能确切得出上述判断的结果。虽然在向map[interface{}]byte类型的m的值添加键值对的时候，我们可以总以非零值的byte类型值作为其中的元素的值，但是我们在做判断的时候就需要编写更多的代码：

    if v := m["a"]; v != 0 { //如果“m“中不存在以”"a"作为键的键值对
      //省略若干语句
    }
    //对于map[interface{}]bool类型的值来说
    if m["a"]{//如果“m“中不存在以”"a"作为键的键值对
      //省略若干语句
    }

接下来我们考虑初始化HashSet类型值的问题。由于字典类型的零值为nil，所以我们不能简单使用new函数来创建一个HashSet类型值，因为 new(HashSet).m 的求值结果将会是一个 nil 。因此，这里需要编写一个专门用于创建和初始化 HashSet 类型值的函数，该函数声明如下：

    func NewHashSet() *HashSet {
      return &HashSet{m: make(map[interface{}]bool)}
    }

如上可以看到，使用make函数对字段m进行了初始化。同时注意观察函数 NewHashSet 的结果声明的类型是 *HashSet 而不是 HashSet，目的是让这个结果值的方法集合中包含调用接收者类型为 HashSet 或 *HashSet 的所有方法。这样做的好处将在后面编写 Set 接口类型的时候再予以说明。

2.基本功能

依据其他编程语言中的 HashSet 类型可知，它们大部分应该提供的基本功能如下：

- 添加元素值。
- 删除元素值。
- 清除所有元素值。
- 判断是否包含某个元素值。
- 获取元素值的数量。
- 判断与其他HashSet类型值是否相同。
- 获取所有元素值，即生成可迭代的快照。
- 获取自身的字符串表示形式。

(1).添加元素值

    //方法Add会返回一个bool类型的结果值，以表示添加元素值的操作是否成功。
    //方法Add的声明中的接收者类型是*HashSet。
    func (set *HashSet) Add(e interface{}) bool {
      if !set.m[e] {//当前的m的值中还未包含以e的值为键的键值对
        set.m[e] = true//将键为e(代表的值)、元素为true的键值对添加到m的值当中
        return true //添加成功
      }
      return false //添加失败
    }

这里使用 *HashSet 而不是 HashSet，主要是从节约内存空间的角度出发，分析如下：

当 Add 方法的接收者类型为 HashSet 的时候，对它的每一次调用都需要对当前 HashSet 类型值进行一次复制。虽然在 HashSet 类型中只有一个引用类型的字段，但是这也是一种开销。而且这里还没有考虑 HashSet 类型中的字段可能会变得更多的情况。

当 Add 方法的接收者类型为 *HashSet 的时候，对它进行调用时复制的当前 *HashSet 的类型值只是一个指针值。在大多数情况下，一个指针值占用的内存空间总会被它指向的那个其他类型的值所占用的内存空间小。无论一个指针值指向的那个其他类型值所需的内存空间有多么大，它所占用的内存空间总是不变的。

(2).删除元素值

    //调用delete内建函数删除HashSet内部支持的字典值
    func (set *HashSet) Remove(e interface{}) {
      delete(set.m, e)//第一个参数为目标字典类型，第二个参数为要删除的那个键值对的键
    }

(3).清除所有元素

    //为HashSet中的字段m重新赋值
    func (set *HashSet) Clear() {
      set.m = make(map[interface{}]bool)
    }

如果接收者类型是 HashSet，该方法中的赋值语句的作用只是为当前值的某个复制品中的字段m赋值而已，而当前值中的字段 m 则不会被重新赋值。方法 Clear 中的这条赋值语句被执行之后，当前的 HashSet 类型值中的元素就相当于被清空了。已经与字段 m 解除绑定的那个旧的字典值由于不再与任何程序实体存在绑定关系而成为了无用的数据。它会在之后的某一时刻被Go语言的垃圾回收器发现并回收。

(4).判断是否包含某个元素值。

    //方法Contains用于判断其值是否包含某个元素值。
    //这里判断结果得益于元素类型为bool的字段m
    func (set *HashSet) Contains(e interface{}) bool {
      return set.m[e]
    }

当把一个 interface{} 类型值作为键添加到一个字典值的时候，Go语言会先获取这个 interface{} 类型值的实际类型（即动态类型），然后再使用与之对应的 hash 函数对该值进行 hash 运算，也就是说，interface{} 类型值总是能够被正确地计算出 hash 值。但是字典类型的键不能是函数类型、字典类型或切片类型，否则会引发一个运行时恐慌，并提示如下： 

    panic: runtime error: hash of unhashable type <某个函数类型、字典类型或切片类型的名称>

(5).获取元素值的数量。

    //方法Len用于获取HashSet元素值数量
    func (set *HashSet) Len() int {
      return len(set.m)
    }

(6).判断与其他HashSet类型值是否相同。

    //方法Same用来判断两个HashSet类型值是否相同
    func (set *HashSet) Same(other *HashSet) bool {
      if other == nil {
        return false
      }
      if set.Len() != other.Len() {
        return false
      }
      for key := range set.m {
        if !other.Contains(key) {
          return false
        }
      }
      return true
    }

两个 HashSet 类型值相同的必要条件是，它们包含的元素应该是完全相同的。由于 HashSet 类型值中的元素的迭代顺序总是不确定的，所以也就不用在意两个值在这方面是否一致。如果要判断两个 HashSet 类型值是否是同一个值，就需要利用指针运算进行内存地址的比较。

(7).获取所有元素值，即生成可迭代的快照。

所谓 快照，就是目标值在某一个时刻的映像。对于一个 HashSet 类型值来说，它的快照中的元素迭代顺序总是可以确定的，快照只反映了该 HashSet 类型值在某一个时刻的状态。另外，还需要从元素可迭代且顺序可确定的数据类型中选取一个作为快照的类型。这个类型必须是以单值作为元素的，所以字典类型最先别排除。又由于 HashSet 类型值中的元素数量总是不固定的，所以无法用一个数组类型的值来表示它的快照。如上分析可知，Go语言中可以使用的快照的类型应该是一个切片类型或者通道类型。

    //方法Elements用于生成快照
    func (set *HashSet) Elements() []interface{} {
      initialLen := len(set.m)
      //初始化一个[]interface{}类型的变量snapshot来存储m的值中的元素值
      snapshot := make([]interface{}, initialLen)
      actualLen := 0
      //按照既定顺序将迭代值设置到快照值(变量snapshot的值)的指定元素位置上,这一过程并不会创建任何新值。
      for key := range set.m {
        if actualLen < initialLen {
          snapshot[actualLen] = key
        } else {
          snapshot = append(snapshot, key)
        }
        actualLen++//实际迭代的次数
      }
     
      if actualLen < initialLen {
        snapshot = snapshot[:actualLen]
      }
      return snapshot
    }

之所以我们使用这么多条语句来实现这个方法是因为需要考虑到在从获取字段m的值的长度到对m的值迭代完成的这个时间段内，m的值中的元素数量可能发生变化。如果在迭代完成之前，m的值中的元素数量有所增加，使得实际迭代的次数大于先前初始化的快照值的长度 ，那么我们再使用appeng函数向快照值追加元素值。这样做既提高了快照生成的效率，又不至于在元素数量增加时引发索引越界的运行时恐慌。

对于已被初始化的[]interface{}类型的切片值来说，未被显示初始化的元素位置上的值均为nil。如果在迭代完成前，m的值中的元素数量有所减少， 致使快照值的尾部存在若干个没有任何意义的值为nil的元素，我们需要把这些无用的元素从快照中去掉。可以通过snapshot = snapshot[:actualLen]将无用的元素值从快照值中去掉。

注意：在 Elements 方法中针对并发访问和修改 m 的值的情况采取了一些措施。但是由于m的值本身并不是并发安全的，所以并不能保证 Elements 方法的执行总会准确无误。要做到真正的并发安全，还需要一些辅助的手段，比如读写互斥量。

(8).获取自身的字符串表示形式。



    //这个String方法的签名算是一个惯用法。 
    //代码包fmt中的打印函数总会使用参数值附带的具有如此签名的String方法的结果值作为该参数值的字符串表示形式。
    func (set *HashSet) String() string {
      var buf bytes.Buffer//作为结果值的缓冲区
      buf.WriteString("HashSet{")
      first := true
      for key := range set.m {
        if first {
          first = false
        } else {
          buf.WriteString(",")
        }
        buf.WriteString(fmt.Sprintf("%v", key))
      }
      //n := 1
      //for key := range set.m {
      // buf.WriteString(fmt.Sprintf("%v", key))
      // if n == len(set.m) {//最后一个元素的后面不添加逗号
      // break;
      // } else {
      // buf.WriteString(",")
      // }
      // n++;
      //}
      buf.WriteString("}")
      return buf.Strin
    }

如上已经完整地编写了一个具备常用功能的Set的实现类型，后面将讲解更多的高级功能来完善它。
