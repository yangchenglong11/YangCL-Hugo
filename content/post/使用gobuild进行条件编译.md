---
author : "杨承龙"
date : "2017-08-07T12:58:45+08:00"
draft : false
title : "使用gobuild进行条件编译"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---
使用gobuild进行条件编译

当我们编写的Go代码依赖特定平台或者cpu架构的时候，我们需要给出不同的实现

C语言有预处理器，可以通过宏或者#define包含特定平台指定的代码进行编译

但是Go没有预处理器，他是通过 go/build包 里定义的tags和命名约定来让Go的包可以管理不同平台的代码

这篇文章将讲述Go的条件编译系统是如何实现的，并且通过实例来说明如何使用

预备知识：go list命令的使用

在讲条件编译之前需要了解go list的简单用法

go list访问源文件里那些能够影响编译进程内部的数据结构

go list与go build ,test,install大部分的参数相同，但是go list不会执行编译操作。使用-f参数可以让我们提供的text/template里的代码在包含go/build.Package上下文的环境里正确执行（就是让go/build.Package里的上下文去格式化 text/template里这种格式 '{{.GoFiles}}'里的占位符，写过http server程序的同学看到应该很熟悉）

使用格式化参数，我们能通过go list获取将会被编译的文件名

    % go list -f '{{.GoFiles}}' os/exec  
    
    [exec.go lp_unix.go]  

上面这个例子里我们用go list来查看在Linux/arm平台下 os/exec包里有哪些文件将会被编译。

结果显示：exec.go包含了通用的代码在所有的平台下可用，lp_unix.go包含了*nix系统里的exec.LookPath

在windows系统下运行同样的命令，结果如下：

    C:\go> go list -f '{{.GoFiles}}' os/exec  
    
    [exec.go lp_windows.go]  

上面这个例子是Go 条件编译系统的两个部分，称之为：编译约束，下面将详细描述

第一种条件编译的方法：编译标签

在源代码里添加标注，通常称之为编译标签( build tag)

编译标签是在尽量靠近源代码文件顶部的地方用注释的方式添加

go build在构建一个包的时候会读取这个包里的每个源文件并且分析编译便签，这些标签决定了这个源文件是否参与本次编译

编译标签添加的规则（附上原文）：

- a build tag is evaluated as the OR of space-separated options
- each option evaluates as the AND of its comma-separated terms
- each term is an alphanumeric word or, preceded by !, its negation
- 编译标签由空格分隔的编译选项(options)以"或"的逻辑关系组成
- 每个编译选项由逗号分隔的条件项以逻辑"与"的关系组成
- 每个条件项的名字用字母+数字表示，在前面加!表示否定的意思

例子（编译标签要放在源文件顶部）

    // +build darwin freebsd netbsd openbsd  

这个将会让这个源文件只能在支持kqueue的BSD系统里编译

一个源文件里可以有多个编译标签，多个编译标签之间是逻辑"与"的关系

    // +build linux darwin  
    
    // +build 386  

这个将限制此源文件只能在 linux/386或者darwin/386平台下编译

关于注释的说明

刚开始使用编译标签经常会犯下面这个错误

    // +build !linux  
    
    package mypkg // wrong  

空行

注释

下面这个是正确的标签的书写方式，标签的结尾添加一个空行这样标签就不会当做其他声明的注释

    // +build !linux  
    
    
    
    package mypkg // correct  

用go vet命令也可以检测到这个缺少空行的错误，初期可以用这个命令来避免缺少空行的错误

    % go vet mypkg  
    
    mypkg.go:1: +build comment appears too late in file  
    
    exit status 1  

作为参考，下面的例子将licence声明,编译标签和包声明放在一起，请大家注意分辨

    % head headspin.go   
    
    // Copyright 2013 Way out enterprises. All rights reserved.  
    
    // Use of this source code is governed by a BSD-style  
    
    // license that can be found in the LICENSE file.  
    
    
    
    // +build someos someotheros thirdos,!amd64  
    
    
    
    // Package headspin implements calculates numbers so large  
    
    // they will make your head spin.  
    
    package headspin  



第二种条件编译方法：文件后缀

这个方法通过改变文件名的后缀来提供条件编译，这种方案比编译标签要简单，go/build可以在不读取源文件的情况下就可以决定哪些文件不需要参与编译

文件命名约定可以在go/build 包里找到详细的说明，简单来说如果你的源文件包含后缀：$GOOS.go，那么这个源文件只会在这个平台下编译，$GOARCH.go也是如此。这两个后缀可以结合在一起使用，但是要注意顺序：$GOOS$GOARCH.go，    不能反过来用：$GOARCH$GOOS.go

例子如下：

    mypkg_freebsd_arm.go // only builds on freebsd/arm systems  
    
    mypkg_plan9.go       // only builds on plan9  

源文件不能只提供条件编译后缀，还必须有文件名：

    _linux.go  
    
    _freebsd386.go  

这两个源文件在所有平台下都会被忽略掉，因为go/build将会忽略所有以下划线或者点开头的源文件 

编译标签和文件后缀的选择

编译标签和文件后缀的功能上有重叠，例如一个文件名：mypkg_linux.go包含了// +build linux将会出现冗余

通常情况下，如果源文件与平台或者cpu架构完全匹配，那么用文件后缀，例如：

    mypkg_linux.go         // only builds on linux systems  
    
    mypkg_windows_amd64.go // only builds on windows 64bit platforms  

相反，如果这个源文件可以在超过一个平台或者超过一个cpu架构下可以使用或者需要去除指定平台，那么使用编译标签，例如下面的编译标签可以在所有*nix平台上编译：

    % grep '+build' $HOME/go/src/pkg/os/exec/lp_unix.go   
    
    // +build darwin dragonfly freebsd linux netbsd openbsd  

下面是可以在除了windows的所有平台下编译

    % grep '+build' $HOME/go/src/pkg/os/types_notwin.go   
    
    // +build !windows  

总结

这篇文章主要关注所有可以被go tool编译的go源文件,编译标签和文件后缀名（也包括了.c 和.s文件)

Go的标准库里包含了很多的样例，特别是runtime,syscall,os和net包，读者可以通过这些包来学习

Test文件也支持编译标签和文件后缀条件编译，并且作用方式与go源文件相同。可以在不同平台下有条件的包含一些测试样例。同样，标准库也包含了大量的例子

最后，这篇文件是讲如何用go tool来达到条件编译，但是条件编译不限于go tool，你可以用go/build包编写自己的条件编译工具


