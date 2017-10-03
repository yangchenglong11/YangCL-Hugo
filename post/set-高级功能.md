---
title: set 高级功能
date: 2016-10-23 10:51:12 
categories: Golang 
tags: [Golang] 
description: 
---
在集合代数中除了对集合基本性质和规律的描述外，还包含了对各种集合运算和集合关系的说明。集合的运算包括并集，交集，差集还有对称差集。集合的关系包括相等和真包含。这篇文章我们就来实现这些功能。

<!--more-->
首先我们添加集合真包含的判断功能。根据集合代数中的描述，如果集合 A 真包含了集合 B，那么就可以说集合 A 是集合 B 的一个超集。

```go
// 判断集合 set 是否是集合 other 的超集 
func (set *HashSet) IsSuperset(other *HashSet) bool {
  if other == nil {         // 如果other为nil，则other不是set的子集
    return false
  }
  setLen := set.Len()       // 获取set的元素值数量
  otherLen := other.Len()   // 获取other的元素值数量
  if setLen == 0 || setLen == otherLen {   // set的元素值数量等于0或者等于other的元素数量
    return false
  }
  if setLen > 0 && otherLen == 0 {         // other为元素数量为0，set元素数量大于0，则set也是other的超集
    return true
  }
  for _, v := range other.Elements() {
    if !set.Contains(v) {                  // 只要set中有一个other中的数据不在其中，就返回false
      return false
    }
  }
  return true
}
```

集合的运算包括**并集、交集、差集**和**对称差集**。 

**并集运算**是指把两个集合中的所有元素都合并起来并组合成一个集合。 

**交集运算**是指找到两个集合中共有的元素并把它们组成一个集合。 

**集合 A** 对**集合 B **进行**差集运算**的含义是找到只存在于**集合 A** 中但不存在于**集合 B** 中的元素并把它们组成一个集合。 

对称差集运算与差集运算类似但有所区别。对称差集运算是指找到只存在于**集合 A** 中但不存在于**集合 B** 中的元素，再找到只存在于集合 B 中但不存在于集合 A 中的元素，最后把它们合并起来并组成一个集合。

**实现并集运算**

```go
// 生成集合 set 和集合 other 的并集
func (set *HashSet) Union(other *HashSet) *HashSet {
  if set == nil || other == nil {    // set和other都为nil，则它们的并集为nil
    return nil
  }
  unionedSet := NewHashSet()         // 新创建一个HashSet类型值，它的长度为0，即元素数量为0
  for _, v := range set.Elements() { // 将set中的元素添加到unionedSet中
    unionedSet.Add(v)
  }
  if other.Len() == 0 {
    return unionedSet
  }
  for _, v := range other.Elements() {  // 将other中的元素添加到unionedSet中，如果遇到相同，则不添加（在Add方法逻辑中体现）
    unionedSet.Add(v)
  }
  return unionedSet
}
```

**实现交集运算**



```go
// 生成集合 set 和集合 other 的交集
func (set *HashSet) Intersect(other *HashSet) *HashSet {
  if set == nil || other == nil {   // set和other都为nil，则它们的交集为nil
    return nil
  }
  intersectedSet := NewHashSet()    // 新创建一个HashSet类型值，它的长度为0，即元素数量为0
  if other.Len() == 0 {             // other的元素数量为0，直接返回intersectedSet
    return intersectedSet
  }
  if set.Len() < other.Len() {      // set的元素数量少于other的元素数量
    for _, v := range set.Elements() {   // 遍历set
      if other.Contains(v) {        // 只要将set和other共有的添加到intersectedSet
        intersectedSet.Add(v)
      }
    }
  } else {                          // set的元素数量多于other的元素数量
    for _, v := range other.Elements() {   // 遍历other
      if set.Contains(v) {          // 只要将set和other共有的添加到intersectedSet
        intersectedSet.Add(v)
      }
    }
  }
  return intersectedSet
}
```

**差集**



```go
// 生成集合 set 对集合 other 的差集
func (set *HashSet) Difference(other *HashSet) *HashSet {
  if set == nil || other == nil {  // set和other都为nil，则它们的差集为nil
    return nil
  }
  differencedSet := NewHashSet()   // 新创建一个HashSet类型值，它的长度为0，即元素数量为0
  if other.Len() == 0 {            // 如果other的元素数量为0
    for _, v := range set.Elements() {  // 遍历set，并将set中的元素v添加到differencedSet
      differencedSet.Add(v)
    }
    return differencedSet          // 直接返回differencedSet
  }
  for _, v := range set.Elements() {  // other的元素数量不为0，遍历set
    if !other.Contains(v) {        // 如果other中不包含v，就将v添加到differencedSet中
      differencedSet.Add(v)
    }
  }
  return differencedSet
}
```

**对称差集**



```go
// 生成集合 one 和集合 other 的对称差集
func (set *HashSet) SymmetricDifference(other *HashSet) *HashSet {
  if set == nil || other == nil {  // set和other都为nil，则它们的对称差集为nil
    return nil
  }
  diffA := set.Difference(other)   // 生成集合 set 对集合 other 的差集
  if other.Len() == 0 {            // 如果other的元素数量等于0，那么other对集合set的差集为空，则直接返回diffA
    return diffA
  }
  diffB := other.Difference(set)   // 生成集合 other 对集合 set 的差集
  return diffA.Union(diffB)        // 返回集合 diffA 和集合 diffB 的并集
}
```

**4.进一步重构**

目前所实现的 **HashSet **类型提供了一些必要的集合操作功能，但是不同应用场景下可能会需要使用功能更加丰富的集合类型。当有多个集合类型的时候，应该在它们之上抽取出一个接口类型以标识它们共有的行为方式。依据 **HashSet **类型的声明，可以如下声明 Set 接口类型：

```go
type Set interface {
  Add(e interface{}) bool
  Remove(e interface{})
  Clear()
  Contains(e interface{}) bool
  Len() int
  Same(other Set) bool
  Elements() []interface{}
  String() string
}
```

**注意：** **Set **中的 **Same **方法的签名与附属于 **HashSet**类型的 **Same **方法有所不同。这里不能再接口类型的方法的签名中包含它的实现类型。因此这里的改动如下：

```go
func (set *HashSet) Same(other Set) bool {
  // 省略若干语句
}
```

修改了 **Same **方法的签名，目的是让 ***HashSet **类型成为 Set 接口类型的一个实现类型。

高级功能的方法应该适用于所有的实现类型，完全可以抽离出成为独立的函数。并且，也不应该在每个实现类型中重复地实现这些高级方法。如下为改造后的 **IsSuperset **方法的声明：

```go
// 判断集合 one 是否是集合 other 的超集
// 读者应重点关注IsSuperset与附属于HashSet类型的IsSuperset方法的区别
func IsSuperset(one Set, other Set) bool {
  if one == nil || other == nil {
    return false
  }
  oneLen := one.Len()
  otherLen := other.Len()
  if oneLen == 0 || oneLen == otherLen {
    return false
  }
  if oneLen > 0 && otherLen == 0 {
    return true
  }
  for _, v := range other.Elements() {
    if !one.Contains(v) {
      return false
    }
  }
  return true
}
```

以上就是Go语言之自定义集合Set的全部内容，希望对大家学习Go语言有所帮助。



点此查看代码[github代码](https://github.com/hyper0x/goc2p)