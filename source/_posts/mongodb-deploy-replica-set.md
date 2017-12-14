---
title: mongodb-deploy-replica-set
date: 2017-03-01 09:43:15
tags:
     - mongo
---


### monogdb deploy-replica-set

参考文档：

https://docs.mongodb.com/manual/tutorial/deploy-replica-set/

http://www.tuicool.com/articles/YruU7z


### 安装

```bash
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.4.2.tgz
tar -xzvf mongodb-linux-x86_64-rhel70-3.4.2.tgz
mkdir mongo-replic
mv mongodb-linux-x86_64-rhel70-3.4.2.tgz  mongo-replic/mongodb
mkdir -p data/master
mkdir -p data/slaver  
mkdir -p data/arbiter
```
<!--more-->

### 配置文件

```
#master.conf
#数据存放目录
dbpath=/opt/monogodb/mongo-replic/mongodb/data/master
#日志存放路径
logpath=/opt/monogodb/mongo-replic/mongodb/log/master.log
#进程文件，方便停止mongodb
pidfilepath=/opt/monogodb/mongo-replic/mongodb/pid/master.pid
#为每一个数据库按照数据库名建立文件夹存放
directoryperdb=true
#以追加的方式记录日志
logappend=true
#replica set的名字
replSet=testrs
#mongodb所绑定的ip地址
bind_ip=192.168.2.185
#mongodb进程所使用的端口号，默认为27017
port=27017
#mongodb操作日志文件的最大大小。单位为Mb，默认为硬盘剩余空间的5%
oplogSize=1024
#以后台方式运行进程
fork=true
#不预先分配存储
#noprealloc=true

```


### 启动

``` bash
bin/mongod -f config/master.conf
bin/mongod -f config/slaver.conf
bin/mongod -f config/arbiter.conf
```


### initiate the replica set

```
bin/mongo 192.168.2.185:27017

use admin

rs.initiate( {
   _id : "testrs",
   members: [ { _id : 0, host : "192.168.2.185:27017" }, { _id : 1, host : "192.168.2.185:27018" },{ _id : 2, host : "192.168.2.185:27019",arbiterOnly:true }]
})
```


### 注意事项

1、默认情况下SECONDARY不能读写，如果想进行读取操作，可以执行rs.slaveOk()，就可以从SECONDARY读取了。



### 异常

1、about to fork child process, waiting until server is ready for connections.
   ERROR: child process failed, exited with error number 1

> 解决：把配置中相应用到的目录(data、log、pid)都手动创建好后，重新启动就正常了。s