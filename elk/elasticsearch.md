# Elasticsearch入门

## 一、什么是Elasticsearch

Elaticsearch，简称为es，es是一个开源的高扩展的**分布式全文检素引擎**，它可以近乎实时的存储、检索数据。

本身扩展性很好，可以扩展到上百台服务器，处理PB级别（大数据时代）的数据。

es使用lava开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful APl来隐藏Lucene的复杂性，从而让全文搜素变得简单。

官网地址：https://www.elastic.co/cn/elasticsearch/

## 二、安装

### 2.1、linux版本安装

```sh
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.7.0-linux-x86_64.tar.gz
tar -xzvf elasticsearch-7.7.0-linux-x86_64.tar.gz
cd elasticsearch-7.7.0
./bin/elasticsearch
```

**注意：需要先安装JDK环境**

### 2.2、ik分词器

- **es内置分词器**

    >**standard 分词器：**（默认的）他会将词汇单元转换成小写形式，并去除停用词和标点符号，支持中文采用的方法为单字切分。
    >
    >**simple 分词器**：首先会通过非字母字符来分割文本信息，然后将词汇单元统一为小写形式。该分析器会去掉数字类型的字符。
    >
    >**Whitespace 分词器：** 仅仅是去除空格，对字符没有lowcase化,不支持中文； 并且不对生成的词汇单元进行其他的标准化处理。
    >
    >**language 分词器：**特定语言的分词器，不支持中文。
- **ik分词器**

    >**（一）下载**
    >
    >下载地址：[下载](https://github.com/medcl/elasticsearch-analysis-ik) 对应ES版本的分词器。
    >
    >**（二）解压**
    >
    >```shell
    >unzip elasticsearch-analysis-ik-master.zip
    >```
    >
    >（三）编译
    >
    >```shell
    >mvn clean install -Dmaven.test.skip=true
    >```
    >
    >（四）安装
    >
    >在ES的plugins文件夹下创建目录ik，将编译后生成的elasticsearch-analysis-ik-版本.zip移动到ik下，并解压。解压后的内容移动到ik目录下。

### 2.3、kibana安装

TODO

## 三、ES核心概念

### 3.1、名词解释

- **索引(Index)**

> ES将数据存储于一个或多个索引中，索引是具有类似特性的文档的集合。
>
> 类比传统的关系型数据库领域来说，索引相当于SQL中的一个数据库，或者一个数据存储方案(schema)。
>
> 索引由其名称(必须为全小写字符)进行标识，并通过引用此名称完成文档的创建、搜索、更新及删除操作。
>
> 一个ES集群中可以按需创建任意数目的索引。

- **类型(Type)**

> 类型是索引内部的逻辑分区(category/partition)，然而其意义完全取决于用户需求。因此，一个索引内部可定义一个或多个类型(type)。
>
> 一般来说，类型就是为那些拥有相同的域的文档做的预定义。
>
> 例如，在索引中，可以定义一个用于存储用户数据的类型，一个存储日志数据的类型，以及一个存储评论数据的类型。
>
> 类比传统的关系型数据库领域来说，类型相当于“表”。

- **文档(Document)**

> 文档是Lucene索引和搜索的原子单位，它是包含了一个或多个域的容器，基于JSON格式进行表示。
>
> 文档由一个或多个域组成，每个域拥有一个名字及一个或多个值，有多个值的域通常称为“多值域”。
>
> 每个文档可以存储不同的域集，但同一类型下的文档至应该有某种程度上的相似之处。

- **映射(Mapping)**

> ES中，所有的文档在存储之前都要首先进行分析。
>
> 用户可根据需要定义如何将文本分割成token、哪些token应该被过滤掉，以及哪些文本需要进行额外处理等等。
>
> 另外，ES还提供了额外功能，例如将域中的内容按需排序。
>
> 事实上，ES也能自动根据其值确定域的类型。

- **节点(Node)**

> 运行了单个实例的ES主机称为节点，它是集群的一个成员，可以存储数据、参与集群索引及搜索操作。
>
> 类似于集群，节点靠其名称进行标识，默认为启动时自动生成的随机Marvel字符名称。
>
> 用户可以按需要自定义任何希望使用的名称，但出于管理的目的，此名称应该尽可能有较好的识别性。
>
> 节点通过为其配置的ES集群名称确定其所要加入的集群。

- **分片(Shard)**

> ES的分片(shard)机制可将一个**索引内部的数据分布地存储于多个节点**。
>
> 它通过将一个索引切分为多个底层物理的Lucene索引完成索引数据的分割存储功能，这每一个物理的Lucene索引称为一个分片(shard)。
>
> 每个分片其内部都是一个全功能且独立的索引，因此可由集群中的任何主机存储。
>
> 创建索引时，用户可指定其分片的数量，默认数量为5个。 

- **副本(Replica)**

> Shard有两种类型：primary和replica，即主shard及副本shard。
>
> Primary shard用于文档存储，每个新的索引会自动创建5个Primary shard。
>
> 当然此数量可在索引创建之前通过配置自行定义，不过，一旦创建完成，其Primary shard的数量将不可更改。
>
> Replica shard是Primary Shard的副本，用于冗余数据及提高搜索性能。
>
> 每个Primary shard默认配置了一个Replica shard，但也可以配置多个，且其数量可动态更改。
>
> ES会根据需要自动增加或减少这些Replica shard的数量。

ES集群可由多个节点组成，各Shard分布式地存储于这些节点上。

ES可自动在节点间按需要移动shard，例如增加节点或节点故障时。简而言之，分片实现了集群的分布式存储，而副本实现了其分布式处理及冗余功能。

### 3.2、倒排索引

TODO

## 四、操作

### 4.1、交互方式

- **命令行请求**

```sh
curl -X <VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

**VERB**包括`GET`, `POST`, `PUT`, `HEAD`, or `DELETE`

**PROTOCOL**使用http

- **Kibana**

```sh
<VERB> /<PATH>?<QUERY_STRING>
```

_**以下命令均使用Kibana查询**_

### 4.2、常用的查询命令

- **查看集群支持的命令**

    ```http
    GET /_cat
    ```

> 每个命令都支持使用?v参数，来显示表头。
>
> ```http
> GET /_cat/nodes?v
> ```
>
> 每个命令都支持使用help参数，来输出可以显示的列。
>
> ```http
> GET /_cat/nodes?help
> ```
>
> 通过h参数，可以指定输出的字段。
>
> ```http
> GET /_cat/nodes?h=id,pid,version
> ```
>
> 使用管道命令将结果传递给其他工具，比如 `sort` 、 `grep` 或者 `awk`
>
> ```sh
> curl 'localhost:9200/_cat/indices?bytes=b' | sort -rnk8
> ```
>
> 使用bytes参数，返回可读性的大小数字，比如使用mb或者kb来表示。b表示已字节输出。
>
> ```http
> GET /_cat/indices?bytes=b
> ```

- **查看所有节点信息**

    ```http
    GET /_cat/nodes?v
    ```

- **查看master节点信息**

    ```HTTP
    GET /_cat/master?v
    ```

- **查看所有节点上的热点线程**

    ```http
    GET /_nodes/hot_threads
    ```

- **查看有问题的分片或索引**

    ```HTTP
    GET /_cluster/allocation/explain?pretty
    ```

- **查看线程池设置**

    ```HTTP
    GET /_nodes/thread_pool
    ```

- **统计全部信息**

    ```http
    GET /_cluster/stats?human&pretty
    ```

- **查看集群健康状态**

    ```http
    GET /_cluster/health?pretty
    ```

	- green：所有功能都是完好的;
	- yellow：所有数据是可用的，但是一些副本还没有被分配;
	- red：代表一些数据由于某些原因已经不可用。
	- 注意：尽管一个集群是red状态，它仍然可以提供部分服务（比如，它会继续从可用的切片数据里搜索），但是在失去部分数据后，需要尽快去修复。


### 4.2、关于索引的操作

- **创建索引**

```HTTP
PUT /test
{
    "settings": {
        "index": {
            "number_of_shards": 3,  // 设置分片的数量
            "number_of_replicas": 2  // 设置每个分片副本的数量
        }
    },
    "mappings": {		// 映射关系设置
        "properties": {
            "name": {	// 字段名称
                "type": "text"
            },
            "age": {
                "type": "short"
            },
            "city": {
                "type": "keyword"
            }
        }
    },
    "aliases": {	// 设置别名
        "alias_1": {},	// 别名名称
        "alias_2": {
            "filter": {
                "term": {
                    "user.id": "kimchy"
                }
            },
            "routing": "shard-1"
        }
    }
}
```

**返回结果：**

```json
{
    "acknowledged": true,	-- 索引在集群中是否被成功创建
    "shards_acknowledged": true,	-- 是否在超时之前为索引中的每个分片启动了必要数量的分片副本
    "index": "test"
}
```

**注1：** ***acknowledged 或 shards_acknowledged 返回 false，但索引创建成功。返回值仅仅表示在超时前完成了操作。***

- **删除索引**

```http
DELETE /test  
```

- **查询所有的索引**

```http
GET /_cat/indices?v&bytes=b
```
- **查询单个索引信息**

```http
GET /test
```

- **查询索引是否存在** 

```http
HEAD /test
```

- **导入索引**

```sh
curl -u elastic -H "Content-Type: application/json" -XPOST "IP:9200/accounts/_bulk" --data-binary "@accounts.json"
```

**注1：** ***test为索引名称。***

**注2：** ***7.x默认type为`_doc`，可以省略不写。（8.x以后可能要去掉type）***

**注3：** ***accounts.json下载地址*** [accounts.json](https://github.com/elastic/elasticsearch/blob/7.7/docs/src/test/resources/accounts.json)

### 4.3、关于文档的操作









**查询bank下面所有的文档**

```console
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```

**分页查询**

```console
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10, # 开始的位置
  "size": 10  # 查询的总条数
}
```

**查询address中有mill或lane的数据**

```console
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```

> 要执行词组搜索而不是匹配单个词，请使用 `match_phrase`代替`match`

**属于40岁客户的帐户，但不包括居住在爱达荷州（ID）的任何人**

```console
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

要构造更复杂的查询，可以使用`bool`查询来组合多个查询条件。您可以根据需要（must match），期望（should match）或不期望（must not match）指定条件。

**查询账户金额在2万到3万的用户**

```json
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

**聚合操作**

```console
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

**创建一个文档**

```console
PUT /customer/_doc/1
{
  "name": "John Doe"
}
```

>customer  文档所在的index
>
>_doc 文档所在的type
>
>1 文档的id
>
>{ } 文档的内容
>
>**如果索引不存在会自动创建**

查询一个文档

```
GET /customer/_doc/1
```

批量插入文档

```sh

```



聚合查询

```json
{
    "aggs" : { // 聚合操作
        "live_group" : { // 名称。随便取
            "terms" : { // 分组  avg-平均值
                "field" : "first_category" // 分组字段
            }
        }
    },
    "size" : 0 // 结果不返回原始数据
}

```

映射关系

```json
{
	"properties" : { // 映射关系关键字
		"name" : {	// 字段名称
			"type" : "text", // test可以被分词
			"index" : true // 是否能被搜索
		},
		"sex" : {
            "type" : "keyword", // 不能被分词，只能被完全匹配
            "index" : true
        },
        "tel" : {
            "type" : "keyword",
            "index" : false
        }
	}
}
```



