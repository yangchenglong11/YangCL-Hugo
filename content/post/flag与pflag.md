---
author : "杨承龙"
date : "2017-06-30T16:28:06+08:00"
draft : false
title : "flag与pflag"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---
flag

flag 是Go 标准库提供的解析命令行参数的包。flag跟以往的传统命令行解析不同,它将参数定义, 参数默认值, 命令帮助, 参数读取集成到一起,摆脱以前繁琐的命令判断流程.

使用方式：

    flag.Type(name, defValue, usage)

其中Type为String, Int, Bool等；并返回一个相应类型的指针。

    flag.TypeVar(&flagvar, name, defValue, usage)

将flag绑定到一个变量上。

返回指针

通过flag.String, flag.Int, flag.Bool等可以直接返回一个参数指针,因此后续使用的时候需要加*进行取值:

        str = flag.String("mystring", "default-value", "mystring is my test flag of String")
        i = flag.Int("myint", 123, "test int flag")

序列化到变量

通过flag.StringVar, flag.IntVar, flag.BoolVar等,可以直接把值序列化到变量:

        flag.StringVar(&vStr, "mystring2", "default-value2", "mystring2 is my test flag of String")
    	flag.IntVar(&vI, "myint2", 123, "test int flag")

自定义flag

只要实现flag.Value接口即可：

    type Value interface {
      String() string
      Set(string) error
    }

通过如下方式定义该flag：

    flag.Var(&flagvar, name, usage)

example：

    package main
    import(
    	"flag"
    	"fmt"
    	"strconv"
    ) 
    
    type percentage float32
    
    func (p *percentage) Set(s string) error {
    	v, err := strconv.ParseFloat(s, 32)
    	*p = percentage(v)
    	return err
    }
    
    func (p *percentage) String() string {
    	return fmt.Sprintf("%f", *p) 
    }
    
    func main() {
    	namePtr := flag.String("name", "yang", "user's name")
    	agePtr := flag.Int("age", 20, "user's age")
    	vipPtr := flag.Bool("vip", true, "is a vip user")
    	
    	var email string
    	
    	flag.StringVar(&email, "email", "xxxxxx@gmail.com", "user's email")
    	
    	var pop percentage
    	
    	flag.Var(&pop, "pop", "popularity")
    	
    	flag.Parse()
    	
    	others := flag.Args()
    	fmt.Println("name:", *namePtr)
    	fmt.Println("age:", *agePtr)
    	fmt.Println("vip:", *vipPtr)
    	fmt.Println("pop:", pop)
    	fmt.Println("email:", email)
    	fmt.Println("other:", others)
    }

输出：

    $ go run test.go 
    name: yang
    age: 20
    vip: true
    email: xxxxxx@gmail.com
    other: []



    $ go run test.go -name golang -age 4 -vip=true -pop 99 简洁 高并发 等等
    name: golang
    age: 4
    vip: true
    pop: 99
    email: lyhopq@gmail.com
    other: [简洁 高并发 等等]



    $ go run test.go -h
    Usage of go /var/folders/6y/z4y473d12fj8ss2gkr276dx80000gn/T/go-build217780999/command-line-arguments/_obj/exe/test :
     -age=20: user's age
     -email="xxxxxx@gmail.com": user's email
     -name="yang": user's name
     -pop=0.0: popularity
     -vip=true: is a vip user



pflag

pflag是flag扩展,主要是加入了对短参数的支持.

安装

    go get github.com/ogier/pflag

测试

    go test github.com/ogier/pflag

使用

如果你之前已经import了flag包,那么直接把那行改为:

    import flag "github.com/ogier/pflag"

短参数支持

在函数后面加上P即可:

        pflag.StringVarP(&p1, "stringflag", "s", "test-p-flag-string", "test pflag for string.")
    	pflag.IntVarP(&p2, "intflag", "i", 12345, "test pflag for int.")

程序运行结果

    $ go run main.go
    
    Usage of /private/var/folders/98/pmb6b_8x5w5319m9ts6h6z6m0000gn/T/Build main.go and run1go:
      -i, --intflag=12345: test pflag for int.
      -s, --stringflag="test-p-flag-string": test pflag for string.
