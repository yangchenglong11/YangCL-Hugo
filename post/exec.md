---
author : "杨承龙"
date : "2017-06-18T12:27:43+08:00"
draft : false
title : "exec"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---
## package exec

exec包执行外部命令。它包装了os.StartProcess函数以便更容易的修正输入和输出，使用管道连接I/O，以及作其它的一些调整。

<!--*more*-->

### func [LookPath](https://github.com/golang/go/blob/master/src/os/exec/lp_unix.go#L33)

```go
func LookPath(file string) (string, error)
```

在环境变量PATH指定的目录中搜索可执行文件，如file中有斜杠，则只在当前目录搜索。返回完整路径或者相对于当前目录的一个相对路径。

Example

```go
path, err := exec.LookPath("fortune")
if err != nil {
    log.Fatal("installing fortune is in your future")
}
fmt.Printf("fortune is available at %s\n", path)
```

### type [Cmd](https://github.com/golang/go/blob/master/src/os/exec/exec.go#L35)

```go
type Cmd struct {
    // Path是将要执行的命令的路径。
    // 该字段不能为空，如为相对路径则为相对于Dir字段。
    Path string
  // Args存储命令的参数，其中Args[0]为当前位置到该命令的路径，使用Args[1:]可获得参数。
    // 典型用法下，Path和Args都被Command函数设定。
    Args []string
    // Env指定进程的环境，如为nil，则是在当前进程的环境下执行。
    Env []string
    // Dir指定命令的工作目录。如为空字符串，会在调用者的进程当前目录下执行。
    Dir string
    // Stdin指定进程的标准输入，如为nil，进程会从空设备读取（os.DevNull）
    Stdin io.Reader
    // Stdout和Stderr指定进程的标准输出和标准错误输出。
    // 如果任一个为nil，Run方法会将对应的文件描述符关联到空设备（os.DevNull）
    // 如果两个字段相同，同一时间最多有一个线程可以写入。
    Stdout io.Writer
    Stderr io.Writer
    // ExtraFiles指定额外被新进程继承的已打开文件流，不包括标准输入、标准输出、标准错误输出。
    // 如果本字段非nil，entry i会变成文件描述符3+i。
    //
    // BUG: 在OS X 10.6系统中，子进程可能会继承不期望的文件描述符。
    // http://golang.org/issue/2603
    ExtraFiles []*os.File
    // SysProcAttr保管可选的、各操作系统特定的sys执行属性。
    // Run方法会将它作为os.ProcAttr的Sys字段传递给os.StartProcess函数。
    SysProcAttr *syscall.SysProcAttr
    // Process是底层的，只执行一次的进程。
    Process *os.Process
    // ProcessState包含一个已经存在的进程的信息，只有在调用Wait或Run后才可用。
    ProcessState *os.ProcessState
    // 内含隐藏或非导出字段
}
```

Cmd代表一个正在准备或者在执行中的外部命令。

#### func (*Cmd) [StdoutPipe](https://github.com/golang/go/blob/master/src/os/exec/exec.go#L453)

```go
func (c *Cmd) StdoutPipe() (io.ReadCloser, error)
```

StdoutPipe方法返回一个在命令Start后与命令标准输出关联的管道。Wait方法获知命令结束后会关闭这个管道，一般不需要显式的关闭该管道。但是在从管道读取完全部数据之前调用Wait是错误的；同样使用StdoutPipe方法时调用Run函数也是错误的。返回值io.ReadCloser是一个接口类型并扩展了io.Reader。参见下例：

Example

```go
cmd := exec.Command("echo", "-n", `{"Name": "Bob", "Age": 32}`)
stdout, err := cmd.StdoutPipe()
if err != nil {
    log.Fatal(err)
}
if err := cmd.Start(); err != nil {
    log.Fatal(err)
}
var person struct {
    Name string
    Age  int
}
if err := json.NewDecoder(stdout).Decode(&person); err != nil {
    log.Fatal(err)
}
if err := cmd.Wait(); err != nil {
    log.Fatal(err)
}
fmt.Printf("%s is %d years old\n", person.Name, person.Age)
```

#### func (*Cmd) [StdinPipe](https://github.com/golang/go/blob/master/src/os/exec/exec.go#L411)

```go
func (c *Cmd) StdinPipe() (io.WriteCloser, error)
```

StdinPipe方法返回一个在命令Start后与命令标准输入关联的管道。Wait方法获知命令结束后会关闭这个管道。必要时调用者可以调用Close方法来强行关闭管道，例如命令在输入关闭后才会执行返回时需要显式关闭管道。

#### func [Command](https://github.com/golang/go/blob/master/src/os/exec/exec.go#L112)

```go
func Command(name string, arg ...string) *Cmd
```

函数返回一个*Cmd，用于使用给出的参数执行name指定的程序。返回值只设定了Path和Args两个参数。

如果name不含路径分隔符，将使用LookPath获取完整路径；否则直接使用name。参数arg不应包含命令名。

Example

```go
cmd := exec.Command("tr", "a-z", "A-Z")
cmd.Stdin = strings.NewReader("some input")
var out bytes.Buffer
cmd.Stdout = &out
err := cmd.Run()
if err != nil {
    log.Fatal(err)
}
fmt.Printf("in all caps: %q\n", out.String())
```

#### func (*Cmd) [StderrPipe](https://github.com/golang/go/blob/master/src/os/exec/exec.go#L478)

```go
func (c *Cmd) StderrPipe() (io.ReadCloser, error)
```

StderrPipe方法返回一个在命令Start后与命令标准错误输出关联的管道。Wait方法获知命令结束后会关闭这个管道，一般不需要显式的关闭该管道。但是在从管道读取完全部数据之前调用Wait是错误的；同样使用StderrPipe方法时调用Run函数也是错误的。请参照StdoutPipe的例子。

#### func (*Cmd) [Run](https://github.com/golang/go/blob/master/src/os/exec/exec.go#L235)

```go
func (c *Cmd) Run() error
```

Run执行c包含的命令，并阻塞直到完成。

如果命令成功执行，stdin、stdout、stderr的转交没有问题，并且返回状态码为0，方法的返回值为nil；如果命令没有执行或者执行失败，会返回*ExitError类型的错误；否则返回的error可能是表示I/O问题。

#### func (*Cmd) [Start](https://github.com/golang/go/blob/master/src/os/exec/exec.go#L272)

```go
func (c *Cmd) Start() error
```

Start开始执行c包含的命令，但并不会等待该命令完成即返回。Wait方法会返回命令的返回状态码并在命令返回后释放相关的资源。

Example

```go
cmd := exec.Command("sleep", "5")
err := cmd.Start()
if err != nil {
    log.Fatal(err)
}
log.Printf("Waiting for command to finish...")
err = cmd.Wait()
log.Printf("Command finished with error: %v", err)
```

#### func (*Cmd) [Wait](https://github.com/golang/go/blob/master/src/os/exec/exec.go#L349)

```go
func (c *Cmd) Wait() error
```

Wait会阻塞直到该命令执行完成，该命令必须是被Start方法开始执行的。

如果命令成功执行，stdin、stdout、stderr的转交没有问题，并且返回状态码为0，方法的返回值为nil；如果命令没有执行或者执行失败，会返回*ExitError类型的错误；否则返回的error可能是表示I/O问题。Wait方法会在命令返回后释放相关的资源。

#### func (*Cmd) [Output](https://github.com/golang/go/blob/master/src/os/exec/exec.go#L379)

```go
func (c *Cmd) Output() ([]byte, error)
```

执行命令并返回标准输出的切片。

Example

```go
out, err := exec.Command("date").Output()
if err != nil {
    log.Fatal(err)
}
fmt.Printf("The date is %s\n", out)
```

#### func (*Cmd) [CombinedOutput](https://github.com/golang/go/blob/master/src/os/exec/exec.go#L391)

```go
func (c *Cmd) CombinedOutput() ([]byte, error)
```

执行命令并返回标准输出和错误输出合并的切片。



文章来自：http://studygolang.com/pkgdoc