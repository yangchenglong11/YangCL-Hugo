# Go语言版crontab

## 1、cron 表达式的基本格式

用过 linux 的应该对 cron 有所了解。linux 中可以通过 crontab -e 来配置定时任务。不过，linux 中的 cron 只能精确到分钟。而我们这里要讨论的 Go 实现的 cron 可以精确到秒，除了这点比较大的区别外，cron 表达式的基本语法是类似的。（如果使用过 Java 中的 Quartz，对 cron 表达式应该比较了解，而且它和这里我们将要讨论的 Go 版 cron 很像，也都精确到秒）

cron(计划任务)，顾名思义，按照约定的时间，定时的执行特定的任务（job）。cron 表达式 表达了这种约定。

cron 表达式代表了一个时间集合，使用 6 个空格分隔的字段表示。

| 字段名             | 是否必须 | 允许的值            | 允许的特定字符   |
| --------------- | ---- | --------------- | --------- |
| 秒(Seconds)      | 是    | 0-59            | * / , -   |
| 分(Minutes)      | 是    | 0-59            | * / , -   |
| 时(Hours)        | 是    | 0-23            | * / , -   |
| 日(Day of month) | 是    | 1-31            | * / , – ? |
| 月(Month)        | 是    | 1-12 or JAN-DEC | * / , -   |
| 星期(Day of week) | 否    | 0-6 or SUM-SAT  | * / , – ? |

**注：**
1）月(Month)和星期(Day of week)字段的值不区分大小写，如：SUN、Sun 和 sun 是一样的。
2）星期
(Day of week)字段如果没提供，相当于是 *

## 2、特殊字符说明

### 1）星号(*)

表示 cron 表达式能匹配该字段的所有值。如在第5个字段使用星号(month)，表示每个月

### 2）斜线(/)

表示增长间隔，如第1个字段(minutes) 值是 3-59/15，表示每小时的第3分钟开始执行一次，之后每隔 15 分钟执行一次（即 3、18、33、48 这些时间点执行），这里也可以表示为：3/15

### 3）逗号(,)

用于枚举值，如第6个字段值是 MON,WED,FRI，表示 星期一、三、五 执行

### 4）连字号(-)

表示一个范围，如第3个字段的值为 9-17 表示 9am 到 5pm 直接每个小时（包括9和17）

### 5）问号(?)

只用于 日(Day of month) 和 星期(Day of week)，表示不指定值，可以用于代替 *

## 3、主要类型或接口说明

[查看源码](https://github.com/robfig/cron)

### 1）Cron：包含一系列要执行的实体；支持暂停【stop】；添加实体等

```
type Cron struct {
    entries  []*Entry
    stop     chan struct{}   // 控制 Cron 实例暂停
    add      chan *Entry     // 当 Cron 已经运行，增加新的 Entity 是通过 add 这个 channel 实现的
    snapshot chan []*Entry   // 获取当前所有 entity 的快照
    running  bool            // 当已经运行时为true；否则为false
}
```

注意，Cron 结构没有导出任何成员。

注意：有一个成员 stop，类型是 `chan struct{}`，即一个空结构体的通道。

### 2）Entry：调度实体

```
type Entry struct {
    // The schedule on which this job should be run.
    // 负责调度当前 Entity 中的 Job 执行
    Schedule Schedule

    // The next time the job will run. This is the zero time if Cron has not been
    // started or this entry's schedule is unsatisfiable
    // Job 下一次执行的时间
    Next time.Time

    // The last time this job was run. This is the zero time if the job has never
    // been run.
    // 上一次执行时间
    Prev time.Time

    // The Job to run.
    // 要执行的 Job
    Job Job
}
```

### 3）Job：每一个实体包含一个需要运行的Job

这是一个接口，只有一个方法：run

```
type Job interface {
    Run()
}
```

由于 Entity 中需要 Job 类型，因此，我们希望定期运行的任务，就需要实现 Job 接口。同时，由于 Job 接口只有一个无参数无返回值的方法，为了使用方便，作者提供了一个类型：

```
type FuncJob func()
```

它通过简单的实现 Run() 方法来实现 Job 接口：

```
func (f FuncJob) Run() { f() }
```

这样，任何无参数无返回值的函数，通过强制类型转换为 FuncJob，就可以当作 Job 来使用了，AddFunc 方法 就是这么做的。

### 4）Schedule：每个实体包含一个调度器（Schedule）

负责调度 Job 的执行。它也是一个接口。

```
type Schedule interface {
    // Return the next activation time, later than the given time.
    // Next is invoked initially, and then each time the job is run.
    // 返回同一 Entity 中的 Job 下一次执行的时间
    Next(time.Time) time.Time
}
```

Schedule 的具体实现通过解析 Cron 表达式得到。

库中提供了 Schedule 的两个具体实现，分别是 SpecSchedule 和 ConstantDelaySchedule。

**① SpecSchedule**

```
type SpecSchedule struct {
    Second, Minute, Hour, Dom, Month, Dow uint64
}
```

从开始介绍的 Cron 表达式可以容易得知各个字段的意思，同时，对各种表达式的解析也会最终得到一个 SpecSchedule 的实例。库中的 Parse 返回的其实就是 SpecSchedule 的实例（当然也就实现了 Schedule 接口）。

**② ConstantDelaySchedule**

```
type ConstantDelaySchedule struct {
    Delay time.Duration // 循环的时间间隔
}
```

这是一个简单的循环调度器，如：每 5 分钟。注意，最小单位是秒，不能比秒还小，比如 毫秒。

通过 Every 函数可以获取该类型的实例，如：

```
constDelaySchedule := Every(5e9)
```

得到的是一个每 5 秒执行一次的调度器。

## 4、主要实例化方法

### 1） 函数

**① 实例化 Cron**

```
func New() *Cron {
    return &Cron{
        entries:  nil,
        add:      make(chan *Entry),
        stop:     make(chan struct{}),
        snapshot: make(chan []*Entry),
        running:  false,
    }
}
```

可见实例化时，成员使用的基本是默认值；

**② 解析 Cron 表达式**

```
func Parse(spec string) (_ Schedule, err error)
```

spec 可以是：

- Full crontab specs, e.g. “* * * * * ?”
- Descriptors, e.g. “@midnight”, “@every 1h30m”

**② 成员方法**

```
// 将 job 加入 Cron 中
// 如上所述，该方法只是简单的通过 FuncJob 类型强制转换 cmd，然后调用 AddJob 方法
func (c *Cron) AddFunc(spec string, cmd func()) error

// 将 job 加入 Cron 中
// 通过 Parse 函数解析 cron 表达式 spec 的到调度器实例(Schedule)，之后调用 c.Schedule 方法
func (c *Cron) AddJob(spec string, cmd Job) error

// 获取当前 Cron 总所有 Entities 的快照
func (c *Cron) Entries() []*Entry

// 通过两个参数实例化一个 Entity，然后加入当前 Cron 中
// 注意：如果当前 Cron 未运行，则直接将该 entity 加入 Cron 中；
// 否则，通过 add 这个成员 channel 将 entity 加入正在运行的 Cron 中
func (c *Cron) Schedule(schedule Schedule, cmd Job)

// 新启动一个 goroutine 运行当前 Cron
func (c *Cron) Start()

// 通过给 stop 成员发送一个 struct{}{} 来停止当前 Cron，同时将 running 置为 false
// 从这里知道，stop 只是通知 Cron 停止，因此往 channel 发一个值即可，而不关心值是多少
// 所以，成员 stop 定义为空 struct
func (c *Cron) Stop()
```

## 5、使用示例

```
package main

import (
    "github.com/robfig/cron"
    "log"
)

func main() {
    i := 0
    c := cron.New()
    spec := "*/5 * * * * ?"
    c.AddFunc(spec, func() {
        i++
        log.Println("cron running:", i)
    })
    c.Start()

    select{}
}
```

这是一个简单示例。

输出类似这样：

> 2014/02/22 21:23:40 cron running: 1
> 2014/02/22 21:23:45 cron running: 2
> 2014/02/22 21:23:50 cron running: 3
> 2014/02/22 21:23:55 cron running: 4
> 2014/02/22 21:24:00 cron running: 5

可见 cron 的使用还是挺简单的。

[查看原文](http://blog.studygolang.com/2014/02/go_crontab/)