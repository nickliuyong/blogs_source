---
title: 关于mina.md
date: 2016-12-07 17:02:50
tags: 
      - rpc
      - nio
---

### 关于mina

```
官方文档：http://mina.apache.org/mina-project/documentation.html

中文文档： https://waylau.gitbooks.io/apache-mina-2-user-guide/ (vpn)

          http://blog.csdn.net/defonds/article/category/1844073

参考博客：http://blog.csdn.net/huwenfeng_2011/article/details/43413009

```

### 工作原理

<!-- more -->

![工作原理](/blogs/images/mina-3.png)

### IoBuffer

```
- 零拷贝策略：一旦我们从套接字中读取了数据，我们想在稍后的过程中避免再次拷贝（Mina未实现） 

- heap buffer 和 direct buffer 区别:

      heap buffer这种缓冲区是分配在堆上面的，直接由Java虚拟机负责垃圾回收，可以直接想象成一个字节数组的包装类。
      direct buffer则是通过JNI在Java的虚拟机外的内存中分配了一块缓冲区(所以即使在运行时通过-Xmx指定了Java虚拟
      机的最大堆内存，还是可以实例化超出该大小的Direct ByteBuffer),该块并不直接由Java虚拟机负责垃圾回收收集，
      但是在direct buffer包装类被回收时，会通过Java Reference机制来释放该内存块。(但Direct Buffer的JAVA对象
      是归GC管理的，只要GC回收了它的JAVA对象，操作系统才会释放Direct Buffer所申请的空间)

      两者各有优劣势:direct buffer对比 heap buffer:
      
      劣势：创建和释放Direct Buffer的代价比Heap Buffer得要高；
      优势：当我们把一个Direct Buffer写入Channel的时候，就好比是“内核缓冲区”的内容直接写入了Channel，这样显然快了，
            减少了数据拷贝（因为我们平时的read/write都是需要在I/O设备与应用程序空间之间的“内核缓冲区”中转一下的）。
            而当我们把一个Heap Buffer写入Channel的时候，实际上底层实现会先构建一个临时的Direct Buffer，然后把
            Heap Buffer的内容复制到这个临时的Direct Buffer上，再把这个Direct Buffer写出去。当然，如果我们多次
            调用write方法，把一个Heap Buffer写入Channel，底层实现可以重复使用临时的Direct Buffer，这样不至于因为
            频繁地创建和销毁Direct Buffer影响性能。

      结论：Direct Buffer创建和销毁的代价很高，所以要用在尽可能重用的地方。 比如周期长传输文件大采用direct buffer，
            不然一般情况下就直接用heap buffer 就好。
```

### 编解码过滤器

```



```

### Executor 过滤器

![线程模型-1](/blogs/images/mina-1.png)
![线程模型-2](/blogs/images/mina-2.png)

