---
title: 【SpringCloud】Eueka注册中心
tags:
  - SpringCloud
  - Eueka
  - 注册中心
categories:
  - SpringCloud
---
# 【SpringCloud】Eueka注册中心

## 客户端启动时如何注册到服务端

Eureka客户端在启动时，首先会创建一个心跳的定时任务，定时向服务端发送心跳信息，服务端会对客户端心跳做出响应，如果响应状态码为404时，表示服务端没有该客户端的服务信息，那么客户端则会向服务端发送注册请求，注册信息包括服务名、ip、端口、唯一实例ID等信息。





