---
author : "杨承龙"
date : "2017-06-12T20:24:00+08:00"
draft : false
title : "MongoDB"
tags : ["MongoDB"]
comments : true     
share : true        
menu : "main" 
          
---


# mongodb

介绍

MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。

MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。

MongoDB 非常强大，同时也非常容易上手。这里介绍一些 MongoDB 的基本概念。

- 文档是 MongoDB 中的数据单元，非常类似于关系数据库管理系统中的行（但是比行要复杂得多）。
- 集合可以看作没有模式的表。
- MongoDB 的单个实例可以容纳多个独立的数据库，每一个都有自己的集合和权限。
- MongoDB 自带简洁但功能强大的 JavaScript shell，用来管理 MongoDB 实例和操作数据。
- 每一个文档都有一个特殊的键 "_id",它在文档所处的集合中是唯一的。

安装

这里先介绍 mac 下如何安装，然后介绍使用 docker 进行安装。

mac下安装

- 首先，下载好 MongoDB 数据库，然后再打开下载好的文件；
- 在终端中，选择合适的位置，输入： sudo mkdir -p /data/db，创建数据库日志文件夹；
- 在终端输入：sudo chown -R  用户名 /data/db ，给予数据库日志文件夹操作权限；
- 进入 MongoDB 的 "bin"目录，使用命令 ./mongod 启动 mongoDB server，启动成功后最后一行应该是端口号，到这里 MongoDB 已经安装成了；
- 新建终端窗口，并输入 ./mongo ，登陆到数据库，接下来就可以进行操作了。

docker 安装

首先把 docker 镜像 pull 下来：

    sudo docker pull mongo

然后启动 mongo 镜像：

    docker run --name mongo-name -v /home/docker/mongo_data:/data/db -d -p 27047:27017 mongo 

上面的命令中，我们创建一个 mongo 容器，mongo-name 就是其名字， -v 参数挂载本地目录 /home/docker/mongo_data 到容器的 /data/db 目录作为数据卷，-d 参数使其在后台运行，-p 参数设置端口，使外界可以通过27047端口访问容器内27017端口，连接数据库。

mongo 镜像已经启动，我们可以使用下面的命令进入 mongo 进行操作 ：

    sudo docker exec -it mongo-name mongo



接下来再介绍下 mongo 的常用命令。

简单命令

首先是连接 mongo 服务，即启动客户端。如果要在27017端口连接本地的 mongo  服务，使用下面的命令即可：

    mogno

如果要连接远端的 mongo 服务，则稍微麻烦点：

    mongo xx.xx.xx.xx:port

如果想退出 mongo 客户端，输入 exit 即可。

那如何关闭 mongo 服务呢？正常情况下我们可能会使用  exit 或者 Ctrl+C 退出，但这样做是有隐患的，可能会导致数据丢失，或者无法再次启动 mongo 服务，正确的做法是下面这样的：

    > use admin
    > db.shutdownServer()

然后就是各种操作了～～～

db

查看当前使用的数据库

    yangs-Air(mongod-3.4.0) > db
    test

切换数据库

    yangs-Air(mongod-3.4.0) > use admin
    switched to db admin

查看 MongoDB 实例拥有哪些数据库

    yangs-Air(mongod-3.4.0) > show dbs;
    admin  → 0.000GB
    analy  → 0.000GB
    github → 0.000GB
    local  → 0.000GB
    media  → 0.001GB
    test   → 0.000GB
    test2  → 0.000GB
    test3  → 0.000GB
    test4  → 0.000GB
    user   → 0.000GB
    work   → 0.000GB

不需要显式创建数据库，当向数据库的某个 collection 插入文档时，数据库就被创建

    yangs-Air(mongod-3.4.0) test> show dbs
    admin  → 0.000GB
    analy  → 0.000GB
    test   → 0.000GB
    local  → 0.000GB
    yangs-Air(mongod-3.4.0) test> use user
    switched to db user
    yangs-Air(mongod-3.4.0) github> show dbs
    admin → 0.000GB
    analy → 0.000GB
    local → 0.000GB
    test  → 0.000GB
    yangs-Air(mongod-3.4.0) user> db.users.insert({"name":"yang"})
    Inserted 1 record(s) in 87ms
    WriteResult({
      "nInserted": 1
    })
    yangs-Air(mongod-3.4.0) user> show dbs
    admin  → 0.000GB
    analy  → 0.000GB
    local  → 0.000GB
    test   → 0.000GB
    user   → 0.000GB

有一些数据库名是保留的，可以直接访问这些具有特殊语义的数据库，同时自己命名数据库时注意不要使用这些名字。

shell

mongo 是一个简化的 JavaScript shell，shell 内置了帮助文档，使用 help 查看

    yangs-Air(mongod-3.4.0) github> help
    	db.help()                    help on db methods
    	db.mycoll.help()             help on collection methods
    	sh.help()                    sharding helpers
    	rs.help()                    replica set helpers
    	help admin                   administrative help
    	......
    	exit                         quit the mongo shell

通过 db.help() 查看数据库级别的帮助：

    > db.help()
    DB methods:
    	db.auth(username, password)
    	db.createCollection(name, { size : ..., capped : ..., max : ... } )
    	db.createView(name, viewOn, [ { $operator: {...}}, ... ], { viewOptions } )
    	db.createUser(userDocument)
    	......
    	db.dropDatabase()
    	db.eval() - deprecated
    	db.getName()
    	db.getPrevError()
    	db.printSlaveReplicationInfo()
    	db.dropUser(username)
    	db.shutdownServer()
    	db.stats()
    	db.version() current version of the server

使用 db.foo.help() 查看集合级别的帮助:

    > db.foo.help()
    DBCollection help
    	db.foo.find().help() - show DBCursor help
    	db.foo.find(...).count()
    	db.foo.find(...).limit(n)
    	......
    	db.foo.findOne([query], [fields], [options], [readConcern])
    	db.foo.latencyStats() - display operation latency histograms for this collection

在 shell 中输入函数名，不加小括号，就可以看到相应函数的 JavaScript 实现代码

    > db.user.find
    function (query, fields, limit, skip, batchSize, options) {
        var cursor = new DBQuery(this._mongo,
                                 this._db,
                                 this,
                                 this._fullName,
                                 this._massageObject(query),
                                 fields,
                                 limit,
                                 skip,
                                 batchSize,
                                 options || this.getQueryOptions());
    
        var connObj = this.getMongo();
        var readPrefMode = connObj.getReadPrefMode();
        if (readPrefMode != null) {
            cursor.readPref(readPrefMode, connObj.getReadPrefTagSet());
        }
    
        var rc = connObj.getReadConcern();
        if (rc) {
            cursor.readConcern(rc);
        }
    
        return cursor;
    }

shell中的基本操作

insert

insert 即向  collection 添加新的 documents.如果插入时集合不存在,插入操作会创建该集合。

MongoDB 中提供了以下方法来插入文档到一个集合:

- db.collection.insert()
- db.collection.insertOne()
- db.collection.insertMany()

    db.COLLECTION_NAME.insert(document)

实例

db.collection.insert() 向集合插入一个或多个文档.要想插入一个文档,传递一个文档给该方法;要想插入多个文档,传递文档数组给该方法.

    >db.col.insert({"name":"test"})

该操作返回了含有操作状态的  WriteResult 对象.插入文档成功返回如下 WriteResult 对象:

    WriteResult({ "nInserted" : 1 })

nInserted 字段指明了插入文档的总数.如果该操作遇到了错误, WriteResult 对象将包含该错误信息.

下面我们看下插入多个文档时的情况：

    > db.users.insert(
    ...    [
    ...      { name: "bob", age: 42, status: "A", },
    ...      { name: "ahn", age: 22, status: "A", },
    ...      { name: "xi", age: 34, status: "D", }
    ...    ]
    ... )

该方法返回了包含操作状态的 BulkWriteResult 对象.成功插入文档返回如下 BulkWriteResult 对象:

    BulkWriteResult({
    	"writeErrors" : [ ],
    	"writeConcernErrors" : [ ],
    	"nInserted" : 3,
    	"nUpserted" : 0,
    	"nMatched" : 0,
    	"nModified" : 0,
    	"nRemoved" : 0,
    	"upserted" : [ ]
    })

现在我们可以调用 find() 查看一下:

    > db.col.find()
    { "_id" : ObjectId("5870c0b3d599544ddbd1d575"), "name" : "test" }
    { "_id" : ObjectId("5870cf70d599544ddbd1d579"), "name" : "bob", "age" : 42, "status" : "A" }
    { "_id" : ObjectId("5870cf70d599544ddbd1d57a"), "name" : "ahn", "age" : 22, "status" : "A" }
    { "_id" : ObjectId("5870cf70d599544ddbd1d57b"), "name" : "xi", "age" : 34, "status" : "D" }

可以看到已经插入成功，但每个文档都多了一个字段："_id",它是那里来的呢？

MongoDB 中储存的文档必须有一个 "id" 键。这个键可以是任意类型的，默认为 ObjectId 对象。在一个集合中，每个文档都有唯一的 "id" 值，来确保集合里面每个文档都能被唯一标识。如果插入文档的时候没有 "id" 键,系统会自动帮你创建一个。这就是为什会多出来一个 "id" 字段。

接下来看下另外两个函数。

db.collection.insertOne() 向集合插入单个 document。

db.collection.insertMany() 向集合插入多个 documents。

    > db.users.insertMany(
       [
         { name: "bob", age: 42, status: "A", },
         { name: "ahn", age: 22, status: "A", },
         { name: "xi", age: 34, status: "D", }
       ]
    )

例子同前。

query

MongoDB 查询数据的语法格式如下：

    >db.COLLECTION_NAME.find(<query filter>, <projection>)

第一个大括号为 filter，第二个为投影，即你想显示的值。

find() 方法以非结构化的方式来显示所有文档。

如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式如下：

    >db.COLLECTION_NAME.find().pretty()

除了 find() 方法之外，还有一个 findOne() 方法，它只返回一个文档。

条件查询

    >db.users.find( { status: "A" } )
    
    >db.users.find( { status: "A", age: { $lt: 30 } } )

上面四个例子中，第一个查询字段 status 的值为 "A" 的文档，第二个查询 status 的值为"A",且 age 的值小于30的文档。

下表是一一些比较操作的示例：

  操作   	格式格式                  	范例                              	RDBMS类似语句         
  等于   	{<key>:<value>}       	db.col.find({"name":"mon"})     	where name = "mon"
  小于   	{<key>:{$lt:<value>}} 	db.col.find({"count":{$lt:50}}) 	where count < 50  
  小于或等于	{<key>:{$lte:<value>}}	db.col.find({"count":{$lte:50}})	where count <= 50 
  大于   	{<key>:{$gt:<value>}} 	db.col.find({"count":{$gt:50}}) 	where count> 50   
  大于或等于	{<key>:{$gte:<value>}}	db.col.find({"count":{$gte:50}})	where count >= 50 
  不等于  	{<key>:{$ne:<value>}} 	db.col.find({"count":{$ne:50}}) 	where count != 50 

返回指定键

    db.user.find({}, {"name" : 1, "_id" : 0})

第二个大括号，我们可以对 find 结果显示的值进行限定，设为1表示显示，0表示不显示，_id 默认显示。如果要将 _id 屏蔽，需显示设置 _id为0。

多条件

    db.users.find(
      {
        status: "A",
        $or: [ { age: { $lt: 30 } }, { type: 1 } ]
      }
    )

$or表示条件数组中的条件只要有一个符合就进行显示。$or其后的条件数组中，其字段的设置。

查询一个条件的多个值

    db.user.find({"num" : {"$in" : [12, 13, 23]}})

如果要对同一条件的多个值进行过滤，可以使用$in。与之相对的是$nin。

取反

    db.user.find({"num" : {"$not" : [12, 13]})

当要选取的元素比较多时我们可以使用$not，它会使不在数组中的值的文档被选中。

update

db.collection.update(): 第一个参数为过滤条件，第二个为 upsert 选项，第三个为是否更新多个文档。

简单实例

    db.people.update({"_id": 1},{"$set" : {"name": "yang"})

上面的例子中，第一个大括号为 filter，第二个为进行的改动。

下面插入一条数据，然后对它进行改动：

    db.users.insert(
     {
        _id: 1,
        name: "sue",
        age: 19,
        type: 1,
        status: "P",
        favorites: { artist: "Picasso", food: "pizza" },
        finished: [ 17, 3 ],
        badges: [ "blue", "black" ],
        points: [
            { points: 85, bonus: 20 },
            { points: 85, bonus: 10 }
         ]
     }
    )

修改内嵌文档

使用$set修改器进行，可直接通过.进行元素的访问

    db.users.update(
       { "favorites.artist": "Pisanello" },
       {
         $set: { "favorites.food": "pizza"}
       }
    )

 错误的情况：

    db.users.update(
       { "favorites.artist": "Pisanello" },
       {
         { "favorites.food": "pizza"}
       }
    )

不使用$set修改器，其意义为进行文档替换，结果就是原来的其他元素都没了，只剩下{ "favorites.food": "pizza"}.

 $set不仅可以对字段的值进行更改，同时也可以对类型进行修改。

$unset可以将某个不需要的值尽行删除。

数字增加或减少

    db.user.update({"_id" : 1}, {"$inc" : {"article" : 1}})

$inc表示对该字段加上某一个值，该值由文档中的字段的值决定，值为负数时可进行减法。记住，该字段必须为数字。

数组操作

插入

    db.comment.update({"_id" : 1}, {"$push" : {"comments" : "first"})

$push表示在数组已有元素的末尾插入一个值。

添加多个

    db.blog.post.update({"title" : "Post"}, {"$push" : {"tag" : {"$each" : ["go", "linux", "database"]}}})

$push和$each结合使用可以将一个数组中的多个值添加到指定字段.

避免重复

    db.user.update({"_id" : 1}, {"$addToSet" : {"emails" : "23132132132@qq.com"})

$addToSet 可以避免重复插入，当数组中已经有要添加的值时，该语句相当于不执行。

删除

对于数组的删除，有两种不同方法。

pop

    db.comment.update({"_id" : 1}, {"$pop" : {"comments" : 1})

$pop表示从数组末尾删除几个元素，若为负数，表示从头删除。

pull

    db.comment.update({}, {"$pull" : {"comments" : "xxxx"})

与$pop根据元素位置删除元素不同，$pull依据条件删除元素。

位置

    db.user.update({"article" : "post"}, {"$inc" : {"comments.0.votes" : 1}})

对于数组元素，我们也可以根据下标进行访问。

定位

    db.user.update({"article" : "post"}, {"$set" : {"comments.$.author" : "Jim"}})

有些时候，我们不知道元素在数组中的下标，但经过前面 filter 的过滤，可以确定该元素，可以使用$ 占位符，它就替代了前面 filter 所得到的元素。

upsert

如果 db.collection.update()，db.collection.updateOne()， db.collection.updateMany() 或者 db.collection.replaceOne()包含 upsert : true  并且没有文档匹配指定的过滤器，那么此操作会创建一个新文档并插入它。如果有匹配的文档，那么此操作修改或替换匹配的单个或多个文档。

    db.user.update({"rep" : 25}, {"$inc" : {"rep" : 3}}, true)

remove

    db.user.remove({})

简单示例如下：

    db.users.remove( { status : false }, 1)

当然还有 delete 方法也可以删除文档：

    db.collection.deleteOne({ status: "D" })

index

创建索引

下面是一个创建索引的例子：

    db.collection.ensureIndex({"field" : 1})

当然我们也可以创建唯一索引：

    db.collection.ensureIndex({"field" : 1},{"unique" : true})

索引的字段也可以是多个，即复合索引：

    db.collection.ensureIndex({"field_one" : 1},{"field_two" : 1})

 查看某一集合的所有索引：

    db.collections.getIndexes()

删除索引：

    db.collections.dropIndex("field")




