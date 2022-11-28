---
title: 【环境安装】ElasticSearch安装
tags:
  - ElasticSearch
  - ES
categories:
  - 环境集成
---
# 【环境安装】ElasticSearch安装

**ElasticSearch官方下载地址：https://www.elastic.co/cn/downloads/elasticsearch**

```shell
#下载官方安装包
wget -b https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.7.1-linux-x86_64.tar.gz
#解压安装包
tar -zxvf elasticsearch-7.7.1-linux-x86_64.tar.gz
#修改所属组和所属用户（注意：当前目录所属一定不要是root）
adduser es #创建ES用户
chown -R es ${es解压所在目录}
#切换用户
su es
```

如需下载其他版本的点击这里：

![](https://i.loli.net/2020/06/06/7bNiQrTAcyjpLdO.png)

选择所需要下载的版本

![](https://i.loli.net/2020/06/06/4bvzXuQqL8deapw.png)

## ElasticSearch文件目录

| 目录    | 配置文件          | 描述                                                      |
| ------- | ----------------- | --------------------------------------------------------- |
| bin     |                   | 脚本文件，包括启动ElasticSearch，安装插件。运行统计数据等 |
| config  | elasticsearch.yml | 集群配置文件，user，role based相关配置                    |
| JDK     |                   | Java运行环境                                              |
| data    | path.data         | 数据文件                                                  |
| lib     |                   | Java类库                                                  |
| logs    | path.log          | 日志文件                                                  |
| modules |                   | 包含所有ES模块                                            |
| plugins |                   | 包含所有已安装插件                                        |

## JVM 配置

+ 修改JVM - config/jvm.options
  + 7.1 下载默认设置是 1 GB
+ 配置的建议
  + Xms和Xmx设置成一样大
  + Xmx不要超过机器内存的30%
  + 不要超过30GB

## ElasticSearch插件

在命令行输入：

```shell
#安装插件
bin/elasticsearch-plugin install analysis-icu
#查看插件
bin/elasticsearch-plugin list
#通过URL请求查看安装的插件
GET http://localhost:9200/_cat/plugins?v
```

## ElasticSearch启动

单实例启动

配置elasticsearch.yml

```yaml
cluster.name: my-nodes 						#集群名称
node.name: node-1 							#节点名称
path.data: /opt/elasticsearch-7.6.2/data 	#数据
path.logs: /opt/elasticsearch-7.6.2/logs 	#日志
network.host: 0.0.0.0 						#ip访问限制
discovery.seed_hosts: ["192.168.1.70"] 		#主机Host列表
cluster.initial_master_nodes: ["node-1"] 	#节点名，单机版只有一个
geteway.recover_after_nodes: 1 				#需要被发现的节点数
```

跨域访问

```shell
#跨域访问的配置方法，找到config文件夹下的elasticsearch.yml，在文件的末尾添加如下内容
http.cors.enabled: true 
http.cors.allow-origin: "*"
```

利用AJAX跨域修改数据的方法如下：

```javascript
var baseUrl = 'http://localhost:9200/',
    putUrl = baseUrl + 'events/logon_event/2',
    datas = {
        "name": "郭富城",
        "occur_time": "2015-05-09",
        "enabled": false,
        "times": 22
    };
//  提交数据内容
$.ajax(putUrl, {
    type : 'PUT',
    //  一定要设置contentType，以payload方式提交
    contentType: 'application/json',
    //  数据一定转换为字符串
    data : JSON.stringify(datas),
    success : function(datas) {
        console.log(datas)
    }
})
```

现在就可以用javascript来制造数据了。

**结论：**ElasticSearch以payload的方式提交数据，所以设置“Content-Type”头信息是AJAX执行成功的关键。

配置好后启动

```shell
bin/elasticsearch
```

多实例启动

```shell
#node.name ：节点名称
#cluster.name ：集群名称
#path.data：es数据存储目录
#network.host：监听IP，默认127.0.0.1，只允许本地连接
#http.port：端口，默认是9200，9201，9202三个端口
#cluster.initial_master_nodes：配置集群内可选为主节点的节点，这里设置三个节点，配置network.host后，必
./bin/elasticsearch -E node.name=node01 -E cluster.name=Practice -E path.data=/opt/elasticsearch-7.6.2/data/node01/ -E network.host=0.0.0.0 -E cluster.initial_master_nodes=node01,node02,node03 -d 
./bin/elasticsearch -E node.name=node02 -E cluster.name=Practice -E path.data=/opt/elasticsearch-7.6.2/data/node02/ -E network.host=0.0.0.0 -E cluster.initial_master_nodes=node01,node02,node03 -d
./bin/elasticsearch -E node.name=node03 -E cluster.name=Practice -E path.data=/opt/elasticsearch-7.6.2/data/node03/ -E network.host=0.0.0.0 -E cluster.initial_master_nodes=node01,node02,node03 -d
```

启动完成后打开地址：http://ip:port

```json
{
name: "node02",
cluster_name: "Practice",
cluster_uuid: "PqERjAFRSvqVloFf1jiqPw",
version: {
number: "7.6.2",
build_flavor: "default",
build_type: "tar",
build_hash: "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
build_date: "2020-03-26T06:34:37.794943Z",
build_snapshot: false,
lucene_version: "8.4.0",
minimum_wire_compatibility_version: "6.8.0",
minimum_index_compatibility_version: "6.0.0-beta1"
},
tagline: "You Know, for Search"
}
```

http://ip:port

## _cat

Elasticsearch中信息很多，如果单凭肉眼来寻找复杂数据之间的关系，是很困难的。因此cat命令应运而生，它帮助开发者快速查询Elasticsearch的相关信息。

### _cat命令

通过使用_cat可以查看支持的命令：

```
$ curl localhost:9200/_cat
=^.^=
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
```

### verbose 

每个命令都支持使用?v参数，来显示详细的信息：

```
$ curl localhost:9200/_cat/master?v
id                     host      ip        node
QG6QrX32QSi8C3-xQmrSoA 127.0.0.1 127.0.0.1 Manslaughter
```

### help

每个命令都支持使用help参数，来输出可以显示的列：

```
$ curl localhost:9200/_cat/master?help
id   |   | node id
host | h | host name
ip   |   | ip address
node | n | node name
```

### headers

通过h参数，可以指定输出的字段：

```
$ curl localhost:9200/_cat/master?v
id                     host      ip        node
QG6QrX32QSi8C3-xQmrSoA 127.0.0.1 127.0.0.1 Manslaughter

$ curl localhost:9200/_cat/master?h=host,ip,node
127.0.0.1 127.0.0.1 Manslaughter
```

### 数字类型的格式化

很多的命令都支持返回可读性的大小数字，比如使用mb或者kb来表示。

```
$ curl localhost:9200/_cat/indices?v
health status index pri rep docs.count docs.deleted store.size pri.store.size
yellow open   test    5   1          3            0      9.kb          9.kb
```
