---
title: 【环境安装】Kibana安装
tags:
  - Kibana
categories:
  - 环境集成
---
# 【环境安装】Kibana安装

**Kibana官方下载地址：https://www.elastic.co/cn/kibana**

```shell
#下载官方安装包
wget -b https://artifacts.elastic.co/downloads/kibana/kibana-7.7.1-linux-x86_64.tar.gz
#解压安装包
tar -zxvf kibana-7.7.1-linux-x86_64.tar.gz
#修改所属组和所属用户（注意：当前目录所属一定不要是root）
adduser kibana #创建kibana用户
chown -R kibana ${kibana解压所在目录}
#切换用户
su kibana
```

Kibana配置文件

```yaml
server.port: 5601 									#启动端口号
server.host: "0.0.0.0"								#ip访问限制
elasticsearch.hosts: ["http://192.168.1.70:9200"]	#配置ES连接地址
elasticsearch.requestTimeout: 50000 				#Kibana连接ES超时时，可修改
#Kibana的汉化，从Kibana 6.8 和 ES 7.0开始支持。通过在 kibana.yml文件中，增加
i18n.locale: "zh-CN" #就支持中文显示了。
```

进入bin目录下，启动

```shell
bin/kibana
# 后台启动
nohup bin/kibana &
```

插件安装

```shell
bin/kibana-plugin list					#查看已安装插件
bin/kibana-plugin install ${插件名称}	  #安装插件
bin/kibana-plugin remove				#卸载插件
```

