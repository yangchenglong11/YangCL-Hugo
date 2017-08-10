---
author : "杨承龙"
date : "2017-06-21T13:48:24+08:00"
draft : false
title : "iptables"
tags : ["linux"]
comments : true     
share : true        
menu : "main" 
          
---
# iptables

## Netfilter 与 iptables 的关系

Linux 系统在内核中提供了对报文数据包过滤和修改的官方项目名为 Netfilter，它指的是 Linux 内核中的一个框架，它可以用于在不同阶段将某些钩子函数（hook）作用域网络协议栈。Netfilter 本身并不对数据包进行过滤，它只是允许可以过滤数据包或修改数据包的函数挂接到内核网络协议栈中的适当位置。这些函数是可以自定义的。

iptables 是用户层的工具，它提供命令行接口，能够向 Netfilter 中添加规则策略，从而实现报文过滤，修改等功能。Linux 系统中并不止有 iptables 能够生成防火墙规则，其他的工具如 firewalld 等也能实现类似的功能。

## 使用 iptables 进行包过滤

iptables利用的是数据包过滤机制，他会分析数据包的报头数据，根据报头数据与定义的规则来决定该数据包是否可以进入主机或者是被丢弃。也就是说，根据数据包的分析资料对比预先定义的规则内容，若数据包数据数据与规则内容相同则进行动作，否则就继续下一条规则的对比。重点是对比与分析的顺序。

![tu](/Users/yang/mysite/content/post/images/iptables_01.png)



  在上图中，当网络数据包开始Rule 1 的比对时，如果比对结果符合Rule 1，此时这个网络数据包就会进行Action 1 的动作，而不会理会后续的 Rule 2，Rule 3等规则了。如果一个一个规则对比下去，所有规则都不符合的话， 就会执行默认操作(Policy)，来决定这个数据包的去向。

### 表

上图中列出的规则仅是iptables众多表格当中的一个链而已。iptables中有多个表格，每个表都定义自己的默认策略与规则。

iptables 根据功能分类，内建有多个表，如包过滤（filter）或者网络地址转换（NAT）。iptables 中共有 4 个表：filter，nat，mangle 和 raw。filter 表主要实现过滤功能，nat 表实现 NAT 功能，mangle 表用于修改分组数据，raw 用于配置数据包，raw中的数据包不会被系统跟踪。

### 链

一个 iptables 链就是一个规则集，这些规则按序与包含某种特征的数据包进行比较匹配。

每个表都有一组内置链，用户还可以添加自定义的链。最重要的内置链是 filter 表中的 INPUT、OUTPUT 和 FORWARD 链。

* filter,用于路由网络数据包
  + INPUT 发往本机的报文
  + OUTPUT 由本机发出的报文
  + FORWARD 经由本机转发的报文
* nat,用于NAT表。这个表主要用来进行来源与目的地的IP或port的转换，与linux本机无关，主要与linux主机后的局域网内计算机相关。
  + PREROUTING 网络数据包到达服务器时可以被修改
  + POSTROUTING 网络数据包在即将从服务器发出时可以被修改
  + OUTPUT 网络数据包流出服务器
* mangle,用于修改网络数据包的表，如TOS(Type Of Service),TTL(Time To Live),等
  + INPUT 发往本机的报文
  + OUTPUT 由本机发出的报文
  + FORWARD 经由本机转发的报文
  + PREROUTING 网络数据包到达服务器时可以被修改
  + POSTROUTING 网络数据包在即将从服务器发出时可以被修改
* raw, 用于决定数据包是否被跟踪机制处理
  + OUTPUT 由本机发出的报文
  + PREROUTING 网络数据包到达服务器时可以被修改

3.数据包过滤匹配流程

​	1>.规则表之间的优先顺序

​	依次应用：raw、mangle、nat、filter表

​	2>.规则链之间的优先顺序

​		入站数据流向

​		转发数据流向

​		出站数据流向

​	3>.规则链内部各条防火墙规则之间的优先顺序

所以如果liunx是作为服务器，就要让客户端可以访问你的服务，就得要处理 filter 的 INPUT 链； 而如果你的 Linux 是作为局域网络的路由器，那么就得要分析 nat 的各个链以及 filter 的 FORWARD 链才行。也就是说， 其实各个表格的链结之间是有关系的。



![tu](/Users/yang/mysite/content/post/images/iptables_02.png)



 iptables 可以控制三种封包的流向：

- 封包进入 Linux 主机使用资源 (路径 A)： 在路由判断后确定是向 Linux 主机要求数据的封包，主要就会透过 filter 的 INPUT 链来进行控管；
- 封包经由 Linux 主机的转递，没有使用主机资源，而是向后端主机流动 (路径 B)： 在路由判断之前进行封包表头的修订作业后，发现到封包主要是要透过防火墙而去后端，此时封包就会透过路径 B 来跑动。 也就是说，该封包的目标并非我们的 Linux 本机。主要经过的链是 filter 的 FORWARD 以及 nat 的 POSTROUTING, PREROUTING。
- 封包由 Linux 本机发送出去 (路径 C)： 例如响应客户端的要求，或者是 Linux 本机主动送出的封包，都是透过路径 C 来跑的。先是透过路由判断， 决定了输出的路径后，再透过 filter 的 OUTPUT 链来传送。当然，最终还是会经过 nat 的 POSTROUTING 链。

iptables 至少有三个预设的 table (filter, nat, mangle)，较常用的是本机的 filter 表格， 也是默认表格。另一个则是后端主机的 nat 表格，至于 mangle 较少使用，所以这里我们并不会讨论 mangle。 由于不同的 table 他们的链不一样，导致使用的指令语法或多或少都有点差异。 这里，我们主要将针对 filter 这个默认表格的三条链来做介绍。

#### 显示当前规则

```shell
[root@www ~]# iptables [-t tables] [-L] [-nv]
选项与参数：
-t ：后面接 table ，例如 nat 或 filter ，若省略此项目，则使用默认的 filter
-L ：列出目前的 table 的规则
-n ：不进行 IP 与 HOSTNAME 的反查，显示讯息的速度会快很多
-v ：列出更多的信息，包括通过该规则的封包总位数、相关的网络接口等

范例：列出 filter table 三条链的规则
[root@www ~]# iptables -L -n
Chain INPUT (policy ACCEPT)                                       <==INPUT 链，预设为可接受
target  prot opt source     destination                           <==说明栏
ACCEPT  all  --  0.0.0.0/0  0.0.0.0/0   state RELATED,ESTABLISHED <==第 1 条规则
ACCEPT  icmp --  0.0.0.0/0  0.0.0.0/0                             <==第 2 条规则
ACCEPT  all  --  0.0.0.0/0  0.0.0.0/0                             <==第 3 条规则
ACCEPT  tcp  --  0.0.0.0/0  0.0.0.0/0   state NEW tcp dpt:22      <==以下类推
REJECT  all  --  0.0.0.0/0  0.0.0.0/0   reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)  <==针对 FORWARD 链，且预设政策为可接受
target  prot opt source     destination
REJECT  all  --  0.0.0.0/0  0.0.0.0/0   reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)  <==针对 OUTPUT 链，且预设政策为可接受
target  prot opt source     destination

范例：列出 nat table 三条链的规则
[root@www ~]# iptables -t nat -L -n
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
在上表中，每一个 Chain 就是前面提到的每个链， Chain 那一行里面括号的 policy 就是预设的政策， 那底下的 target, prot 释义如下：

- target：代表进行的动作， ACCEPT 是放行，而 REJECT 则是拒绝，此外，也有 DROP (丢弃) 的项目；
- prot：代表使用的封包协议，主要有 tcp, udp 及 icmp 三种封包格式；
- opt：额外的选项说明；
- source ：代表此规则是针对哪个“来源 IP”进行限制；
- destination ：代表此规则是针对哪个“目标 IP”进行限制。

在输出结果中，第一个范例因为没有加上 -t 的选项，所以默认就是 filter 这个表格内的 INPUT, OUTPUT, FORWARD 三条链的规则。若针对单机来说，INPUT 与 FORWARD 算是比较重要的管制防火墙链， 所以你可以发现最后一条规则的政策是 REJECT (拒绝) 。虽然 INPUT 与 FORWARD 的政策是放行 (ACCEPT)， 不过在最后一条规则就已经将全部的封包都拒绝了。

不过这个指令的观察只是作个格式化的查阅，要详细解释每个规则会比较不容易解析。举例来说， 我们将 INPUT 的 5 条规则依据输出结果来说明一下，结果会变成：

1. 只要是封包状态为 RELATED,ESTABLISHED 就予以接受
2. 只要封包协议是 icmp 类型的，就予以放行
3. 无论任何来源 (0.0.0.0/0) 且要去任何目标的封包，不论任何封包格式 (prot 为 all)，通通都接受
4. 只要是传给 port 22 的主动式联机 tcp 封包就接受
5. 全部的封包信息通通拒绝

可以看下第 3 条规则，怎么会所有的封包信息都予以接受，如果都接受的话，那么后续的规则根本就不会有用了。 其实那条规则是仅针对每部主机都有的内部循环测试网络 (lo) 接口。如果没有列出接口，那么我们就很容易搞错。 所以，建议使用 iptables-save 这个指令来观察防火墙规则。因为 iptables-save 会列出完整的防火墙规则，只是并没有规格化输出而已。

```shell
[root@www ~]# iptables-save [-t table]
选项与参数：
-t ：可以仅针对某些表格来输出，例如仅针对 nat 或 filter 等等

[root@www ~]# iptables-save
# Generated by iptables-save v1.4.7 on Fri Jul 22 15:51:52 2011
*filter                      <==星号开头的指的是表格，这里为 filter
:INPUT ACCEPT [0:0]          <==冒号开头的指的是链，三条内建的链
:FORWARD ACCEPT [0:0]        <==三条内建链的政策都是 ACCEPT 
:OUTPUT ACCEPT [680:100461]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT <==针对 INPUT 的规则
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT  <==这条很重要，针对本机内部接口开放
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited <==针对 FORWARD 的规则
COMMIT
# Completed on Fri Jul 22 15:51:52 2011
```

其中，每个表都包含链和规则，链的详细说明是:<chain-name> <chain-policy> [<packet-counter>:<byte-counter>]，即[<packet-counter>:<byte-counter>]表示该链已经匹配了多少个 [包，字节]

由上面的输出来看，在内容含有 lo 的那条规则当中，" -i lo" 指的就是由 lo 适配卡进来的封包, 这样看就清楚多了。 不过，既然这个规则不是我们想要的，下面看下如何修改规则

#### 清除规则:

```shell
[root@www ~]# iptables [-t tables] [-FXZ]
选项与参数：
-F ：清除所有的已订定的规则；
-X ：杀掉所有使用者 "自定义" 的 chain ；
-Z ：将所有的 chain 的计数与流量统计都归零

范例：清除本机防火墙 (filter) 的所有规则
[root@www ~]# iptables -F
[root@www ~]# iptables -X
[root@www ~]# iptables -Z
```

这三个指令会将本机防火墙的所有规则都清除，但却不会改变预设政策(policy)

#### 定义预设   (policy)

预设规则指当您的数据包不在您设定的规则之内时，则该包的通过与否，以 Policy 的设定为准

```shell
[root@www ~]# iptables [-t nat] -P [INPUT,OUTPUT,FORWARD] [ACCEPT,DROP]
选项与参数：
-P ：定义政策( Policy )。注意，这个 P 为大写
ACCEPT ：该封包可接受
DROP   ：该封包直接丢弃，不会让 client 端知道为何被丢弃。

范例：将本机的 INPUT 设定为 DROP ，其他设定为 ACCEPT
[root@www ~]# iptables -P INPUT   DROP
[root@www ~]# iptables -P OUTPUT  ACCEPT
[root@www ~]# iptables -P FORWARD ACCEPT
[root@www ~]# iptables-save
# Generated by iptables-save v1.4.7 on Fri Jul 22 15:56:34 2011
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
COMMIT
# Completed on Fri Jul 22 15:56:34 2011
```

#### 数据包的基础比对：IP, 网域及接口装置

```shell
[root@www ~]# iptables [-AI 链名] [-io 网络接口] [-p 协议] \
> [-s 来源IP/网域] [-d 目标IP/网域] -j [ACCEPT|DROP|REJECT|LOG]
选项与参数：
-AI 链名：针对某的链进行规则的 "插入" 或 "累加"
    -A ：新增加一条规则，该规则增加在原本规则的最后面。例如原本已经有四条规则，
         使用 -A 就可以加上第五条规则！
    -I ：插入一条规则。如果没有指定此规则的顺序，默认是插入变成第一条规则。
         例如原本有四条规则，使用 -I 则该规则变成第一条，而原本四条变成 2~5 号
    链 ：有 INPUT, OUTPUT, FORWARD 等，此链名称又与 -io 有关，请看底下。

-io 网络接口：设定封包进出的接口规范
    -i ：封包所进入的那个网络接口，例如 eth0, lo 等接口。需与 INPUT 链配合；
    -o ：封包所传出的那个网络接口，需与 OUTPUT 链配合；

-p 协定：设定此规则适用于哪种封包格式
   主要的封包格式有： tcp, udp, icmp 及 all 。

-s 来源 IP/网域：设定此规则之封包的来源项目，可指定单纯的 IP 或包括网域，例如：
   IP  ：192.168.0.100
   网域：192.168.0.0/24, 192.168.0.0/255.255.255.0 均可。
   若规范为"不许"时，则加上 ! 即可，例如：
   -s ! 192.168.100.0/24 表示不许 192.168.100.0/24 的封包来源；

-d 目标 IP/网域：同 -s ，只不过这里指的是目标的 IP 或网域。

-j ：后面接动作，主要的动作有接受(ACCEPT)、丢弃(DROP)、拒绝(REJECT)及记录(LOG)
```

iptables 的基本参数就如同上面所示的，仅只谈到 IP 、网域与接口装置等等的信息， 至于 TCP, UDP  包特有的端口与状态则在下小节才会谈到。 好，先让我们来看看最基本的几个规则，例如开放 lo 这个本机的接口以及某个 IP 来源。

```shell
范例一:所有的来自 lo 这个接口的数据包，都予以接受
[root@linux ~]# iptables -A INPUT -i lo -j ACCEPT
# 仔细看上面并没有列出 -s, -d 等等的规则，这表示:不论本包来自何处或去到哪里， 
# 只要是来自 lo 这个接口，就予以接受。

范例：只要是来自内网的 (192.168.100.0/24) 的封包通通接受
[root@www ~]# iptables -A INPUT -i eth1 -s 192.168.100.0/24 -j ACCEPT

范例：只要是来自 192.168.100.10 就接受，但 192.168.100.230 这个恶意来源就丢弃
[root@www ~]# iptables -A INPUT -i eth1 -s 192.168.100.10 -j ACCEPT
[root@www ~]# iptables -A INPUT -i eth1 -s 192.168.100.230 -j DROP

[root@www ~]# iptables-save
# Generated by iptables-save v1.4.7 on Fri Jul 22 16:00:43 2011
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [17:1724]
-A INPUT -i lo -j ACCEPT
-A INPUT -s 192.168.100.0/24 -i eth1 -j ACCEPT
-A INPUT -s 192.168.100.10/32 -i eth1 -j ACCEPT
-A INPUT -s 192.168.100.230/32 -i eth1 -j DROP
COMMIT
# Completed on Fri Jul 22 16:00:43 2011
```

#### 记录某个规则的日志

```shell
[root@linux ~]# iptables -A INPUT -s 192.168.2.200 -j LOG
 
[root@linux ~]# iptables -L -n
target prot opt source         destination
LOG    all  --  192.168.2.200  0.0.0.0/0   LOG flags 0 level 4
```


看到输出结果的最左边，会出现的是 LOG 。只要有 包来自 192.168.2.200 这个 IP 时， 那么该 包
的相关信息就会被写入到日志中，亦即是 /var/log/messages 这个文件当中。LOG 这个动作仅在进行记录而已，并不会影响到这个数据包的其它规则比对的。

#### TCP, UDP 的规则比对

```shell
[root@linux ~]#  iptables [-AI 链] [-io 网络接口] [-p tcp,udp] 
> [-s 来源IP/网域] [--sport 端口范围] 
> [-d 目标IP/网域] [--dport 端口范围] -j [ACCEPT|DROP|REJECT]

参数:
选项与参数：
--sport 端口范围：限制来源的端口号码，端口号码可以是连续的，例如 1024:65535
--dport 端口范围：限制目标的端口号码。
```


事实上就是多了那个 --sport 及 --dport 这两个选项，重点在那个 port number， 仅有 tcp 与 udp 封包具有端口，因此你想要使用 --dport, --sport 时，得要加上 -p tcp 或 -p udp 的参数才会成功。底下让我们来进行几个小测试:

```shell
范例一:想要联机进入本机 port 21 的数据包都丢掉:
[root@linux ~]# iptables -A INPUT -i eth0 -p tcp --dport 21 -j DROP
范例二:想连到我这部主机的(upd port 137,138 tcp port 139,445) 就放行 
[root@linux ~]# iptables -A INPUT -i eth0 -p udp --dport 137:138 -j ACCEPT 
[root@linux ~]# iptables -A INPUT -i eth0 -p tcp --dport 139 -j ACCEPT 
[root@linux ~]# iptables -A INPUT -i eth0 -p tcp --dport 445 -j ACCEPT
```

你可以利用 UDP 与 TCP 协议所拥有的端口号码来进行某些服务的开放或关闭。

例如:只要来自 192.168.1.0/24 的 1024:65535 端口的数据包，只要想要联机到本机的 ssh port 就予以抵挡，可以这样做:

```shell
[root@linux ~]# iptables -A INPUT -i eth0 -p tcp -s 192.168.1.0/24 \
  --sport 1024:65534 --dport ssh -j DROP
```

如果你有使用到 --sport 及 --dport 的参数时，忘了指定 -p tcp 或 -p udp，就会出现如下的错误:

```shell
[root@k8s ~]# iptables -A INPUT -i eth0 --dport 21 -j DROP
iptables v1.4.21: unknown option "--dport"
Try `iptables -h' or 'iptables --help' for more information.			
```

在 iptables里面还支持 --syn 的处理方式，我们以底下的例子来说明:

```shell
范例:将来自任何地方来源 port 1:1023 的主动联机到本机端的 1:1023  联机丢弃 
[root@linux ~]# iptables -A INPUT -i eth0 -p tcp --sport 1:1023  \
  --dport 1:1023 --syn -j DROP
```

#### 状态模块:MAC 与 STATE

```shell
[root@linux ~]# iptables -A INPUT [-m state] [--state 状态]
选项与参数：
-m ：一些 iptables 的外挂模块，主要常见的有：
     state ：状态模块
     mac   ：网络卡硬件地址 (hardware address)
--state ：一些数据包的状态，主要有：
     INVALID    ：无效的数据包，例如数据破损的数据包状态
     ESTABLISHED：已经联机成功的联机状态；
     NEW        ：想要新建立联机的数据包状态；
     RELATED    ：这个最常用，表示这个数据包是与我们主机发送出去的数据包有关

范例:只要已建立或相关数据包就予以通过，只要是不合法数据包就丢弃
[root@linux ~]# iptables -A INPUT -m state \
--state RELATED,ESTABLISHED -j ACCEPT
[root@linux ~]# iptables -A INPUT -m state --state INVALID -j DROP
```

所以说，如果你的 Linux 主机只想要作为 client 的用途，不许所有主动对你联机的来源， 那么你可以这样做即可:

1. 清除所有已经存在的规则 (iptables -F...)
2. 设定预设  ，除了 INPUT 预设为 DROP 其它为预设 ACCEPT;
3. 开放本机的 lo 可以自由放行;
4. 设定有相关的数据包状态可以联机进入本机。

你可以在某个 script 上面这样做即可:

```shell
#!/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin  export PATH
iptables -F
iptables -X
iptables -Z 
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
#iptables -A INPUT -i eth0 -s 192.168.1.0/24 -j ACCEPT
```

那如果局域网络内有其它的主机时，再将上表最后一行的 # 取消，就可以接受来自本地 LAN 的其它主机的联机了。 如果你担心某些 LAN 内的恶意来源主机会主动的对你联机时，那你还可以针对信任的本地端主机的 MAC 进行过滤。这次的状态则是 MAC 的比对。举例来说:

```shell
范例一:针对 域网络内的 aa:bb:cc:dd:ee:ff 主机开放其联机
[root@linux ~]# iptables -A INPUT -m mac --mac-source aa:bb:cc:dd:ee:ff  \
  -j ACCEPT
参数:
--mac-source :是来源主机的 MAC .
```

#### ICMP  包规则的比对

在网络基 的 ICMP 协议当中我们知道 ICMP 的格式相当的多，而且很多 ICMP  包的类型格式都是为了
要用来进行网络 测用的。所以最好不要将所有的 ICMP 数据包都丢弃。通常我们会把 ICMP type 8 (echo
request) 拿掉而已，让远程主机不知道我们是否存在，也不会接受 ping 的响应就是了。ICMP  包格式
的处理是这样的:

```shell
[root@linux ~]# iptables -A INPUT [-p icmp] [--icmp-type 类型] -j ACCEPT

选项与参数：
--icmp-type ：后面必须要接 ICMP 的数据包类型，也可以使用代号，
              例如 8  代表 echo request 的意思。
#范例:让 0,3,4,11,12,14,16,18 的 ICMP type 可以进入 机: 
[root@linux ~]# vi somefile
#!/bin/bash
icmp_type= 0 3 4 11 12 14 16 18 
for typeicmp in  icmp_type
do
   iptables -A INPUT -i eth0 -p icmp --icmp-type  typeicmp -j ACCEPT
done
[root@linux ~]# sh  somefile
```

这样就能够开放部分的 ICMP 数据包格式进入本机进行网络检测的工作了。



