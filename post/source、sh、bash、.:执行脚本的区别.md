---
author : "杨承龙"
date : "2017-06-14T12:47:31+08:00"
draft : false
title : "执行脚本的区别"
tags : ["linux"]
comments : true     
share : true        
menu : "main" 
          
---
## source、sh、bash、./执行脚本的区别

### sh文件介绍

.sh为Linux的脚本文件，我们可以通过.sh执行一些命令，可以理解为windows的.bat批处理文件。

## source命令用法：　　

```shell
source FileName　　
```

作用:在**当前bash环境下**读取并执行**FileName中**的命令。

是在当前shell执行脚本里面的命令，不需要执行权限，有读取权限（r权限）即可  

注：该命令通常用命令“.”来替代。    

如：

```shell
$ source ./bash_profile        

$ . ./bash_profile

```

两者等效。    

source(或点)命令通常用于重新执行刚修改的初始化文档。  

source命令(从 C Shell 而来)是bash shell的内置命令。    

点命令，就是个点符号，(从Bourne Shell而来)。 

## sh和bash命令用法：     

```shell
sh FileName     

bash FileName  
```

作用:在**当前bash环境下**读取并执行**FileName中**的命令。

是新建一个shell执行脚本里面的命令，不需要执行权限，有读取权限（r权限）即可，

 注：两者在执行文件时的不同，是分别用自己的shell来跑文件。   

 sh使用“-n”选项进行shell脚本的语法检查，使用“-x”选项实现shell脚本逐条语句的跟踪，   可以巧妙地利用shell的内置变量增强“-x”选项的输出信息等。 

## ./的命令用法：     

```shell
./FileName     
```

作用:打开一个**子shell**来读取并执行FileName中命令。      

注：运行一个shell脚本时会启动**另一个**命令解释器.  

把test.sh当成一个文件执行，这时候我们需要拥有test.sh的运行权限（x权限）       

每个shell脚本有效地运行在父shell([parent](http://www.linuxsir.org/main/doc/abs/abs3.7cnhtm/internal.html#FORKREF) shell)的一个子进程里.            

这个父shell是指在一个控制终端或在一个*xterm*窗口中给你命令指示符的进程.         shell脚本也可以启动他自已的子进程.            

这些子shell(即子进程)使脚本并行地，有效率地地同时运行脚本内的多个子任务. 

shell的嵌入命令：

: 空，永远返回为true

.   从当前shell中执行操作

break 退出for、while、until或case语句
cd 改变到当前目录
continue 执行循环的下一步
echo 反馈信息到标准输出
eval 读取参数，执行结果命令
exec 执行命令，但不在当前shell
exit 退出当前shell
export 导出变量，使当前shell可利用它
pwd 显示当前目录
read 从标准输入读取一行文本
readonly 使变量只读
return 退出函数并带有返回值
set 控制各种参数到标准输出的显示
shift 命令行参数向左偏移一个
test 评估条件表达式
times 显示shell运行过程的用户和系统时间
trap 当捕获信号时运行指定命令
ulimit 显示或设置shell资源
umask 显示或设置缺省文件创建模式
unset 从shell内存中删除变量或函数
wait 等待直到子进程运行完毕 

## 示例

假如有一个文件test.sh，脚本内容如下

```shell
#!/bin/bash
echo "step 1 sleeping"
sleep 200
echo "step 2 sleeping"
sleep 200
```

那么，现在按以下4种方式执行：

1）./test.sh

2）sh test.sh

3）. test.sh

4）source test.sh

他们有何区别？

1）第一种方式，当我们在执行此命令时，有2个新进程在运行，一个是test.sh，一个是sleep，如果我们在执行第一个sleep时按ctrl+c终止脚本，test.sh和sleep一起终止，并且第二个sleep不会执行，因为整个test.sh运行已经终止。

2）第二种方式，在执行此命令时，有2个新进程在运行，一个是bash，一个是sleep，如果执行第一个sleep时按ctrl+c，bash被终止，结果和第一种方式一样，第二个sleep不会执行。

3）第三种方式，在执行此命令时，只有一个新进程在运行，就是sleep，如果在执行第一个sleep时按ctrl+c终止，那么第二个sleep接着运行，直到脚本所有命令执行完。

4）第四种方式和第三种方式一致。