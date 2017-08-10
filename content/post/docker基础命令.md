---
author : "杨承龙"
date : "2017-05-12T13:48:24+08:00"
draft : false
title : "Docker基本命令"
tags : ["go"]
comments : true     
share : true        
menu : "main" 
          
---

创建mongo容器，并在其他端口启动

``docker run --name clmongo -p 6667:27017 -d mongo``

创建mongo容器，将数据放在 /home/docker/clmongo_data目录下，在本机27017端口启动，在外部通过6667访问

``docker run --name clmongo -v /home/docker/clmongo_data:/data/db -d -p 6667:27017 mongo``

获取镜像，通过制定tag可以下载特定标签的镜像。比如
``docker pull image_name:tag``

未指定tag时，将会默认下载标签为latest标签的镜像

``sudo docker pull ubuntu:14.04``

使用镜像

``sudo docker run -t -i ubuntu /bin``

查看镜像信息

``docker imges``

使用docker tag为本地镜像添加新的标签, 添加后可以发现多了一个所添加标签的镜像，但它们标签一致，即标签在这里起到了引用或快捷方式的作用。

``sudo docker tag name:tag new_name:new_tag``

显示 Docker 系统信息，包括镜像和容器数。

``docker info``

使用docker inspect查看该镜像的详细信息

``sudo docker inspect image_id``

查询镜像

``sudo docker search name``

删除镜像

``sudo docker mi``


