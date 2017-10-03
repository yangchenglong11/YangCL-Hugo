---
title: Linux中改变文件属性与权限
date: 2016-11-26 21:52:46 
categories: Linux 
tags: [Linux] 
description: 
---

这篇文章讲了在 Linux 中怎样改变文件的属性与权限。

<!--more--> 

### chgrp

**chgrp改变文件所属用户组**

chgrp即为change group的简写，如果要改变文件的用户组，要被改变的用户组必须要在/etc/group文件中存在才行，否则就会显示错误，可以进入这个目录查看文件内容，但建议使用cat命令，不要使用vi/vim，因为一旦不慎修改了此文件，系统文件出错后果是很严重的。

改变之前：

```shell
yangs-MacBook-Air:code yang$ ls -al
yangs-MacBook-Air:code yang$ -rw-r--r--   1 yang  staff    14 Jan 12 17:01 te1
```
执行命令：

```shell
yangs-MacBook-Air:code yang$ chgrp everyone te1
```

让我们查看一下：

```shell
yangs-MacBook-Air:code yang$ ls -al
yangs-MacBook-Air:code yang$ -rw-r--r--   1 yang  everyone    14 Jan 12 17:04 te1
```

可以看到已经被修改了。

当所改用户组未在文件中时：

```shell
yangs-MacBook-Air:code yang$ chgrp eve te1
chgrp: eve: illegal group name
```

这个命令还有一个可选参数 -R,即进行递归(recursive)的持续更改，即连同子目录下的所有文件目录都更新成为这个用户组，常用在更改某一目录内所有的文件情况。

### chown

**chown改变文件所有者**

chown即为 change owner 的简写，同样的，所改变的用户也必须是在/etc/passwd这个文件中有记录的用户名才可以。

chown 也可以直接修改用户组的名称，如果要连目录内所有子目录和文件都同时修改的话，直接加上-R即可。

```shell
yangs-MacBook-Air:code yang$ chown bin te1
// 下面这个是将 te1 的所有者与用户组都改为 root
yangs-MacBook-Air:code yang$ chown root:root
```

可能你会有疑问，上述两个命令有什么用呢？最常见的例子就是复制文件给你之外的其他人时，比如使用cp命令：

```shell
yangs-MacBook-Air:code yang$ cp 源文件 目标文件
```

由于复制行为(cp)会复制执行者的属性与权限，那么对于其他人可能仍是无法修改此文件的。

### chmod

**chmod改变文件权限**

chmod 命令用来变更文件或目录的权限。

在UNIX系统家族里，文件或目录权限的控制分别以读取、写入、执行3种一般权限来区分，另有3种特殊权限可供运用。用户可以使用 chmod 指令去变更文件与目录的权限，设置方式采用文字或数字代号皆可。符号连接的权限无法变更，如果用户对符号连接修改权限，其改变会作用在被连接的原始文件。 

权限范围的表示法如下： 

```shell
- u User，即文件或目录的拥有者；
- g Group，即文件或目录的所属群组；
- o Other，除了文件或目录拥有者或所属群组之外，其他用户皆属于这个范围；
- a All，即全部的用户，包含拥有者，所属群组以及其他用户；
- r 读取权限，数字代号为“4”; w 写入权限，数字代号为“2”；
- x 执行或切换权限，数字代号为“1”； - 不具任何权限，数字代号为“0”；
- s 特殊功能说明：变更文件或目录的权限。 
```

语法 chmod (选项) (参数) 

选项 

```shell
- -c或——changes：效果类似“-v”参数，但仅回报更改的部分；
- -f或--quiet或——silent：不显示错误信息；
- -R或——recursive：递归处理，将指令目录下的所有文件及子目录一并处理；
- -v或——verbose：显示指令执行过程；
- --reference=<参考文件或目录>：把指定文件或目录的所属群组全部设成和参考文件或目录的所属群组相同；
- <权限范围>+<权限设置>：开启权限范围的文件或目录的该选项权限设置；
- <权限范围>-<权限设置>：关闭权限范围的文件或目录的该选项权限设置；
- <权限范围>=<权限设置>：指定权限范围的文件或目录的该选项权限设置； 
```
    
参数 

```shell
- 权限模式：指定文件的权限模式； 
- 文件：要改变权限的文件。
``` 

---

知识扩展: Linux 用户分为：拥有者、组群(Group)、其他（other），Linux 系统中，预设的情況下，系统中所有的帐号与一般身份使用者，以及 root 的相关信 息， 都是记录在/etc/passwd文件中。每个人的密码则是记录在/etc/shadow文件下。 此外，所有的组群名称记录在/etc/group內。

---

例：rwx　rw-　r-- 　

r 为读取属性　　  // 值＝4 

w 为写入属性　　 // 值＝2 

x 为执行属性　　  // 值＝1　

```shell
chmod u+x,g+w f01　　//为文件f01设置自己可以执行，组员可以写入的权限 

chmod u=rwx,g=rw,o=r f01

chmod 764 f01

chmod a+x f01　　//对文件f01的u,g,o都设置可执行属性 文件的属主和属组属性设置 

chown user:market f01　　//把文件f01给uesr，添加到market组 ll -d f1 查看目录f1的属性
```
    




