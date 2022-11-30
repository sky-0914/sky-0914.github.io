---
title: 【环境安装】MongoDB安装
tags:
  - MongoDB
categories:
  - 环境集成
---
# 【环境安装】MongoDB安装

## 副本集

**去官网下载MongoDB安装压缩包**
```shell script
# 下载
wget https://xxxxx.com/xxx.tar
# 解压
tar -zxvf xxx.tar
# 创建目录
mkdir -p /mongodb/bin
# 进去解压后的mongodb，bin目录
cd xxx/bin
# 复制bin下的文件到刚刚创建的bin目录下
cp * /mongodb/bin
```

**创建目录**
```shell script
for  i in 37777 37778 37779
    do 
      mkdir -p /mongodb/$i/conf
      mkdir -p /mongodb/$i/data
      mkdir -p /mongodb/$i/log
done
```

**创建配置**
```shell script
cat >>/mongodb/37777/conf/mongod.conf<<'EOF'
systemLog:
  destination: file
  path: /mongodb/37777/log/mongodb.log
  logAppend: true
storage:
  journal:
    enabled: true
  dbPath: /mongodb/37777/data
  directoryPerDB: true
  wiredTiger:
    engineConfig:
      directoryForIndexes: true
    collectionConfig:
      blockCompressor: zlib
    indexConfig:
      prefixCompression: true
processManagement:
  fork: true
net:
  port: 37777
replication:
  oplogSizeMB: 2048
  replSetName: mongo_data #名称
EOF
```

**复制配置**
```shell script
for i in 37778 37779
  do  
   \cp  /mongodb/37777/conf/mongod.conf  /mongodb/$i/conf/
done
```

**修改配置**
```shell script
for i in 37778 37779
  do 
    sed  -i  "s#37777#$i#g" /mongodb/$i/conf/mongod.conf
done
```

**启动**
```shell script
for i in 37777 37778 37779
  do  
    mongod -f /mongodb/$i/conf/mongod.conf  --bind_ip_all
done
```

**关闭**
```shell script
for i in 37777 37778 37779
   do  
     mongod --shutdown  -f /mongodb/$i/conf/mongod.conf 
done
```

**mongodb日志文件分隔删除**
```shell script
# 先切割日志文件
vim mongodb-log.sh
#!/bin/bash
for pid in $(pidof /mongodb/bin/mongod)
 do
	kill -SIGUSR1 $pid
done 
# 再删除7天之前的文件
for i in 37777 37778 37779
  do  
    find /mongodb/37777/log/testlog/ -mtime 7 -delete
done
exit

# 保存退出：:wq
# 添加执行文件
chmod -x mongodb-log.sh

# 添加linux定时器 crontab
crontab -e
59 23 * * * /mongodb/mongodb-log.sh # 每天23点59分执行一次

```

**配置mongodb集群**
```shell script
mongo --port 37777 #连接mongodb
```
```mongojs
config = {_id: 'mongo_data', members: [
                          {_id: 0, host: '${ip}:37777'},
                          {_id: 1, host: '${ip}:37778'},
                          {_id: 2, host: '${ip}:37779'}]
          }
rs.initiate(config) //初始化这个配置
rs.status()         //查看整体复制集状态
rs.isMaster()       //查看当前是否是主节点

rs.add("${ip}:${port}")    // 新增从节点
rs.addArb("${ip}:${port}") // 新增仲裁节点
rs.remove("${ip}:${port}") // 删除一个节点
rs.reconfig(cfg)           // 重写复制集配置
```
**修改集群IP**
```shell
# 获取副本集配置
replication:OTHER> cfg=rs.conf()
replication:OTHER> printjson(cfg)
# 查看配置节点
replication:OTHER> printjson(cfg.members[0])
# 修改节点端口
replication:OTHER> cfg.members[0].host="192.168.249.181:27000"
replication:OTHER> cfg.members[1].host="192.168.249.181:27001"
replication:OTHER> cfg.members[2].host="192.168.249.181:27002"
# 重新配置
replication:OTHER> rs.reconfig(cfg)    # 可能会报错"errmsg" : "replSetReconfig should only be run on PRIMARY, but my state is REMOVED; use the \"force\" argument to override",
# 报错的话，根据报错信息加强制指令执行
replication:OTHER> rs.reconfig(cfg, {force : true})
```


