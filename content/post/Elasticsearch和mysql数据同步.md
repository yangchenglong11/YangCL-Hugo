---
author : "杨承龙"
date : "2017-08-04T14:42:48+08:00"
draft : false
title : "Elasticsearch和mysql数据同步"
tags : ["elk"]
comments : true     
share : true        
menu : "main" 
          
---
# Elasticsearch和mysql数据同步

**1、介绍**

​     对mysql、oracle等数据库数据进行同步到ES有三种做法：一个是通过elasticsearch提供的API进行增删改查，一个就是通过中间件进行数据全量、增量的数据同步，另一个是通过收集日志进行同步。

​     明显通过API增上改查比较麻烦，这里介绍的是利用中间件进行数据同步。

 

**2、常用的同步中间件的介绍和对比**

（1）elasticsearch-jdbc独立的第三方工具 [https://github.com/jprante/elasticsearch-jdbc](https://github.com/jprante/elasticsearch-jdbc) 
（2）elasticsearch-river-mysql [https://github.com/scharron/elasticsearch-river-mysql](https://github.com/scharron/elasticsearch-river-mysql) 
（3）go-mysql-elasticsearch（国内） [https://github.com/siddontang/go-mysql-elasticsearch](https://github.com/siddontang/go-mysql-elasticsearch)

 

都可以完成数据同步；

elasticsearch-jdbc更通用，GitHub活跃度很高；

elasticsearch-river-mysql 自2013年后便不再更新；

go-mysql-elasticsearch仍处理开发不稳定阶段；

elasticsearch-river-jdbc和elasticsearch-river-mysql都不支持对删掉的数据进行同步，go-mysql-elasticsearch希望可以改善这个问题。

总的来说，elasticsearch-jdbc更适合使用，对于删掉的数据可以采用API进行同步，或者在数据中不进行物理删除可以避免该问题的出现。

 

**3、elasticsearch的安装**

这里使用的是2.3.2版本，可以到官方网站下载，这里不提供官方地址，或者访问 [http://download.csdn.net/detail/carboncomputer/9648227](http://download.csdn.net/detail/carboncomputer/9648227) 下载本篇文章所用到的两个安装包。

得到**elasticsearch-2.3.2.tar.gz**

** **

[zsz@zsz ~]$ tar -zxvf elasticsearch-2.3.2.tar.gz

[zsz@zsz ~]$ mv elasticsearch-2.3.2 /usr/local/elasticsearch-2.3.2

 

**启动elasticsearch服务**

[zsz@zsz ~]$./bin/elasticsearch

另外，bin/elasticsearch -d(后台运行)；

如何需要修改配置，可以查看/elasticsearch-2.3.2/config/elasticsearch.yml；

 

**查看节点情况：**

[zsz@zsz downloads]$ curl 'localhost:9200/_cat/nodes?v'
host      ip        heap.percent ram.percent load node.role master name  
127.0.0.1 127.0.0.1           12          79 0.18 d         *      node-1

 

**查看索引，当前为无索引：**

[zsz@zsz downloads]$  curl 'localhost:9200/_cat/indices?v'
health status index pri rep docs.count docs.deleted store.size pri.store.size 

 

**创建索引：**

[zsz@zsz downloads]$ curl -XPUT 'localhost:9200/customer?pretty'
​            {
​              "acknowledged" : true
​            }

 

[zsz@zsz downloads]$  curl 'localhost:9200/_cat/indices?v'
health status index    pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer   5   1          0            0       650b           650b 

 

**增加索引并搜索：**

[zsz@zsz downloads]$ curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
\>         {
\>           "name": "John Doe"
\>         }'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "_shards" : {
​    "total" : 2,
​    "successful" : 1,
​    "failed" : 0
  },
  "created" : true
}

可以看到，一个新的文档在customer索引和external类型中被成功创建。文档也有一个内部id 1， 这个id是在增加索引的时候指定的。下面来检索这个记录：

[zsz@zsz downloads]$ curl -XGET 'localhost:9200/customer/external/1?pretty'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
​    "name" : "John Doe"
  }
}

[zsz@zsz downloads]$ curl 'localhost:9200/customer/_search?q=John'
{"took":2,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":1,"max_score":0.19178301,"hits":[{"_index":"customer","_type":"external","_id":"1","_score":0.19178301,"_source":
​        {
​          "name": "John Doe"
​        }}]}}

 

**更新这个索引：**

[zsz@zsz downloads]$ curl -XPOST 'localhost:9200/customer/external/1/_update?pretty' -d '
\>         {
\>           "doc": { "name": "Jane Doe Haha" }
\>         }'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 2,
  "_shards" : {
​    "total" : 2,
​    "successful" : 1,
​    "failed" : 0
  }
}

[zsz@zsz downloads]$ curl -XGET 'localhost:9200/customer/external/1?pretty'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 2,
  "found" : true,
  "_source" : {
​    "name" : "Jane Doe Haha"
  }
}

 

**删除该索引：**

[zsz@zsz downloads]$ curl -XDELETE 'localhost:9200/customer/external/1?pretty'
{
  "found" : true,
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 3,
  "_shards" : {
​    "total" : 2,
​    "successful" : 1,
​    "failed" : 0
  }
}
[zsz@zsz downloads]$ curl -XGET 'localhost:9200/customer/external/1?pretty'
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "found" : false
}

**ES必要的插件**
必要的Head、kibana、IK（中文分词）、graph等插件的详细安装和使用。
[http://blog.csdn.net/column/details/deep-elasticsearch.html](http://blog.csdn.net/column/details/deep-elasticsearch.html)

 

**ES对外接口**
JAVA API接口
http://www.ibm.com/developerworks/library/j-use-elasticsearch-java-apps/index.html
RESTful API接口
常见的增、删、改、查操作实现：
[http://blog.csdn.net/laoyang360/article/details/51931981](http://blog.csdn.net/laoyang360/article/details/51931981)

**ES遇到常见问题**
[https://discuss.elastic.co/](https://discuss.elastic.co/)， [http://elasticsearch.cn/](http://elasticsearch.cn/)

 

**3、elasticsearch-jdbc的安装配置**

需要的安装包：**elasticsearch-jdbc-2.3.2.0-dist.zip**，它是与elasticsearch-2.3.2.tar.gz相对应的，其他版本会出错。

下载地址： [http://download.csdn.net/detail/carboncomputer/9648227](http://download.csdn.net/detail/carboncomputer/9648227)

 

[zsz@zsz downloads]$ vi /etc/profile

\#增加elasticsearch-jdbc插件的环境变量

export JDBC_IMPORTER_HOME=/home/downloads/elasticsearch-jdbc-2.3.2.0

 

[zsz@zsz downloads]$ source /etc/profile

 

** 创建同步：**

[zsz@zsz downloads]$ mkdir /odbc_es

[zsz@zsz downloads]$ cd  /odbc_es

[zsz@zsz odbc_es]$ vi mysql_import_es.sh

> \#!/bin/sh
>
> bin=$JDBC_IMPORTER_HOME/bin
>
> lib=$JDBC_IMPORTER_HOME/lib
>
> echo '{
>
> "type" : "jdbc",
>
> "jdbc": {
>
> "elasticsearch.autodiscover":true,
>
> "elasticsearch.cluster":"elasticsearch",##需要与/elasticsearch-2.3.2/config/elasticsearch.yml的配置对应
>
> "url":"jdbc:mysql://***:3306/**",
>
> "user":"**",
>
> "password":"**",
>
> "sql":"select * from news",
>
> "elasticsearch" : {
>
>   "host" : "127.0.0.1",
>
>   "port" : 9300
>
> },
>
> "index" : "myindex",
>
> "type" : "mytype"
>
> }
>
> }'| java \
>
>   -cp "${lib}/*" \
>
>   -Dlog4j.configurationFile=${bin}/log4j2.xml \
>
>   org.xbib.tools.Runner \
>
>   org.xbib.tools.JDBCImporter
>
>  
>
> **##根据个人项目情况填写以上的地址**
>
> ** **

**运行数据同步脚本mysql_import_es.sh：**

[zsz@zsz odbc_es]$ ./mysql_import_es.sh

**查看是否同步成功：**

[zsz@zsz odbc_es]$ curl 'localhost:9200/_cat/indices?v'
health status index    pri rep docs.count docs.deleted store.size pri.store.size 
**yellow open   myindex    5   1        163            0    146.5kb        146.5kb **
yellow open   customer   5   1          0            0       795b           795b 

[zsz@zsz odbc_es]$ curl -XGET 'http://127.0.0.1:9200/myindex/mytype/_search?pretty'
{
  "took" : 9,
  "timed_out" : false,
  "_shards" : {
​    "total" : 5,
​    "successful" : 5,
​    "failed" : 0
  }

......

说明同步数据成功。

 

**问题与解决：**

1、提示no cluster nodes available, check settings 

解决：请查看/elasticsearch-2.3.2/config/elasticsearch.yml。一般都是该文件配置错误造成的，比如单机模式的，配置了node节点，或者cluster.name错误。