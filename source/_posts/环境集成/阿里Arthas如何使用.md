---
title: 阿里Arthas如何使用
tags:
  - Arthas
categories:
  - 环境集成
---
# 阿里Arthas如何使用

[arthas官方文档](https://arthas.gitee.io/index.html)

本文介绍下arthas如何监控服务集群

## [Web Console](https://arthas.gitee.io/web-console.html)

> 先从arthas官方github上releases下载[arthas-tunnel-server](https://github.com/alibaba/arthas/releases)
>
> Arthas tunnel server是一个spring boot fat jar应用，直接`java -jar`启动：
>
> `java -jar  arthas-tunnel-server.jar`
>
> 默认情况下，arthas tunnel server的web端口是`8080`，arthas agent连接的端口是`7777`。
>
> 启动之后，可以访问 http://127.0.0.1:8080/ ，再通过`agentId`连接到已注册的arthas agent上。
>
> 通过Spring Boot的Endpoint，可以查看到具体的连接信息： http://127.0.0.1:8080/actuator/arthas ，登陆用户名是`arthas`，密码在arthas tunnel server的日志里可以找到，比如：
>
> ![](https://i.loli.net/2021/05/24/UtBFHiCeG4K21Nj.png)

**[Web页面](http://127.0.0.1:8080/ )：**

![](https://i.loli.net/2021/05/24/BeE5tdvXsuZSlcY.png)

**[Spring Boot的Endpoint查看到具体的连接信息]( http://127.0.0.1:8080/actuator/arthas )**

> 登陆用户名是`arthas`，密码在arthas tunnel server的日志里可以找到

![](https://i.loli.net/2021/05/24/jAkmteLpdfwV7Tz.png)

## Arthas Spring Boot Starter

> 注意：目前只支持SpringBoot2

**添加pom依赖**

```xml
<dependency>
  <groupId>com.taobao.arthas</groupId>
  <artifactId>arthas-spring-boot-starter</artifactId>
  <version>${arthas.version}</version>
</dependency>
```

最新版本：[查看](https://search.maven.org/search?q=arthas-spring-boot-starter)

**配置属性**

```properties
# arthas.agent-id=user	# 指定服务名称，必须唯一。不指定的话，默认拿spring.application.name + 唯一标识Id，生成一个
arthas.tunnel-server=ws://127.0.0.1:7777/ws	# tunnel-server 服务地址
# 如果是防止一个机器上启动多个 arthas端口冲突。可以配置为随机端口，或者配置为 -1，并且通过tunnel server来使用arthas。
arthas.telnetPort=-1 # telnet端口，默认3658；-1：不listen telnet端口；0：则随机telnet端口
arthas.httpPort=-1 # http端口，默认8563；-1：不listen http端口；0：则随机http端口
```

之后直接启动SpringBoot服务后查看

![](https://i.loli.net/2021/05/24/rHPVs4DCxfvnc5t.png)

查看agents注册上来的服务id，之后在填入web端的AgentId，再点击Connect连接。

![](https://i.loli.net/2021/05/24/fMmlnjwycDGIHZ9.png)

之后，我们就可以通过arthas的相关命令去操作该服务

