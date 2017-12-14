---
title: zookeeper
date: 2017-03-29 11:08:26
tags:
    - zookeeper
---

### deploy standalone

conf目录下提供了配置的样例zoo_sample.cfg，要将zk运行起来，需要将其名称修改为zoo.cfg。 

bin/zkServer.sh start zoo-3.cfg (默认去config目录下查找zoo-3.cfg配置)

./zkServer.sh start-foreground

bin/zkCli.sh -server 192.168.0.1:2181

bin/zkCli.sh -server 192.168.229.160:2181,192.168.229.161:2181,192.168.229.162:2181

<!--more-->

### deploy leader-follower

集群模式 mode: leader-follower

server.1=192.168.2.185:3333:4444
server.2=192.168.2.185:3334:4445
server.3=192.168.2.185:3335:4446

server.A=B：C：D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，
需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。
 
同时需要在${dataDir}目录下配置myid文件指定标识该节点，缺失则无法启动

异常信息如下：

``` java
org.apache.zookeeper.server.quorum.QuorumPeerConfig$ConfigException: Error processing /home/crxj-coll/zookeeper-3.4.5/bin/../conf/zoo.cfg
	at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parse(QuorumPeerConfig.java:121)
	at org.apache.zookeeper.server.quorum.QuorumPeerMain.initializeAndRun(QuorumPeerMain.java:101)
	at org.apache.zookeeper.server.quorum.QuorumPeerMain.main(QuorumPeerMain.java:78)
Caused by: java.lang.IllegalArgumentException: /var/tmp/zookeeper/myid file is missing
	at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parseProperties(QuorumPeerConfig.java:344)
	at org.apache.zookeeper.server.quorum.QuorumPeerConfig.parse(QuorumPeerConfig.java:117)
	... 2 more
Invalid config, exiting abnormally
```
