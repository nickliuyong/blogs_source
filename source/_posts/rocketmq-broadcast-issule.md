---
title: rocketmq-broadcast-issule
date: 2017-02-24 16:52:33
tags:
    - rocketmq
---

### 场景描述

rocketmq-broker:v 3.2.6

rocketmq-client:v3.2.6

consumer model:push consumer

message model:broadcast model

问题 ： 在以上场景下，进行消息广播消费，先后建立了俩个组keywordsGroup（前人建立）、keywordsGroup1（我建立）

结果 ： 使用keywordsGroup组进行广播消费，发现拉取不到消息；使用keywordsGroup1一切正常；（怀疑肯定是人品问题？？）

<!--more-->

### 发现问题

读过相关文档或者源码的同学肯定知道，rocketmq 在广播消费时，会吧offset存储在本地的json文件中：其存储路径可以从源代码中找到

``` java
public class LocalFileOffsetStore implements OffsetStore {
    public final static String LocalOffsetStoreDir = System.getProperty(
            "rocketmq.client.localOffsetStoreDir",
            System.getProperty("user.home") + File.separator + ".rocketmq_offsets");
    private final static Logger log = ClientLogger.getLog();
    //以下代码省略
}
```
通过上面可以拼接出完整路径，eg : C:\Users\NickLiu\.rocketmq_offsets\10.3.30.203@gogo\keywordsGroup\offsets.json  (mac的同学请忽略，毕竟屌丝windows)

查看文件内容发现：

``` json
{
	"offsetTable":{}
}
```

于是，开始怀疑，难道消息没有发送到broker(这个通过mqadmin 后台很快的排除这种情况)


附上 mqadmin 一些命令

``` 
   updateTopic          Update or create topic
   deleteTopic          Delete topic from broker and NameServer.
   updateSubGroup       Update or create subscription group
   deleteSubGroup       Delete subscription group from broker.
   updateBrokerConfig   Update broker's config
   topicRoute           Examine topic route info
   topicStatus          Examine topic Status info
   brokerStatus         Fetch broker runtime status data
   queryMsgById         Query Message by Id
   queryMsgByKey        Query Message by Key
   queryMsgByOffset     Query Message by offset
   printMsg             Print Message Detail
   producerConnection   Query producer's socket connection and client version
   consumerConnection   Query consumer's socket connection, client version and subscription
   consumerProgress     Query consumers's progress, speed
   consumerStatus       Query consumer's internal data structure
   cloneGroupOffset     clone offset from other group.
   clusterList          List all of clusters
   topicList            Fetch all topic list from name server
   updateKvConfig       Create or update KV config.
   deleteKvConfig       Delete KV config.
   wipeWritePerm        Wipe write perm of broker in all name server
   resetOffsetByTime    Reset consumer offset by timestamp(without client restart).
   updateOrderConf      Create or update or delete order conf
   cleanExpiredCQ       Clean expired ConsumeQueue on broker.
   startMonitoring      Start Monitoring
   checkMsg             Check Message Store
   statsAll             Topic and Consumer tps stats
   syncDocs             Synchronize wiki and issue to github.com
```

看完了上面命令的人我就不说啥了，偶尔敲还能记得，像我为了解决这个恶心的bug，敲的就整个让人不好了。

说这么多，主要是为了给运维大大们推荐一下：图形话界面工具 rocketmq-console ，先上图感受一下：

![rocketmq-console](/blogs/images/rocketmq-broadcast-issule-1.png)

### rocketmq-console 部署

觉得敲命令比较爽的童鞋，此章忽略

[rocketmq-console下载](https://github.com/zhangyl/rocketmq-console)

修改配置文件(src/main/resources/config.properties)
rocketmq.namesrv.addr=192.168.1.107:9999

其它的按照普通war工程编译部署就好了~

### 源码读取

以上讲了这么多，并没有解决问题，于是只剩下最后一招，闭着眼睛读源码~(我内心绝对是拒绝的)

具体过程不在叙述：附上最后读取完之后的时序图一张，以便帮助大家读取rocketmq的消费流程

![rocketmq-consumer-uml](/blogs/images/rocketmq-broadcast-issule-2.png)

最后在整个消费的回调过程中发现了问题

![rocketmq-consumer-uml](/blogs/images/rocketmq-broadcast-issule-3.png)

没错，竟然不是抛异常，就是只是这么一个信息~

### 解决问题

通过以上信息定位出问题出在rocketmq-broker上面，最终在broker的源码上找到如下代码：

![rocketmq-consumer-uml](/blogs/images/rocketmq-broadcast-issule-4.png)

没错就是这个配置出问题了，根据上下文代码可以最终定位到，consumer在消费时，会在broker
上产生相关的一个订阅的文件，其路径相关代码如下：

``` java
package com.alibaba.rocketmq.broker;
import java.io.File;
public class BrokerPathConfigHelper {
    private static String brokerConfigPath = System.getProperty("user.home") + File.separator + "store"
            + File.separator + "config" + File.separator + "broker.properties";
    public static String getBrokerConfigPath() {
        return brokerConfigPath;
    }
    public static void setBrokerConfigPath(String path) {
        brokerConfigPath = path;
    }

   public static String getTopicConfigPath(final String rootDir) {
        return rootDir + File.separator + "config" + File.separator + "topics.json";
    }
    public static String getConsumerOffsetPath(final String rootDir) {
        return rootDir + File.separator + "config" + File.separator + "consumerOffset.json";
    }
    public static String getSubscriptionGroupPath(final String rootDir) {
        return rootDir + File.separator + "config" + File.separator + "subscriptionGroup.json";
    }
}
```
最终在broker配置文件(/opt/rocketmq/data/broker-a/store/config/subscriptionGroup.json)下发现

```json
"keywordsGroup":{
                        "brokerId":0,
                        "consumeBroadcastEnable":false,
                        "consumeEnable":true,
                        "consumeFromMinEnable":false,
                        "groupName":"keywordsGroup",
                        "retryMaxTimes":16,
                        "retryQueueNums":1,
                        "whichBrokerWhenConsumeSlowly":1
                },
"keywordsGroup1":{
                        "brokerId":0,
                        "consumeBroadcastEnable":true,
                        "consumeEnable":true,
                        "consumeFromMinEnable":true,
                        "groupName":"keywordsGroup1",
                        "retryMaxTimes":16,
                        "retryQueueNums":1,
                        "whichBrokerWhenConsumeSlowly":1
                }
```

> 经过对比可以发现，consumeBroadcastEnable被设置成了false


### 定位问题

通过上文已经找到了问题所在，只要把配置文件中consumeBroadcastEnable设置为true,并重启broker即可生效。

那问题来了，通过源码可以发现，此属性默认为true(此处不在粘贴代码)，什么情况下会被改为false??

最终经过定位发现broker中有如下一段代码
![rocketmq-consumer-uml](/blogs/images/rocketmq-broadcast-issule-5.png)


> 通过以上可以发现，一旦管理员通过后台(mqadmin)执行了updateSubGroup命令， consumeBroadcastEnable属性则会被设定成false.


### issule1 消费异常

异常描述：多个consumer消费，其中和rocketmq部署在同一台的consumer消费正常，部署在其它服务器上的consumer无法消费，异常信息如下

``` java
Caused by: com.alibaba.rocketmq.remoting.exception.RemotingConnectException: connect to <127.0.0.1:10911> failed
        at com.alibaba.rocketmq.remoting.netty.NettyRemotingClient.invokeSync(NettyRemotingClient.java:641)
        at com.alibaba.rocketmq.client.impl.MQClientAPIImpl.sendMessageSync(MQClientAPIImpl.java:306)
        at com.alibaba.rocketmq.client.impl.MQClientAPIImpl.sendMessage(MQClientAPIImpl.java:289)
        at com.alibaba.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendKernelImpl(DefaultMQProducerImpl.java:679)
        at com.alibaba.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendDefaultImpl(DefaultMQProducerImpl.java:500)

```
> 可能原因是broker多网卡识别错误，可以通过修改broker配置文件brokerIP1来手动配置对外广播ip;亦或是brokerIP1配置错误(不能配置成127.0.0.1)








