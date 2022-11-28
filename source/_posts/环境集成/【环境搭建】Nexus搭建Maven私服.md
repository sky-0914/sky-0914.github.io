---
title: 【环境搭建】Nexus搭建Maven私服
tags:
  - Nexus
  - Maven
categories:
  - 环境集成
---
# 【环境搭建】Nexus搭建Maven私服

**Nexus官方下载地址：https://help.sonatype.com/repomanager3/download**

### Docker启动

```shell
#查找镜像;一般安装star数最多的版本，目前最新是sonatype/nexus3
docker search nexus
#拉取镜像
docker pull sonatype/nexus3
```

**简单方式启动**

```shell
#指定数据卷，防止每次启动容器，容器里的数据丢失，实现容器和虚拟机数据共享。
docker run -p 8081:8081 --name nexus sonatype/nexus3

#如果有容器和下面要创建的容器同名的话，先删除
docker rm <container_name>

#指定虚拟机与容器共享的文件夹
mkdir /usr/local/docker/nexus/nexus-data

#启动容器
docker run -p 8081:8081 --name nexus -v /usr/local/docker/nexus/nexus-data:/nexus-data snoatype/nexus3

#指定数据卷后启动，可能会报一些权限错误，导致启动不起来。可能会需要修改文件夹权限
chmod 777 /usr/local/docker/nexus/nexus-data
```

### 利用docker-compose启动

```shell
#创建docker-compose.yml
mkdir /usr/local/docker
cd /usr/local/docker
vi docker-compose.yml
```

**docker-compose.yml内容如下：**

```shell
version: "3.7"
services:
  nexus:
    restart: "no" 
    image: sonatype/nexus3
    container_name: nexus
    ports:
      - 8081:8081 
    volumes:
      - /usr/local/docker/nexus/nexus-data:/nexus-data
```

> version : 指定docker-compose语法版本，版本不同，支持的docker也不同

> restart

> services : 多种服务的根节点

> nexus : 服务名随意起，代表要创建的服务

> restart : 容器的重启策略，有no、always、on-failure、
> unless-stopped四种可选值。

> image : 容器依据的镜像

> container_name : 容器名

> ports : 端口

> volumes : 数据卷

启动后访问地址：http://ip:port/

**默认账号：admin**

**管理员登录密码在 `nexus/nexus-data` 中`admin` 开头的文件中，将其拷贝输入即可。**

**登录之后会要求修改密码，按要求修改即可。**

## Nexus 配置

**在Maven的conf下的setting.xml文件中配置**

```xml
<server>
  <id>releases</id>
  <username>admin</username>
  <password>admin123</password>
</server>
<server>
  <id>snapshots</id>
  <username>admin</username>
  <password>admin123</password>
</server>
```

**配置仓库地址**

```xml
<!-- 配置远程仓库、阿里云代理镜像 -->
<mirror>
   <!--该镜像的唯一标识符。id用来区分不同的mirror元素。 -->
   <id>maven-public</id>
   <!--镜像名称 -->
   <name>maven-public</name>
   <!--*指的是访问任何仓库都使用我们的私服-->
   <mirrorOf>*</mirrorOf>
   <!--该镜像的URL。构建系统会优先考虑使用该URL，而非使用默认的服务器URL。 -->
   <url>http://${ip}:${port}/repository/maven-public/</url>
</mirror>

<mirror>
  <id>nexus-aliyun</id>
  <mirrorOf>*</mirrorOf>
  <name>Nexus aliyun</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>

<mirror>
  <id>alimaven</id>
  <mirrorOf>central</mirrorOf>
  <name>aliyun maven</name>
  <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
```

**在项目pom.xml文件里配置私服仓库地址**

```xml
<repositories>
  <repository>
    <id>maven-nexus</id>
    <name>maven-nexus</name>
    <url>http://${ip}:${port}/repository/maven-public/</url>
    <releases>
      <enabled>true</enabled>
    </releases>
    <snapshots>
      <enabled>true</enabled>
    </snapshots>
  </repository>
</repositories>
```

**发布私有jar包**

```xml
<!-- 私服仓库地址、该id对应的是setting.xml文件里的<server>里id配置。名称必须一样 -->
<distributionManagement>
  <repository>
    <id>releases</id>
    <name>Releases</name>
    <url>http://${ip}:${port}/repository/maven-releases/</url>
  </repository>
  <snapshotRepository>
    <id>snapshots</id>
    <name>Snapshot</name>
    <url>http://${ip}:${port}/repository/maven-snapshots/</url>
  </snapshotRepository>
</distributionManagement>
<!-- 发布私有包时的maven打包插件 -->
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF-8</encoding>
      </configuration>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-source-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

#### 私服工作原理：

配置Nexus之前，需要先了解一下私服的工作原理，如下图：

![](https://i.loli.net/2020/06/06/tk479CNpxclM8D6.png)

找依赖包的流程： 首先在本地仓库中找，如果没命中，那么就找远程私服；远程私服的查找规则同样是先找host属性的私有库，然后再去找proxy属性的远程仓库；可以配置多个proxy；

#### Nexus仓库类型介绍

默认安装有以下这几个仓库，在控制台也可以修改远程仓库的地址，第三方仓库等。

- hosted（宿主仓库库） ：存放本公司开发的jar包（正式版本、测试版本）
- proxy（代理仓库）：代理中央仓库、Apache下测试版本的jar包
- group（组仓库）：使用时连接组仓库，包含Hosted（宿主仓库）和Proxy（代理仓库）

