---
author : "杨承龙"
date : "2017-07-22T10:57:51+08:00"
draft : false
title : "mgo断线重连"
tags : ["MongoDB"]
comments : true     
share : true        
menu : "main" 
          
---
mgo自动重连

使用 mgo 时，当远端的 session 挂掉, 其实底层已经做了重连的机制，但是没有通知上层的 session 来更新当前的 session，而致使当前的 session 不可以用。所以为了实现断线重连，可以使用下面两种方法：

- 不只是使用一个 master session ，而是通过调用 session.Copy() 来创建多个session。每当需要处理独立请求时，通过调用 Copy() 来为每个请求创建一个独立的 session。每当需要一个新 session 时，都会进行 Refresh , 然后在主 session 上进行复制，接着使用该 session 完成操作，使用后将其关闭。当多个任务同时进行时，会创建多个连接，mgo 对连接数作了限制，默认配置的连接数上限是4096，当然一般情况下是足够使用的，当然为了实现更高要求可以将其设置的更大些。此外，调用 Copy() 是非常 cheap 的，不用担心因调用而产生的资源消耗；
- 仍是用一个 session ，但在每次调用 session 时，调用 Session.Refresh() 方法，该方法会更新 session 状态，这样就可以和远端 session 状态保持一致。(如果觉得每次都要 refresh 有些麻烦，也可以在调用函数出错时，对错误进行判断，因mgo断线而抛出的错误是 io.EOF，如果错误类型为 io.EOF，则调用 Session.Refresh() 方法重新进行操作，但mgo断线也可能抛出其他错误，采用上面的方法更为保险)。
