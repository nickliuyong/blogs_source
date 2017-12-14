---
title: java线程中sleep、wait、join的理解
date: 2016-10-18 15:36:36
tags: 
      - 多线程
      - j2se
---

### sleep

``` java
public class Thread1 implements Runnable {
    private Test test;
    public Thread1(Test test) {
        this.test = test;
    }
    public void run() {
        synchronized (test){
            try{
                System.out.println("Thread1....");
                System.out.println("等待中");
                //不释放线程资源
                Thread.sleep(3000);
                System.out.println("资源释放");
            }catch (Exception e){
            }
        }
    }
}
```
<!-- more -->

``` java
public class Thread2 implements Runnable {
    private Test test;
    public Thread2(Test test) {
        this.test = test;
    }
    public void run() {
        synchronized (test){
            try{
                System.out.println("Thread2....");
                //不释放线程资源
                Thread.sleep(3000);
            }catch (Exception e){

            }
        }
    }
}
```
### 输出结果:    
>   Thread1....  
    等待中  
    资源释放  
    Thread2....

### 结论：使用sleep进行线程等待时，占用系统资源

### wait

``` java
public class Thread1 implements Runnable {
    private Test test;
    public Thread1(Test test) {
        this.test = test;
    }
    public void run() {
        synchronized (test){
            try{
                System.out.println("Thread1....");
                //释放线程资源
                System.out.println("进入等待中");
                test.wait();
                System.out.println("等待结束");
            }catch (Exception e){
            }
        }
    }
}
```

``` java
public class Thread2 implements Runnable {
    private Test test;
    public Thread2(Test test) {
        this.test = test;
    }
    public void run() {
        synchronized (test){
            try{
                System.out.println("资源释放了");
                System.out.println("Thread2....");
                //释放线程资源
                test.notify();
            }catch (Exception e){
            }
        }
    }
}
```
### 输出结果:
>   Thread1....  
    进入等待中  
    资源释放了  
    Thread2....  
    等待结束

### 结论：使用wait进行线程等待时，不占用系统资源

### join
``` java
    //源码如下：
    public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            //只等待millis
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
```
### 结论：
> 线程至多等待millis  
  永久等待（如果millios=0），直到线程被唤醒





