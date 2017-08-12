---
author : "杨承龙"
date : "2017-07-06T14:05:14+08:00"
draft : false
title : "RPC"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---
# RPC

我们知道，Socket和HTTP采用的是类似"信息交换"模式，即客户端发送一条信息到服务端，然后(一般来说)服务器端都会返回一定的信息以表示响应。客户端和服务端之间约定了交互信息的格式，以便双方都能够解析交互所产生的信息。但是很多独立的应用并没有采用这种模式，而是采用类似常规的函数调用的方式来完成想要的功能。

RPC就是想实现函数调用模式的网络化。客户端就像调用本地函数一样，然后客户端把这些参数打包之后通过网络传递到服务端，服务端解包到处理过程中执行，然后执行的结果反馈给客户端。

RPC（Remote Procedure Call Protocol）——远程过程调用协议，是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。它假定某些传输协议的存在，如TCP或UDP，以便为通信程序之间携带信息数据。通过它可以使函数调用模式网络化。在OSI网络通信模型中，RPC跨越了传输层和应用层。RPC使得开发包括网络分布式多程序在内的应用程序更加容易。

运行时,一次客户机对服务器的RPC调用,其内部操作大致有如下十步：

- 1.调用客户端句柄；执行传送参数
- 2.调用本地系统内核发送网络消息
- 3.消息传送到远程主机
- 4.服务器句柄得到消息并取得参数
- 5.执行远程过程
- 6.执行的过程将结果返回服务器句柄
- 7.服务器句柄返回结果，调用远程系统内核
- 8.消息传回本地主机
- 9.客户句柄由内核接收消息
- 10.客户接收句柄返回的数据

包rpc提供了通过网络访问一个对象的方法的能力，服务器需要注册对象， 通过对象的类型名暴露这个服务。

 <!--*more*--> 

注册后这个对象的输出方法就可以远程调用，这个库封装了底层传输的细节，包括序列化。 服务器可以注册多个不同类型的对象，但是注册相同类型的多个对象的时候回出错。它支持三个级别的RPC：TCP、HTTP、JSONRPC。但Go的RPC包和传统的RPC系统不同，它只支持Go开发的服务器与客户端之间的交互，因为在内部，它们采用了Gob来编码。

同时，如果对象的方法要能远程访问，它们必须满足一定的条件，否则这个对象的方法会被忽略。

这些条件是：

* 方法的类型是可输出的 (the method's type is exported)
* 方法本身也是可输出的( 即函数必须是导出的(首字母大写)) （the method is exported）
* 方法必须有两个参数，第一个参数是接收的参数，第二个参数是返回给客户端的参数。必须是输出类型或者是内建类型 (the method has two arguments, both exported or builtin types)
* 方法的第二个参数是指针类型 
* 函数还要有一个返回值error

举个例子，正确的RPC函数格式如下：
```go
func (t *T) MethodName(argType T1, replyType *T2) error
```
T、T1和T2类型必须能被encoding/gob包编解码。

任何的RPC都需要通过网络来传递数据，Go RPC可以利用HTTP和TCP来传递数据，利用HTTP的好处是可以直接复用net/http里面的一些函数。详细的例子请看下面的实现

HTTP RPC

http的服务端代码实现如下：
```go
package main

import (
	"errors"
	"fmt"
	"net/http"
	"net/rpc"
)

type Args struct {
	A, B int
}

type Quotient struct {
	Quo, Rem int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
	*reply = args.A * args.B
	return nil
}

func (t *Arith) Divide(args *Args, quo *Quotient) error {
	if args.B == 0 {
		return errors.New("divide by zero")
	}
	quo.Quo = args.A / args.B
	quo.Rem = args.A % args.B
	return nil
}

func main() {

	arith := new(Arith)
	rpc.Register(arith)
	rpc.HandleHTTP()

	err := http.ListenAndServe(":1234", nil)
	if err != nil {
		fmt.Println(err.Error())
	}
}
```
通过上面的例子可以看到，我们注册了一个Arith的RPC服务，然后通过rpc.HandleHTTP函数把该服务注册到了HTTP协议上，然后我们就可以利用http的方式来传递数据了。

请看下面的客户端代码：
```go
package main

import (
	"fmt"
	"log"
	"net/rpc"
	"os"
)

type Args struct {
	A, B int
}

type Quotient struct {
	Quo, Rem int
}

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "server")
		os.Exit(1)
	}
	serverAddress := os.Args[1]

	client, err := rpc.DialHTTP("tcp", serverAddress+":1234")
	if err != nil {
		log.Fatal("dialing:", err)
	}
	// Synchronous call
	args := Args{17, 8}
	var reply int
	err = client.Call("Arith.Multiply", args, &reply)
	if err != nil {
		log.Fatal("arith error:", err)
	}
	fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)

	var quot Quotient
	err = client.Call("Arith.Divide", args, &quot)
	if err != nil {
		log.Fatal("arith error:", err)
	}
	fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B, quot.Quo, quot.Rem)

}
```
我们把上面的服务端和客户端的代码分别编译，然后先把服务端开启，然后开启客户端，输入代码，就会输出如下信息：
```
$ ./http_c localhost
Arith: 17*8=136
Arith: 17/8=2 remainder 1
```
通过上面的调用可以看到参数和返回值是我们定义的struct类型，在服务端我们把它们当做调用函数的参数的类型，在客户端作为client.Call的第2，3两个参数的类型。客户端最重要的就是这个Call函数，它有3个参数，第1个要调用的函数的名字，第2个是要传递的参数，第3个要返回的参数(注意是指针类型)。
在客户端中我们也可以使用异步方式进行远程调用：
```go
quotient := new(Quotient)
divCall := client.Go("Arith.Divide", args, quotient, nil)
replyCall := <-divCall.Done    // will be equal to divCall
// check errors, print, etc.
```

上面我们实现了基于HTTP协议的RPC，接下来我们要实现基于TCP协议的RPC，服务端只要将main函数修改即可
```go
func main() {

	arith := new(Arith)
	rpc.Register(arith)

	tcpAddr, err := net.ResolveTCPAddr("tcp", ":1234")
	checkError(err)

	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)

	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}
		rpc.ServeConn(conn)
	}

}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```
和http的服务器相比，不同在于:在此处我们采用了TCP协议，然后需要自己控制连接，当有客户端连接上来后，我们需要把这个连接交给rpc来处理。
客户端代码和http的客户端代码对比，唯一的区别一个是DialHTTP，一个是Dial(tcp)，其他处理一模一样。
```go
client, err := rpc.Dial("tcp", service)
```
当RPC已经注册的时候就要使用了另外一种了。即一个server只能在DefaultRPC中注册一种类型。
```go
package main
 
import (
        "net/rpc"
        //"net/http"
        "log"
        "net"
        "time"
)
 ......
func main() {
        newServer := rpc.NewServer()
        newServer.Register(new(Arith))
 
        l, e := net.Listen("tcp", "127.0.0.1:1234") // any available address
        if e != nil {
                log.Fatalf("net.Listen tcp :0: %v", e)
        }
 
        go newServer.Accept(l)
        newServer.HandleHTTP("/foo", "/bar")
        time.Sleep(2 * time.Second)
 
        address, err := net.ResolveTCPAddr("tcp", "127.0.0.1:1234")
        if err != nil {
                panic(err)
        }
        conn, _ := net.DialTCP("tcp", nil, address)
        defer conn.Close()
 
        client := rpc.NewClient(conn)
        defer client.Close()
 
        args := &Args{7,8}
        reply := make([]string, 10)
        err = client.Call("Arith.Multiply", args, &reply)
        if err != nil {
                log.Fatal("arith error:", err)
        }
        log.Println(reply)
}
```