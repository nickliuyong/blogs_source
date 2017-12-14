---
title: redis部署与集群搭建
date: 2016-10-13 15:40:25
tags: 
  	 - redis
---

### install redis

``` bash
官网download：
url: http://redis.io/download
redis.cn 中文网站：
url：http://www.redis.cn/
install redis:
wget http://download.redis.io/releases/redis-3.2.0.tar.gz 
tar xzf redis-3.2.0.tar.gz
cd redis-3.2.0
make (need gcc compiler)
```
<!-- more -->

### start redis
``` bash
关闭防火墙：
systemctl disable firewalld.service
systemctl stop firewalld.service
src/redis-server &
```

### start redis with redis.conf
```bash
./redis-server /path/to/redis.conf
redis.conf 详细配置文件
参考文档：http://www.tuicool.com/articles/MvMBf2
http://my.oschina.net/fsmwhx/blog/152186
```

### interact with Redis
``` bash
src/redis-cli
```

### create redis cluster
``` bash
参考文档：
http://www.cnblogs.com/yjmyzz/p/redis-cluster-turotial.html
http://www.redis.io/topics/cluster-tutorial
mkdir redis-cluster
cd redis-cluster
mkdir 7000 7001 7002 7003 7004 7005
将上方编译到src下的文件拷贝到以下目录
cp /root/src /root/redis-cluster/7000
cp /root/src /root/redis-cluster/7001
cp /root/src /root/redis-cluster/7002
cp /root/src /root/redis-cluster/7003
cp /root/src /root/redis-cluster/7004
cp /root/src /root/redis-cluster/7005
cd src
建立集群配置文件 redis-cluster.conf
vi redis-cluster.conf
文件内容如下：
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
以集群节点模式分别启动6个redis
cd 7000/src
./redis-server ../redis-cluster.conf
创建集群，由于依赖redis gem
yum install ruby
由于 rubygems.org 镜像 ，国内无法直接访问，需要指定镜像
参考文档：https://github.com/alibaba/ruby.taobao.org/issues/66
gem sources --add http://mirrors.aliyun.com/rubygems/ --remove https://rubygems.org/
如果链接被重置，出现如下错误：
 Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://ruby.taobao.org/specs.4.8.gz)
 说明镜像地址无效
gem install redis
节点加入集群
./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```
### redis数据导出与导入
``` bash
先安装ruby
sudo gem install redis-dump -V   //安装redis-dump 
redis-dump -u :password@xxx.xxx.xxx.xxx:6379  >test.json //导出数据  
cat test.json | redis-load -u :rediss112fF123@114.55.228.45:6379 //导入数据
注意：
如果redis密码中包含＃符号会导致redis-load命令失效
```
