---
author : "杨承龙"
date : "2017-07-14T10:05:03+08:00"
draft : false
title : "postman"
tags : ["tools"]
comments : true     
share : true        
menu : "main" 
          
---
postman

如何测试接口-->http接口

需要Http请求模拟工具，现在流行的这种工具也挺多的，像火狐浏览器插件-RESTClient，Chrome浏览器插件-Postman等等。这里主要介绍一下Postman。 

Postman说明

　　Postman是一种网页调试与发送网页http请求的chrome插件。我们可以用来很方便的模拟get或者post或者其他方式的请求来调试接口。

使用

可以到谷歌上下载，下载安装后打开，可以看到，界面处有一个长条框，在里面输入需要测试的接口地址，选择请求方式，然后在下面手动添加相应的键值。

（1）接口请求报文拼接

url?param=value&param2=value

这种是最简单的一种，问号前面是请求url，后面是请求的参数名和参数值，多个参数用&来连接

https://api.douban.com/v2/book/search?q=zouweiwei

（2）还有一种就是入参是json串的，那就不用拼接参数了，借助postman来实现，下面会举例说明

（3）GET和POST请求：

如果是get请求的话，直接在浏览器里输入就行了，只要在浏览器里面直接能请求到的，都是get请求，如果是post请求的话，就不行了，就得借助工具来发送。

GET和POST请求的区别：

- GET使用URL或Cookie传参，而POST将数据放在Body中；
- GET的URL会有长度上的限制，而POST的数据则可以非常大；
- POST比GET安全，因为数据在地址栏上不可见；
- 一般get请求用来获取数据，post请求用来发送数据。

（4）body部分编辑分为4个部分：

form-data是web表单默认的传输格式，编辑器允许你通过设置key-value形式的数据来模拟填充表单。你可以在最后的选项中选择添加文件。

urlencoded这个编码格式同样可以通过设置key-value的方式作为URL的参数。

raw：一个raw请求可以包含任何内容。在这里你可以设置我们常用的JSON 和 XML数据格式。

binary：在这里你可以发送视频、音频、文本等文件

（5）Headers

使用拦截器来发送这些受限的headers和cookies

（6）Authorization

身份验证，后边会有用法介绍

3.点击Send即可提交请求，然后在下面查看请求结果，并且可以以Pretty、Raw、Preview三种方式查看

Pretty方式，可以让JSON 和 XML的响应内容显示的更美观规整。

Raw方式，显示最原始的数据，可以帮助你判断是否minified。

Preview方式，可以帮你把HTML页面自动解析显示出来。

HTTP状态码：每发出一个http请求之后，就会有一个响应，http本身会有一个状态码，来标示这个请求是否成功，常见状态码：

         200，2开头的都表示这个请求发送成功，最常见的就是200

         300，3开头的代表重定向，最常见的是302，把这个请求重定向到别的地方了

         400，400代表客户端发送的请求有语法错误，401代表访问的页面没有授权，403代表没有权限访问这个页面，404代表没有这个页面

         500，5开头的代表服务器有异常，500代表服务器内部异常，504代表服务器端超时，没返回结果

 
