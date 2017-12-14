---
title: monogdb-deploy-shard-cluster
date: 2017-03-01 09:43:40
tags:
     - mongo
---

### monogdb deploy-shard-cluster

参考文档: 

https://docs.mongodb.com/master/tutorial/deploy-shard-cluster/

http://www.tuicool.com/articles/NnEBFb

http://blog.csdn.net/luonanqin/article/details/8497860

### 创建配置服务器


``` bash
bin/mongod -f config/configsvr-0.conf
bin/mongod -f config/configsvr-1.conf
bin/mongod -f config/configsvr-2.conf
```

<!--more-->
### 配置文件

```
sharding:
  clusterRole: configsvr
replication:
  replSetName: configrs
systemLog:
   destination: file
   path: "/opt/monogodb/mongo-shard/log/configsvr-0.log"
   logAppend: true
storage:
   dbPath: "/opt/monogodb/mongo-shard/data/configsvr-0"
   journal:
      enabled: true
processManagement:
   fork: true
   pidFilePath: "/opt/monogodb/mongo-shard/pid/configsvr-0.pid" 
net:
   bindIp: 192.168.2.185
   port: 10000
```

### initiate the replica set

```
bin/mongo 192.168.2.185:10000
use admin

rs.initiate( {
   _id : "configrs",
   configsvr: true,
   members: [ { _id : 0, host : "192.168.2.185:10000" }, { _id : 1, host : "192.168.2.185:10001" },{ _id : 2, host : "192.168.2.185:10002"}]
})

```


### 创建分片服务器

此处仅列出与配置服务器不一致的地方

配置文件 sharding:
		 clusterRole: shardsvr

initiate the replica set:

rs.initiate( {
   _id : "shardrs",
   members: [ { _id : 0, host : "192.168.2.185:20000" }, { _id : 1, host : "192.168.2.185:20001" },{ _id : 2, host : "192.168.2.185:20002"}]
})


### 创建路由服务器

此处仅列出与配置服务器不一致的地方，注意可选参数，多数参数只针对于mongod ,此处配置文件是针对于mongs

### 配置文件

```
sharding:
   configDB: configsvr/192.168.2.185:10000,192.168.2.185:10001,192.168.2.185:10002
systemLog:
   destination: file
   path: "/opt/monogodb/mongo-router/log/router-0.log"
   logAppend: true
processManagement:
   fork: true
   pidFilePath: "/opt/monogodb/mongo-router/pid/router-0.pid"
net:
   bindIp: 192.168.2.185
   port: 27017
```

### 执行
``` bash
//start
bin/mongos -f config/router-0.conf 

//connect
bin/mongo 192.168.2.185:27017
sh.addShard("shardrs/192.168.2.185:20000")
sh.addShard("192.168.2.185:30000")
//enable sharding on the target database
sh.enableSharding("<database>")
//shard a Collection
sh.shardCollection("<database>.<collection>", { <key> : <direction> } )

//The following operation shards the target collection using the hashed sharding strategy.
sh.shardCollection("<database>.<collection>", { <key> : "hashed" } )
```

> 路由服务器没有集群配置，若配置多个，可以通过配置代理，做负载均衡的方式来实现

### 查看分片信息

``` bash
db.printShardingStatus();
//可以在 shards 集合中查到所有的片
db.shards.find();
//databases 集合含有已经在片上的数据库列表和一些相关信息
db.databases.find();
//块信息保存在 chunks 集合中，你可以看到数据到底是怎么切分到集群的。
db.chunks.find()
```

### shard key 选取

- 若使用hashed sharding，必须在shard key上建立hashed索引

  若集合为空，则 sh.shardCollection("<database>.<collection>", { <key> : "hashed" } )

  若集合非空，db.person.createIndex({"idCode":"hashed"})建立hashed索引 (??此处分片并未试验成功，待后续尝试)

> 注意不允许给唯一索引建立建立hashed索引，可以给hashed索引建立普通索引
