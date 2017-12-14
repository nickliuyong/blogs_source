---
title: linux-常用命令
date: 2017-03-01 18:04:40
tags:
     - linux 
     - centos7
---

### 防火墙(firewalld)

参考文档：https://www.zhaokeli.com/Article/6321.html

``` bash
//开启80端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
//重启防火墙
systemctl restart firewalld
//启动
systemctl start  firewalld
//查看状态
systemctl status firewalld
firewall-cmd --state
//停止
systemctl disable firewalld
//禁用
systemctl stop firewalld
```
<!--more-->