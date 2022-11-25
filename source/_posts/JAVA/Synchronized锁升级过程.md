---
title: Synchronized锁升级过程
tags:
  - 锁
  - Synchronized
categories:
  - JAVA
---
# Synchronized锁升级过程

![](https://i.loli.net/2021/02/07/kvzPKZTwC4oYUgl.png)

## 无锁

表示刚刚new出来的对象

## 偏向锁

此时有一个线程过来争夺锁，发现没有人用过这把锁，此时会在对象头里标记该线程ID，表示该对象锁偏爱于该线程

## 轻量级锁（自旋锁、自适应锁、CAS）

这时又有线程二来争夺锁资源，那么该对象锁升级为轻量级锁，也就是自旋锁，A线程在使用，B线程循环等待。

JDK1.6中-XX:+UseSpinning开启；
-XX:PreBlockSpin=10 为自旋次数；
JDK1.7后，去掉此参数，由jvm自动控制；

## 重量级锁

向硬件级别CPU去申请锁资源
