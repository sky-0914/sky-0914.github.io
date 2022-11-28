---
title: ElasticSearch入门
tags:
  - ElasticSearch
  - ES
  - 数据库
categories:
  - 数据库
---
# ElasticSearch入门

## ElasticSearch的基本概念

+ **Index**

  > 类似于mysql数据库中的database（数据库）
  >
  > 是文档的容器，是一类文档的集合
  >
  > + Index：体现了逻辑空间的概念，每个索引都有自己的Mapping定义，用于定于包含的文档的字段名和字段类型
  > + Shard：体现了物理空间的概念，索引中的数据分散在Shard上

+ **~~Type~~**

  > ~~类似于mysql数据库中的table（表），es中可以在Index中建立type（table），通过mapping进行映射。~~
  >
  > 在7.0之前，一个Index可以设置多个Type，6.0开始Type已经被废除。7.0开始，一个索引只能创建一个Type "_doc"。

+ **Document**

  > 由于es存储的数据是文档型的，一条数据对应一篇文档即相当于mysql数据库中的一行数据row，一个文档中可以有多个字段也就是mysql数据库一行可以有多列。与MongoDB存储JSON类似。
  >
  > 文档会被序列化成JSON格式保存在ES中、每个文档都有唯一的ID（Unique ID），可以自己指定或者ES生成

+ **Field**

  > es中一个文档中对应的多个列与mysql数据库中每一列对应。字段

+ **Mapping**

  > 文档字段类型

+ **indexed**

  > 就是名义上的建立索引。mysql中一般会对经常使用的列增加相应的索引用于提高查询速度，而在es中默认都是会加上索引的，除非你特殊制定不建立索引只是进行存储用于展示，这个需要看你具体的需求和业务进行设定了。

+ **Query DSL**

  > 类似于mysql的sql语句，只不过在es中是使用的json格式的查询语句，专业术语就叫：QueryDSL

+ **GET/PUT/POST/DELETE**

  > 分别类似与mysql中的select/update/delete......

### 类比：

| 关系型数据库     | ES            |
| ---------------- | ------------- |
| Table            | Index（Type） |
| Row              | Document      |
| Column           | Field         |
| Schema（表定义） | Mapping       |
| SQL              | DSL           |

**每个文档都会有相应的元数据**

```json

```

元数据用于标注文档的相关信息：

+ _index：文档所属索引名（库名）
+ _type：文档所属类型名（表名）
+ _id：唯一ID
+ _source：原始JSON数据
+ ~~_all：整合所有内容到该字段，7.0版本已被废除~~
+ _version：版本号
+ _score：相关性打分

## 分布式系统的可用性和扩展性

+ 高可用
  - 服务可用性-允许所有节点停止服务
  - 数据可用性-部分节点丢失，不会丢失数据
+ 可扩展
  - 请求量提升/数据不断的增长（将数据分布到所有节点上）

### 分布式特性

+ ElasticSearch的分布式架构好处
  - 存储的水平扩容
  - 提高系统的可用性，部分节点停止服务，整个集群的服务不受影响
+ ElasticSearch分布式架构
  - 不同的集群通过不同的名字来区分，默认为“elasticSearch”
  - 通过配置文件可修改，或者在命令行中 ``-E cluster-name=test `` 进行设定
  - 一个集群可以有一个或多个节点

### Master-eligible nodes（Master候选者） 和 Master Node

+ 每个节点启动后，默认就是一个Master-eligible节点
  - 可以设置 ``node.master:false`` 禁止
+ Master-eligible节点可以参加选主流程，称为Master Node节点
+ 当第一个节点启动的时候，会将自己选举成Master Node节点
+ 每个节点都保存了集群的状态，只有Master节点才能修改集群状态信息（如果任意节点都能修改信息的话，那么会造成数据不一致）
  - 集群状态（Cluster State），维护了一个集群中，必要的信息
    + 所有的节点信息
    + 所有的索引和相关的Mapping和Setting信息
    + 分片的路由信息

### Date Node & Coordinating Node（数据整合节点）

+ Data Node
  + 可以保存数据的节点，叫做Data Node。负责保存分片数据。在数据扩展上起到了至关重要的作用
+ Coordinating Node
  + 负责接收Client的请求，将请求分发到合适的几点，最终把结果汇集到一起
  + 每个节点默认都起到Coordinating Node的职责

### 其他节点类型

+ Hot & Warm Node（冷热节点）
  + 不同硬件配置的Data Node，用来实现Hot & Warm架构，降低就能部署的成本
+ Machine Learning Node
  + 负责跑机器学习的Job，用来做异常检测的
+ ~~Tribe Node~~
  + （5.3开始使用Cross Cluster Serarch）Tribe Node连接到不同的Elasticsearch集群，并且支持将这些集群当成一个单例的集群处理

### 配置节点类型

+ 开发环境中一个节点可以承担多种角色
+ 生产环境中，应该设置单一的角色的节点（dedicated node）

| 节点类型          | 配置参数    | 默认值                                                    |
| ----------------- | ----------- | --------------------------------------------------------- |
| maste eligible    | node.master | true                                                      |
| data              | node.data   | true                                                      |
| ingest            | node.ingest | true                                                      |
| coordinating only | 无          | 每个节点默认都是coordinating节点，设置其他类型全部为false |
| machine learning  | node.ml     | true（需enable x-pack）                                   |

### 分片（Primary Shard & Replica Shard）

+ 主分片，用以解决数据水平扩展的问题。用过主分片，可以将数据分布到集群内的所有节点智商
  + 一个分片是一个运行的Lucene（搜索引擎）的实例
  + 主分片数在索引创建时指定，后续不允许修改，除非Reindex
+ 副本，用以解决数据高可用的问题。副本是主分片的拷贝
  + 副本分片数，可以动态调整
  + 增加副本数，还可以在一定程度上提高服务的可用性（读取的吞吐）

### 查看集群的健康状况

![image-20210309114915854](C:\Users\ZC\AppData\Roaming\Typora\typora-user-images\image-20210309114915854.png)

**在Kibana的开发工具中，快捷键按下`Ctrl+/`可以直接跳到ES官方的API中**

### 文档的CRUD

```http

#更新（先删除原数据，在新建新数据，同时版本+1）、创建
PUT {Index}/_doc/{id}
{
  "name":"张三",
  "age":18,
  "sex":"男"
}
#创建（自动生成ID）
POST {Index}/_doc
{
  "name":"张三",
  "age":18,
  "sex":"男"
}
#查询 
GET {Index}/_doc/{id}
#更新（文档必须已经存在，更新只会对相应字段做增量修改）
POST {Index}/_update/{id}
{
  "name":"张三",
  "age":18,
  "sex":"男"
}
#删除
DELETE {Index}/_doc/{id}
```

+ Type名，约定都用_doc
+ Create：如果ID已经存在，会失败
+ Index：如果ID不存在，创建新的文档，否则，先删除现有文档，再创建新的文档，版本会增加
+ Update：文档必须已经存在，更新只会对相应字段做增量修改

> Index和Create不一样的地方在于：如果文档不存在，就索引新的文档。否则现有文档会被删除，新的文档被索引。版本信息+1

### Bulk API

+ 在一次请求调用中，进行多次不同的操作，支持在一次API调用中，对不同的索引进行操作
+ 支持四种类型的操作
  + Index
  + Create
  + Update
  + Delete
+ 可以在URL中指定Index，也可以在请求的Payload中进行
+ 操作中单条操作失败，并不会影响到其他操作
+ 返回结果包括了每一条操作执行结果

```http

POST _bulk
{"index":{"_index":"test","_id":"1"}}
{"field1":"value1"}
{"delete":{"_index":"test","_id":"2"}}
{"create":{"_index":"test2","_id":"3"}}
{"field1":"value3"}
{"update":{"_index":"test3","_id":"3"}}
{"doc":{"field2":"value2"}}
```

### 批量读取-MEGT

批量操作，可以减少网络连接所产生的开销，提高性能

```http

GET _mget
{
  "docs":[
      {
        "_index":"users",
        "_id":1
      },
      {
        "_index":"users",
        "_id":2
      }
    ]
}
```

### 批量查询-MSEARCH 

```http

POST {index}/_msearch
```

### 索引：正排、倒排

正排例如书本的目录，文档ID到文档和单词的关联

倒排书本最后的索引页就是倒排索引，单词到文档ID的关系

**倒排索引的核心组成**

+ 倒排索引包含两个部分
  + 单词词典（Term Dictionary），记录所有文档的单词，记录单词到到倒排列表的关联关系
    + 单词词典一般比较大，可以通过B+树或Hash拉链法实现，以满足高性能的插入与查询
  + 倒排列表（Posting List）-记录了单词对应的文档结合，由倒排索引组成
    + 倒排索引项（Posting ）
      + 文档ID
      + 词频TF：该单词问文档中出现的次数，用于相关性评分
      + 位置（Position）：单词在文档中分词的文职。用于语句搜索
      + 偏移（Offset）：记录单词的开始结束位置，实现高亮显示

### Analysis与Analyzer（分词器）

+ Analysis：文本分析是把全文本转换一系列单词（term/token）的过程，也叫分词
+ Analysis是通过Analyzer来实现的
  + 可以使用Elasticsearch内置的分析器/或者按需自定义定制化分析器
+ 除了在数据写入时转换词条，匹配Query语句时候也需要用相同的分析器对查询语句进行分析

#### Analyzer（分词器）的组成

+ 分词器是专门处理分词的组件，Analyzer由三部分组成
  + Character Filters（针对原始文本处理，例如去除html标签）
  + Tokenizer（按照规则切分为单词）
  + Token Filter（将切分的单词进行加工，小写，删除stopwords（停用词），增加同义词）

#### Elasticsearch内置分词器

+ Standard Analyzer：默认分词器，按词切分，小写处理
+ Simple Analyzer：按照非字母切分（符号被过滤），小写处理
+ Stop Analyzer：小写处理，停用词过滤（the、a、is）
+ Whitespace Analyzer：按照空格切分，不转小写
+ Keyword Analyzer：不分词，直接降输入当做关键字输出
+ Patter Analyzer：正则表达式，默认`\W+`（非支付分隔）
+ Language：提供了30多种常见语言分词器
+ Customer Analyzer：自定义分词器

#### 使用_analyzer API

```http
# 示例


# standard：默认分词器，按词切分，小写处理
GET /_analyze
{
  "analyzer":"standard",
  "text":"2 Who are you, I am a big pig"
}

# simple：按照非字母切分，非字母都被去除，小写处理
GET /_analyze
{
  "analyzer":"simple",
  "text":"2 Who are you, I am a big pig"
}

# stop：相比Simple多了stop filter，停用词过滤（the、a、is）去除修饰性词语
GET /_analyze
{
  "analyzer":"stop",
  "text":"2 Who are you, I am a big pig"
}

# whitespace：按照空格切分
GET /_analyze
{
  "analyzer":"whitespace",
  "text":"2 Who are you, I am a big pig"
}

# keyword：关键字
GET /_analyze
{
  "analyzer":"keyword",
  "text":"2 Who are you, I am a big pig"
}

```

#### ICU Analyzer

+ 需要安装plugin
  + Elasticsearch-plugin install analysis-icu
+ 提供了Unicode的支持，更好的支持亚洲语言

#### 更多的中文分词器

+ IK
  + 支持自定义词库，支持热更新分词字典
  + https://github.com/medcl/elasticsearch-analysis-ik
+ THULAC
  + 清华大学自然语言处理和社会人文计算实验室的一套中文分词器
  + https://github.com/microbun/elasticsearch-thulac-plugin

## Search 搜索API

+ URL Search
  + 在url中使用查询参数
+ Request Body Search
  + 使用Elasticsearch提供的，基于JSON格式的更加完备的Query Domain Specific Language（DSL）

**指定查询索引**

| 语法                   | 范围              |
| ---------------------- | ----------------- |
| /_search               | 集群上所有的索引  |
| /index1/_search        | index1            |
| /index1,index2/_search | index1和index2    |
| /index*/_search        | 以index开头的索引 |

#### URL查询

`http://{ip}:{port}/{index}/_search?q={field}:{value}`

#### Request Body

```http
http://
```

