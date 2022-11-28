---
title: Jenkins 集成 SonarQube Scanner
tags:
  - Jenkins
  - SonarQube
categories:
  - 环境集成
---
# Jenkins 集成 SonarQube Scanner

## 1. 安装Jenkins

下载安装包，这里我们下载war包：https://jenkins.io/download/

运行**jenkins.war**的方式有两种：

第一种：将其放到tomcat中运行（放到webapps目录下，启动tomcat）

第二种：直接执行 **java -jar jenkins.war --httpPort=8080**

第三种：Docker方式部署：

```shell
#查看jenkins版本命令
docker search jenkins
#拉取镜像命令(不标注表示最新的)
docker pull jenkins
#本文的挂载目录是home下
mkdir /home/jenkins
#修改权限（1000:1000 是UID和GID）重点：此目录需要设置权限，否则启动容器报错权限错误
chown -R 1000:1000 /home/jenkins/
#运用镜像启动容器命令
docker run -d -p 8000:8080 -p 50000:50000 -v /home/jenkins:/var/jenkins_home --name jenkins --privileged=true  -u root jenkins
```

https://jenkins.io/doc/pipeline/tour/getting-started/

这里我们选择第一种方式

启动tomcat（bin/startup.sh）访问 http://localhost:8080/jenkins/

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922191715.png)

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922191838.png)

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922191858.png)

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922192442.png)

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922192501.png)

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922192516.png)

Jenkins安装完成。

## 2.安装SonarQube Scanner插件

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922192659.png)

安装完成后、配置SonarQube

在SonarQube中生成一个Token（PS：用token代替输入用户名和密码）。在用户头像下“我的账户”、“安全”下，生成token。

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922192900.png)

接着在Jenkins中配置连接sonarqube服务器的地址，这里用到的token就是刚才在sonarqube中创建的那个token

在Jenkins中的系统配置中设置SonarQube servers的token

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922193011.png)

在Jenkins中的全局配置中安装SonarQube servers

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922193118.png)

## 3.创建任务

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922193619.png)

配置SVN或者GIT的配置

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922193754.png)

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922193848.png)

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200922194021.png)

最最重要的是，配置SonarQube **analysis properties**

可以将其单独写到一个配置文件（sonar-project.properties）里面，也可以像这样每次都写一遍

```properties
sonar.projectKey=${JOB_NAME}
sonar.sources=.
sonar.projectName=${JOB_NAME}
sonar.projectVersion=${BUILD_VERSION}

sonar.language=java
sonar.sourceEncoding=UTF-8

sonar.sources=$WORKSPACE
sonar.java.binaries=$WORKSPACE
```

其中，sonar.java.binaries属性至关重要，笔者也是试了好多次

相关文档在这里：

https://github.com/SonarSource/sonar-scanning-examples/blob/master/sonarqube-scanner/sonar-project.properties

https://docs.sonarqube.org/display/PLUG/Java+Plugin+and+Bytecode
