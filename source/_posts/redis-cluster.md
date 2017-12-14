---
title: redis-cluster
date: 2017-03-29 09:14:03
tags:
     - redis
---


### codis

http://www.cnblogs.com/xuanzhi201111/p/4425194.html


http://www.oschina.net/p/codis/


BSD协议


原子 – Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。


同时，在磁盘格式方面他们是紧凑的以追加的方式产生的，因为他们并不需要进行随机访问。


Slaves能过接口其他slave的链接，除了可以接受同一个master下面slaves的链接以外，还可以接受同一个结构图中的其他slaves的链接。？？？？？？


一致性哈希算法：  http://blog.csdn.net/sparkliang/article/details/5279393


Gossip协议

<!--more-->
redis cluster原理  https://sanwen8.cn/p/232z3Zk.html


sentinel deploy redis cluster

http://blog.csdn.net/donggang1992/article/details/50981341

http://blog.csdn.net/yingxiake/article/details/51671335


master互相之间存储的信息??

如何寻找对应的节点??


redis.call() 和 redis.pcall() 两个函数的参数可以是任意的 Redis 命令：


http://doc.redisfans.com/

http://redisdoc.com/  中文

redission api doc http://www.javadoc.io/doc/org.redisson/redisson/2.8.2


Raft协议一致性

http://www.cnblogs.com/mindwind/p/5231986.html


### Replication servers mode

参考文档：
	https://redis.io/topics/replication
	http://redisdoc.com/topic/replication.html#id1


从服务器同步数据不会阻塞主服务器，同时不会阻塞从服务器(当设置允许使用过期数据时)，否则从服务器在load RDB时会使从服务器处于不能处理命令请求的状态。

#### slave 配置
	
     slaveof 192.168.1.1 6379

缺点：不能进行主从之间切换

优点：可以单独的为主服务器提供数据冗余功能；也可以通过提供只读命令来提高扩展性，用来处理慢查询。
      master配置为不写数据时，不要设置为随着系统重启而重启，否则slave会因为同步master而会把自身磁盘给清空 （一般不推荐开启master turn off writer data）
      master支持远程无磁盘初始化


redis 2.8+ 支持 PSYNC (部分同步) 与 SYNC(完全同步)

从服务器可以配置成可写模式，但会被完全同步策略覆盖数据~~

可以通过增加以下配置来降低Redis主服务器的复制操作

至少从服务器所请求数量： min-slaves-to-write 
网络延迟允许的最大值： min-slaves-max-lag

### Sentinel servers mode

参考文档：
	https://redis.io/topics/sentinel
	http://redisdoc.com/topic/sentinel.html


start command
```bash
    redis-sentinel /path/to/sentinel.conf
or  redis-server /path/to/sentinel.conf --sentinel
```
sentinel monitor <master-group-name> <ip> <port> <quorum>

quorum : 1、master不可到达需要确认的次数，一般不超过sentinel process,到达quorum，其中一个sentinel process 发起failover 
         2、选择有哪个sentinel来真正发起failover,  此时quorum的含义为 发起的leader选举需要获得的投票数
         3、大小配置,一般配置sentinel的N/2+1  N：机器台数

sentinel 可以通过连接master发现其它sentinel信息,通过订阅__sentinel__:hello 频道自动发现。

主观下线：  master-down-after-milliseconds 限制时间内无返回，认为master down.

客户端可以通过SUBSCRIBE订阅Sentinel

疑问： sentinel 客观下线的完整机制？？   sentinel选举算法？？


### cluster servers mode
https://redis.io/topics/cluster-tutorial

redission client wiki: https://github.com/redisson/redisson#quick-start


每一个redis server 占用俩个端口 ，第一个用于client通信 server
另外一个端口+10000（固定值）用于server之间总线通信


2^14 = 16384 
CRC16(key)%16384 进行映射


master-slave：异步写入

性能 和 数据一致性权衡

WATI命令 （等待slave写入完毕返回） 降低数据丢失的风险


deploy：

```bash
tar -xvf redis-3.2.8.tar.gz
mkdir redis-cluster
cd redis-cluster
mkdir 7000 7001 7002 7003 7004 7005
# ruby 依赖安装命令
yum install gem  
gem install redis


# 编译redis
yum install gcc
make
../redis-server ./redis.conf
./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```

master与slave都挂掉了，所属的slots将不可达
当大部分master不可用时，集群不起作用，不再进行主从切换


> 对比： sentinel模式可用性高，不支持分片，选取客户端时，主义需要支持sentinel模式;
         cluster模式支持分片;
	 replication模式不支持主从切换;

### transaction:

redis 不支持回滚

before  v2.6.5  : 入队失败，该命令放弃，其它命令正常执行

after   v2.6.5  : 入队失败，改事物被标记为放弃执行，在使用pipeline时，会比较好

script:

eval " if redis.call('EXISTS', KEYS[1]) == 1 then return  redis.call('ZADD', KEYS[1],ARGV[1],ARGV[2])  end " 1 COMMUNITY:FEEDS:274 1490340908769 134833420623678464

### 分区(partitioning)

1、client实现，直连redis instance

2、协助分区的代理，例如Twemproxy(http://antirez.com/news/44) ;client连接代理，有代理来连接redis instance 

3、查询路由： 由一个instance转发到另一个instance ;  集群模式下，client连接其中一个实例可以获取到各slots分布，然后在客户端经过hash算法，直接决定key的存储实例。