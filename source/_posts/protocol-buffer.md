---
title: protocol-buffer
date: 2017-01-06 10:53:20
tags:
    - protocol buffer 
    - 序列化
    - 网络协议通信
---

### 关于protocol buffer

[官方文档](https://developers.google.com/protocol-buffers/) (需要VPN)

[github地址](https://github.com/google/protobuf/releases) 

### protocol2 与 protocol3 

[参考博客](https://solicomo.com/network-dev/protobuf-proto3-vs-proto2.html)

个人觉得总结的很到位~

<!-- more -->

### protoc 安装

[下载地址](https://github.com/google/protobuf/releases)

在页面最底下

![下载](/blogs/images/protocol-buffer-1.png)

也可以自行拉源码进行编译~


### .proto文件编译

protoc --proto_path=[.proto文件扫描路径]   --java_out=[生成源码放置路径]  [文件名**.proto]

### protocol-java-codec 下载

```
可以自行拉取对应语言的源码进行编译

也可以通过maven下载

<!--protocol buffer-->
    <dependency>
      <groupId>com.google.protobuf</groupId>
      <artifactId>protobuf-java</artifactId>
      <version>3.1.0</version>
    </dependency>
```
### protocol 序列化与反序列化

```
此demo使用了protocol3新特性，即支持json mapping，需依赖相应jar

<dependency>
      <groupId>com.google.protobuf</groupId>
      <artifactId>protobuf-java-util</artifactId>
      <version>3.1.0</version>
</dependency>
```

```java
package demo;
import codec.ReqWrapper;
import com.google.protobuf.util.JsonFormat;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectOutputStream;
/**
 * @author NickLiu on 2017/1/5 0005 16:17.
 */
public class TestProtocol {
    public static void main(String[] args) throws Exception {
//        ReqWrapper.ReqEntity reqEntity = ReqWrapper.ReqEntity.newBuilder().setId(1).setName("NickLiu").build();
        String json = "{\"id\":1,\"name\":\"NickLiu\"}";
        JsonFormat.Parser parser = JsonFormat.parser();
        ReqWrapper.ReqEntity.Builder builder = ReqWrapper.ReqEntity.newBuilder();
        parser.merge(json,builder);
        //将对象序列化
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ReqWrapper.ReqEntity reqEntity = builder.build();
        reqEntity.writeTo(bos);
        byte[] bytes = bos.toByteArray();
        System.out.println("length="+bytes.length);
        //反序列化
        ByteArrayInputStream bis = new ByteArrayInputStream(bytes);
        ReqWrapper.ReqEntity deReqEntity = ReqWrapper.ReqEntity.parseFrom(bis);
        System.out.println("id="+deReqEntity.getId()+",name="+deReqEntity.getName());

        JsonFormat.Printer printer = JsonFormat.printer();
        System.out.println("msg="+printer.print(deReqEntity));
        //jdk序列化
        ByteArrayOutputStream bos_jdk = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos_jdk);
        oos.writeObject(deReqEntity);
        oos.flush();
        oos.close();
        System.out.println("length="+bos_jdk.toByteArray().length);
    }
}
```
