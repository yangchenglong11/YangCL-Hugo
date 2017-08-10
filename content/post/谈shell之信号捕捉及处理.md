---
author : "杨承龙"
date : "2017-07-27T09:41:10+08:00"
draft : false
title : "谈shell之信号捕捉及处理"
tags : ["linux"]
comments : true     
share : true        
menu : "main" 
          
---
谈shell之信号捕捉及处理

在64位系统上，执行kill –l 命令可以看到几乎所有的信号

    [shell@u ~]$ kill -l 
    1) SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL 5) SIGTRAP 6) SIGABRT 7) SIGBUS 8) SIGFPE 9) SIGKILL 10) SIGUSR1 11) SIGSEGV 12) SIGUSR213) SIGPIPE 14) SIGALRM 15) SIGTERM 17) SIGCHLD18) SIGCONT 19) SIGSTOP 20) SIGTSTP 21) SIGTTIN22) SIGTTOU 23) SIGURG 24) SIGXCPU 25) SIGXFSZ26) SIGVTALRM 27) SIGPROF 28) SIGWINCH 29) SIGIO30) SIGPWR 31) SIGSYS 34) SIGRTMIN 35) SIGRTMIN+136) SIGRTMIN+2 37) SIGRTMIN+3 38) SIGRTMIN+4 39) SIGRTMIN+540) SIGRTMIN+6 41) SIGRTMIN+7 42) SIGRTMIN+8 43) SIGRTMIN+944) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+1348) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-1352) SIGRTMAX-12 53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-956) SIGRTMAX-8 57) SIGRTMAX-7 58) SIGRTMAX-6 59) SIGRTMAX-560) SIGRTMAX-4 61) SIGRTMAX-3 62) SIGRTMAX-2 63) SIGRTMAX-164) SIGRTMAX

之所以说“几乎”，是因为有一个信号没有列入其中——信号“0”。

在我们执行exit命令或使用ctrl + D退出一个终端时，所发送的即为信号“0”。

nohup命令在执行时即会忽略SIGHUP(信号1)，也会忽略信号0。

常用信号介绍

    1 SIGHUP 终止进程 终端断开
    
    2 SIGINT 终止进程 按ctrl+c
    
    3 SIGQUIT 终止进程 按 ctrl+\或者ctrl+D
    
    9 SIGKILL 终止进程 无法捕获或忽略
    
    15 SIGTERM 终止进程 软件终止信号
    
    20 SIGTSTP 终止进程 终端来的停止信号

既然分这么多种信号，那么这些信号的区别是什么呢？

    SIGHUP： 终端接口检测到网络连接断开，会将此信号发送给与该终端相关的控制进程（会话首进程）。会话首进程将信号传递给前台进程组，通常用此进程通知守护进程，重新读取它们的配置文件。
    
    SIGINT： 当用户按下中断键（ctrl+c）终端驱动程序产生此信号并送至前台进程组中每个进程。
    
    SIGKILL ：此信号无法被忽略或者捕获。提供了一种可以杀死任何进程的方法。
    
    我们常用kill -9 pid来杀死一个进程。程序接到此信号会立刻退出。
    
    SIGTERM ：kill命令默认发出的程序中止命令。程序正常保存数据后退出。
    
    SIGTSTP： 交互式停止信号，当用户在终端上按（ctrl+z），终端驱动程序产生此信号，送至前台进程组中的所有进程。

nohup让你的程序持续跑完

当我们每次在注销系统后，还想让脚本或程序继续运行，多数会选择nohup命令。那么使用nohup命令起的脚本为什么会让程序一直运行呢？

我们man nohup看到，nohup - run a command immune to hangups, with output to a non-tty。当我们关闭终端的时候，正常脚本或者程序会接受到SIGHUP的信号，从而停止运行。nohup命令会忽略所有挂断（SIGHUP）信号，保证程序和脚本的运行。

nohup使用方法：

nohup Command [ Arg ... ][ & ]无论是否将 nohup 命令的输出重定向到终端，输出都将附加到当前目录的nohup.out 文件中；如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中；如果没有文件能创建或打开用于输出追加，那么Command参数指定的命令不可调用。

创建的 nohup.out 或者 $HOME/nohup.out 文件没有group或者other组的访问权限。

注意：

nohup不会自动地将命令放入后台执行，需要增加一个参数&。

最好将nohup的输出重定向到/dev/null中，有用的信息通过日志保存，否则nohup.out文件日积月累，会占用硬盘空间，占用过大，可能导致服务停止~

退出状态

该命令返回下列出口值：

    126
    
    可以查找但不能调用 Command 参数指定的命令。
    
    127
    
    nohup 命令发生错误或不能查找由 Command 参数指定的命令。
    
    否则，nohup 命令的退出状态是 Command 参数指定命令的退出状态。

异常退出与执行exit——secureCRT异常退出和执行exit的区别？

如果直接关闭secureCRT（此处假设是使用ssh登录终端的），那么对于被登录的系统来说，就是远端程序异常断连。和我们突然断网掉线是一样的效果。这种情况下，用户并没有信号发送，而是sshd服务检测到对端响应超时，然后向之前建立起的连接以及该连接下（ssh登录后会分配一个bash给用户）的进程发送结束信号。如果部分进程忽略sshd发送的信号，进程不退出，在分配给用户的bash退出后，该进程将被init进程接管。

————————————————————————————————————————————————————

一. trap捕捉到信号之后，可以有三种反应方式：

(1)执行一段程序来处理这一信号

(2)接受信号的默认操作

(3)忽视这一信号

二. trap对上面三种方式提供了三种基本形式：

第一种形式的trap命令在shell接收到signal list清单中数值相同的信号时，将执行双引号中的命令串。

    trap 'commands' signal-list
    
    trap "commands" signal-list

为了恢复信号的默认操作，使用第二种形式的trap命令：

    trap signal-list

第三种形式的trap命令允许忽视信号

    trap " " signal-list

注意：

(1) 对信号11(段违例)不能捕捉，因为shell本身需要捕捉该信号去进行内存的转储。

(2) 在trap中可以定义对信号0的处理(实际上没有这个信号)， shell程序在其终止(如执行exit语句)时发出该信号。

(3) 在捕捉到signal-list中指定的信号并执行完相应的命令之后， 如果这些命令没有将shell程序终止的话，shell程序将继续执行收到信号时所执行的命令后面的命令，这样将很容易导致shell程序无法终止。

另外，在trap语句中，单引号和双引号是不同的，当shell程序第一次碰到trap语句时，将把commands中的命令扫描一遍。此时若commands是用单引号括起来的话，那么shell不会对commands中的变量和命令进行替换， 否则commands中的变量和命令将用当时具体的值来替换。



点击点击阅读原文
