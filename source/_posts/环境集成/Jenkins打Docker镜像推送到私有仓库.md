---
title: Jenkins打Docker镜像推送到私有仓库
tags:
  - Jenkins
  - Docker
categories:
  - 环境集成
---
# Jenkins打Docker镜像推送到私有仓库

> 因为我的Jenkins是安装在群晖NAS中的docker，所以我这边就以Docker安装Jenkins为例

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200923212025.png)

```shell
echo '================Docker开始=============='

echo '================登陆阿里云私有镜像仓库=============='
# 定义镜像名称
IMAGE_NAME="scm"
# 查看当前目录
pwd
# 进入web目录Dockerfil所在目录下,scm-web是我项目目录，Dockerfile文件在该目录下
cd scm-web
# 查看当前目录
pwd
echo "# 登陆阿里云docker私有仓库"
docker login --username=${username} --password=${password} registry.cn-hangzhou.aliyuncs.com
echo "# 打镜像tag"
docker -t registry.cn-hangzhou.aliyuncs.com/hopefun/w2002:$IMAGE_NAME-${BUILD_NUMBER} $WORKSPACE/scm-web
echo "# 推送到仓库"
docker push registry.cn-hangzhou.aliyuncs.com/hopefun/w2002:$IMAGE_NAME-${BUILD_NUMBER}
echo "# 删除本地镜像"
docker rmi registry.cn-hangzhou.aliyuncs.com/hopefun/w2002:$IMAGE_NAME-${BUILD_NUMBER}
echo "# 退出私有仓库"
docker logout 

echo '================Docker结束=============='

echo "部署镜像 ===============>>>>>>>>>>>>>>>>>>>>>>> registry.cn-hangzhou.aliyuncs.com/hopefun/w2002:$IMAGE_NAME-${BUILD_NUMBER}"
```

如果上来就直接jenkins sh构建运行shell脚本会报：` docker: command not found`

如果普通docker，把宿主机的这两个文件挂载到jenkins容器中，就可以执行docker命令了。

```shell
/var/run/docker.sock
/bin/docker
```

这个是我在网上看到的教程，我还没试过。。。

有空会试试的，看看行不行。

---



可以是我群晖NAS的docker，我是先在NAS的shell命令下找到docker文件，之后将该文件复制到homes目录下。

```shell
# 群晖NAS的docker文件在/usr/local/bin/下
cp /usr/local/bin/docker /volume1/homes/admin/jenkins_docker/bin
```

之后就在群晖NAS的web页面去操作docker容器，挂载目录。

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200923213259.png)

再然后jenkins sh构建运行shell脚本会报：`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`

这个是因为docker找不到守护进程，那么我们开启宿主机的2375端口。通过这种方式：`docker -H tcp://172.17.0.1:2375 version`

参考文档：https://blog.csdn.net/liumiaocn/article/details/93749435

然后我们在脚本中添加：

```shell
echo '================Docker开始=============='

echo '================登陆阿里云私有镜像仓库=============='
# 定义镜像名称
IMAGE_NAME="scm"
# 查看当前目录
pwd
# 进入web目录Dockerfil所在目录下,scm-web是我项目目录，Dockerfile文件在该目录下
cd scm-web
# 查看当前目录
pwd
echo "# 登陆阿里云docker私有仓库"
docker login --username=${username} --password=${password} registry.cn-hangzhou.aliyuncs.com
echo "# 打镜像tag"
docker -H tcp://172.17.0.1:2375 build -t registry.cn-hangzhou.aliyuncs.com/hopefun/w2002:$IMAGE_NAME-${BUILD_NUMBER} $WORKSPACE/scm-web
echo "# 推送到仓库"
docker -H tcp://172.17.0.1:2375 push registry.cn-hangzhou.aliyuncs.com/hopefun/w2002:$IMAGE_NAME-${BUILD_NUMBER}
echo "# 删除本地镜像"
docker -H tcp://172.17.0.1:2375 rmi registry.cn-hangzhou.aliyuncs.com/hopefun/w2002:$IMAGE_NAME-${BUILD_NUMBER}
echo "# 退出私有仓库"
docker logout 

echo '================Docker结束=============='

echo "部署镜像 ===============>>>>>>>>>>>>>>>>>>>>>>> registry.cn-hangzhou.aliyuncs.com/hopefun/w2002:$IMAGE_NAME-${BUILD_NUMBER}"
```

至此就在Jenkins中应用保存。

接下来直接构建，查看控制台输出：

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200923214015.png)

我们在看下阿里云容器服务的镜像仓库有没有

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200923214126.png)

BingGo！！！

打完收工~~~
