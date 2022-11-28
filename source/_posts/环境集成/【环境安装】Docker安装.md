---
title: 【环境安装】Docker安装
tags:
  - Docker
categories:
  - 环境集成
---
# 【环境安装】Docker安装

## CentoOS-7

### 安装步骤:

**1、卸载已经安装的Docker**

```shell
sudo yum remove docker \
          docker-client \
          docker-client-latest \
          docker-common \
          docker-latest \
          docker-latest-logrotate \
          docker-logrotate \
          docker-selinux \
          docker-engine-selinux \
          docker-engine
```

**2、配置阿里云yum源仓库**

```shell
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
# 如果找不到yum-config-manager就通过命令yum -y install yum-utils 安装就可以使用yum-config-manager命令
```

**3、查看可以安装的docker版本**

```shell
yum list docker-ce --showduplicates | sort -r #查看可以安装的版本并倒序排序
```

**4、安装最新版本Docker**

```shell
sudo yum install -y docker-ce #注意：安装Docker最新版本，无需加版本号；或者选择你需要的版本安装
```

**5、设Docker阿里云加速器**

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://lkq3q0he.mirror.aliyuncs.com"]
}
EOF
```

**6、启动Docker**

```shell
sudo systemctl daemon-reload #重新加载服务配置文件

sudo systemctl enable docker.service && systemctl restart docker.service   #设置开机启动与重启docker服务
```

**7、查看Docker版本**

```shell
~#docker version
Client: Docker Engine - Community
 Version:           19.03.8
 API version:       1.40
 Go version:        go1.12.17
 Git commit:        afacb8b
 Built:             Wed Mar 11 01:27:04 2020
 OS/Arch:           linux/amd64
 Experimental:      false
...........
~#docker info
Client:
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 19.03.8
 Storage Driver: overlay2
  Backing Filesystem: <unknown>
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
```

## Ubuntu-18

**1. docker安装**

- 卸载旧版本docker
- 全新安装时，无需执行该步骤

```shell
$ sudo apt-get remove --purge docker docker-engine docker.io
```

- 安装依赖包

```shell
$ sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```

- 添加官方密钥

```shell
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

显示OK,表示添加成功

- 使用下面的命令来设置stable存储库

```shell
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

- 更新一下apt包索引

```shell
$ sudo apt-get update
```

- 列出可用的版本

```shell
$ apt-cache madison docker-ce
```

- 选择你需要的版本安装，我选择的是适合rancher的

```shell
$ sudo apt-get install docker-ce=18.06.3~ce~3-0~ubuntu
```

## 为Docker替换阿里镜像源

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://t44588bn.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

