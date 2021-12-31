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
    >**language 分词器：** 特定语言的分词器，不支持中文。
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

## 四、API

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
    >
    > 使用pretty参数，格式化返回json的
    >
    > ```http
    > GET /bank/_search?pretty
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

**注2：** ***7.x默认type为`_doc`，可以省略不写。（8.x以后可能要去掉 type）***

**注3：** ***accounts.json下载地址*** [accounts.json](https://github.com/elastic/elasticsearch/blob/7.7/docs/src/test/resources/accounts.json)

- **打开/关闭索引**

```http
POST /test/_close
POST /test/_open
```

### 4.3、关于文档的操作

- **新增文档**

```http
POST /test/_doc/1
{
	"name": "John Doe"	// 字段名称：值
}
```

>**注释：**
>
>- ##### 使用PUT来创建文档，需要指定id
>
>- ##### 使用POST来创建文档，可以不指定id（不指定时随机生成id），指定 id存在就会修改这个数据，并新增版本号；
>
>- test表示索引名称。
>
>
>- _doc表示类型。
>- **如果索引不存在会自动创建**


- **删除文档**

**根据id删除文档**

```http
DELETE /test/_doc/1
```

**根据匹配的条件删除文档**

```http
POST /test/_delete_by_query
{
    "query": {
        "match": {
            "name": "pjjlt"
        }
    }
}
```

- **替换文档**

```http
PUT /test/_doc/1
{
	"name": "John d"
}
```


- **修改文档**

**根据主键修改**

```http
POST /test/_update/1
{
	"doc": {
		"name": "中文名称"
	}
}
```

**根据查询条件修改**

```http
POST /test/_update_by_query
{
    "script": {
        "source": "ctx._source['age']=5"
    },
    "query": {
        "bool": {
            "must": [
                {
                    "exists": {
                        "field": "age"
                    }
                }
            ]
        }
    }
}
```


- **查询所有的文档**

```http
GET /bank/_search
{
	"query": { "match_all": {} },
}
```

- **主键查询文档**

```http
GET /bank/_doc/1
```

- **分页、排序查询**

```http
GET /bank/_search
{
	"query": { "match_all": {} },
    "sort": [
    	{ 
    		"account_number": "asc" 
    	}
    ],
    "from": 10, # 开始的位置
    "size": 10  # 查询的总条数
}
```

- **匹配、高亮数据**

```http
GET /bank/_search
{
	"query": { 
		"match": { 
			"address": "mill lane" 
		} 
	},
	"highlight": {	// 高亮查询
		"pre_tags": ["<tag1>", "<tag2>"],
    	"post_tags": ["</tag1>", "</tag2>"],
		"fields": {	
			"address": {	// 高亮的字段名称
				"type": "plain"  // unified（默认），plain和fvh（fast vector highlighter）
								 // number_of_fragments、fragment_size等写到字段里面，只对当前字段起作用
			}	
		}
	},
	"_source": ["account_number", "address"]  // 设置返回的字段名称
}
```

> 要执行词组搜索而不是匹配单个词，请使用 `match_phrase`代替`match`
>
> **高亮常用参数详解**
>
> | 字段名称            | 字段说明                                                     |
> | ------------------- | ------------------------------------------------------------ |
> | fragment_size       | 突出显示的片段的大小（以字符为单位）默认为100                |
> | fragment_offset     | 控制要开始突出显示的边距。仅在使用 fvh 有效                  |
> | no_match_size       | 如果没有要突出显示的匹配片段，则要从字段开头返回的文本量。默认为0（不返回任何内容） |
> | number_of_fragments | 要返回的最大片段数。如果片段数设置为0，则不返回任何片段。而是突出显示并返回整个字段内容。当您需要突出显示标题或地址等短文本时，这可能很方便，但不需要分段。如果number_of_fragments 为0，fragment_size则忽略。默认为5 |
> | pre_tags            | 与结合使用post_tags来定义用于突出显示文本的HTML标记。默认情况下，高亮文本被包裹在```<em></em>```标签。指定为字符串数组 |
> | post_tags           | 与结合使用pre_tags来定义用于突出显示文本的HTML标记。默认情况下，高亮文本被包裹在```<em></em>```标签。指定为字符串数组 |

- **构建复杂查询**

```http
GET /bank/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "age": "40"
                    }
                }
            ],
            "must_not": [
                {
                    "match": {
                        "state": "ID"
                    }
                }
            ]
        }
    }
}
```

> 要构造更复杂的查询，可以使用`bool`查询来组合多个查询条件。
>
> 您可以根据需要（must match），期望（should match）或不期望（must not match）指定条件。

- **范围查询**

```http
GET /bank/_search
{
    "query": {
        "bool": {
            "must": {
                "match_all": {}
            },
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

- **聚合操作**

```http
GET /bank/_search
{
    "size": 0,
    "aggs": {
        "group_by_state": {
            "terms": {	// terms-分组
                "field": "state.keyword",
                "order": {
                    "average_balance": "desc"
                }
            },
            "aggs": {		// 聚合操作
                "average_balance": { // 名称。随便取
                    "avg": {		 // avg-平均值，min-最小、max-最大、sum-和
                        "field": "balance"	// 计算的字段
                    }
                }
            }
        }
    },
	"size" : 0 // 结果不返回原始数据
}
```

- **设置映射关系**

```http
PUT /bank/_mapping
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

### 4.4、分词结果分析

```http
GET /_analyze
{
	"analyzer" : "ik_smart", 
	"text" : "床前明月光"
}
```

**参数解析：**

>analyzer：分词模式。ik分词器有两种模式 **ik_smart**、**ik_max_word**
>
>text：分词的文本内容。

**返回结果解析：**

>```json
>{
>    "tokens": [
>        {
>            "token": "床",
>            "start_offset": 0,
>            "end_offset": 1,
>            "type": "CN_CHAR",
>            "position": 0
>        },
>        {
>            "token": "前",
>            "start_offset": 1,
>            "end_offset": 2,
>            "type": "CN_CHAR",
>            "position": 1
>        },
>        {
>            "token": "明月光",
>            "start_offset": 2,
>            "end_offset": 5,
>            "type": "CN_WORD",
>            "position": 2
>        }
>    ]
>}
>```
>
>token： 一个实际被存储在索引中的词 。
>
>position：表示该词在原文本中是第几个出现的 。
>
>start_offset：表示词在原文本中的起始位置的下表。下表从1开始
>
>end_offset：表示词在原文本中的结束位置的下表。

### 4.5、批处理操作

- **批量新增、修改、删除**

```http
POST /_bulk
{ "update" : { "_index" : "user", "_id" : "1" } }
{ "doc" : {"age" : 18} }
{ "create" : { "_index" : "user", "_id" : "2" } }
{ "name" : "小明","age":32,"city":"beijing" }
{ "create" : { "_index" : "user", "_id" : "3" } }
{ "name" : "小红","age":21,"city":"sjz" }
{ "create" : { "_index" : "user", "_id" : "4" } }
{ "name" : "mark","age":22,"city":"tianjin" }
{ "delete" : { "_index" : "user", "_id" : "4" } }
```

>- 以上命令对 user 索引 更新了id为1文档的年龄，新增id为2、3、4的文档，再删除id为4的文档。
>
>- **bulk的格式：**
>    { action : { metadata } }
>    { requstbody }
>
>    action（行为）：包含create（文档不存在时创建）、update（更新文档）、index（创建新文档或替换已用文档）、delete（删除一个文档）。
>    metadata（行为操作的具体索引信息）：需要指明数据的_index、_type、_id。
>
>- create 和 index 的区别：如果数据存在，使用 create 操作失败，会提示文档已存在，使用 index 则可以成功执行。
>
>- delete 不需要 requestbody 在下一行。create、update、index 必须有 requestbody

- **批量查询**

```http
GET /_mget
{
    "docs": [
        {
            "_index": "accounts",
            "_id": "1"
        },
        {
            "_index": "accounts",
            "_id": "5"
        }
    ]
}
```

还可以简写形式

```http
POST /accounts/_mget
{
	"ids": ["1", "2", "3"]
}
```

## 五、DSL查询

#### match

>match是一个标准查询，当查询一个文本的时候，会先将文本分词。当查询确切值的时候，会搜索给定的值，例如数字、日期、布尔或者被not_analyzed的字符串。
>
>```json
>{
>    "query": {
>        "match": {
>            "name": "小明"
>        }
>    }
>}
>```
>
>上面的操作会先将“小明”分词为“小”、“明”（当然具体还要看你的分词器），然后再去所有文档中查找与之相匹配的文档，并根据关联度排序返回。

#### match_phrase

>match_phrase会保留空格，match会把空格忽略。
>
>```json
>{
>    "query": {
>        "match_phrase": {
>            "name": "小 明"
>        }
>    }
>}
>```
>
>注意，分词是空格会给前一个元素，比如上面的字符串分子之后是，“小 ”，“明”。

#### multi_match

>多字段查询，一个查询条件，看所有多个字段是否有与之匹配的字段。后面我们也可以使用`should`更加灵活。
>
>```json
>{
>    "query": {
>        "multi_match": {
>            "query": "哈哈",
>            "fields": ["name", "city"]
>        }
>    }
>}
>```

#### match_all

>匹配所有，并可设置这些文档的`_score`，默认`_score`为1，这里没有计算`_score`，所以速度会快很多。
>
>```json
>{
>    "query": {
>        "match_all": {
>            "boost": 1.2
>        }
>    }
>}
>```
>
>`boost`参数可以省略，默认是1。

#### term

>term是一种完全匹配，主要用于精确查找，例如数字、ID、邮件地址等。
>
>```json
>{
>    "query": {
>        "term": {
>            "age": 18
>        }
>    }
>}
>```

#### terms

>terms是term多条件查询，参数可以传递多个，以数组的形式表示。
>
>```json
>{
>        "query": {
>             "terms": {
>                 "age": [18, 21]
>             }
>        }
>}
>```

#### wildcard

>通配符，看示例容易理解，通配符可以解决分词匹配不到的问题，例如'haha' 可以通过'*a'匹配。
>
>```json
>{
>        "query": {
>             "wildcard": {
>                 "name": "*a"
>             }
>        }
>}
>```

#### exists

>查看某文档是否有某属性，返回包含这个`Filed`的文档。
>
>```json
>{
>        "query": {
>             "exists": {
>                 "field": "name"
>             }
>        }
>}
>```

#### fuzzy

>返回与查询条件相同或者相似的匹配内容。
>
>```json
>{
>        "query": {
>             "fuzzy": {
>                 "name": "mjjlt"
>             }
>        }
>}
>```
>
>搜索条件是`mjjlt`，可以搜出来name为`pjjlt`的文档。

#### ids

>多id查询，这个id是主键id，即你规定或者自动生成那个。
>
>```json
>{
>        "query": {
>             "ids": {
>                 "values": [1, 2, 3]
>             }
>        }
>}
>```

#### prefix

>前缀匹配
>
>```json
>{
>        "query": {
>             "prefix": {
>                 "name": "pj"
>             }
>        }
>}
>```

#### range

>范围匹配。参数可以是 **gt**(大于)、**gte**(大于等于)、**lt**(小于)、**lte**(小于等于)
>
>```json
>{
>        "query": {
>             "range": {
>                 "age": {
>                     "gt": 1,
>                     "lt": 30
>                 }
>             }
>        }
>}
>```

#### regexp

>正则匹配。value是正则表达式，flags是匹配格式，默认是ALL，开启所有。更多格式[请戳](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/regexp-syntax.html#regexp-optional-operators)
>
>```json
>{
>        "query": {
>             "regexp": {
>                 "name": {
>                     "value": "p.*t",
>                     "flags": "ALL"
>                 }
>             }
>        }
>}
>```

#### bool

>bool 可以用来组合其他子查询。其中常包含的子查询包含：must、filter、should、must_not

- **must**

    > `must`内部的条件必须包含，内部条件是`and`的关系。如查看所有name中包含“小”并且age是32的用户文档。
    >
    > ```json
    > {
    >     "query": {
    >         "bool": {
    >             "must": [
    >                 {
    >                     "term": {
    >                         "name": "小"
    >                     }
    >                 },
    >                 {
    >                     "term": {
    >                         "age": 32
    >                     }
    >                 }
    >             ]
    >         }
    >     }
    > }
    > ```

- **filter**

    >`filter`是文档通过一些条件过滤下，这是四个关键词中唯一**和关联度无关的，不会计算_score**，经常使用的过滤器会产生缓存。
    >
    >```json
    >{
    >   "query": {
    >        "bool": {
    >            "filter": {
    >                "term": {
    >                    "name": "小"
    >                }
    >            }
    >        }
    >   }
    >}
    >```

- **must_not**

    >这个和`must`相反，文档某字段中一定不能包含某个值，相当于“非”。

- **should**

    >`should`可以看做`or`的关系，例如下面查询name包含"小"或者年龄是18岁的用户。
    >
    >```json
    >{
    >   "query": {
    >        "bool": {
    >            "should": [
    >                {
    >                    "term": {
    >                        "name": "小"
    >                    }
    >                },
    >                {
    >                    "term": {
    >                        "age": 18
    >                    }
    >                }
    >            ]
    >        }
    >   }
    >}
    >```

## 六、聚合查询

Elasticsearch除全文检索功能外提供的针对Elasticsearch数据做统计分析的功能。可以查询某组数据的最大最小值，**分组**查询某些数据。

- Metric(指标)：指标分析类型，如计算最大值、最小值、平均值等等 （对桶内的文档进行聚合分析的操作）
- Bucket(桶)：分桶类型，类似SQL中的GROUP BY语法 （满足特定条件的文档的集合）
- Pipeline(管道)：管道分析类型，基于上一级的聚合分析结果进行在分析

### 6.1、Metric(指标)数据

#### 常用数学操作

> 这里常用的数学操作有`min`(最小)、`max`(最大)、`sum`(和)、`avg`(平均数)。注意这些操作只能输出一个分析结果。使用方式大同小异。
>
> ```json
> {
>        "aggs": {
>            "avg_user_age": {
>                "avg": {
>                    "field": "age"
>                }
>            }
>        }
> }
> ```

#### cardinality

>计算某字段去重后的数量
>
>```json
>{
>        "aggs": {
>             "avg_user": {
>                 "cardinality": {
>                     "field": "age"
>                 }
>             }
>        }
>}
>```

#### percentiles

>对指定字段的值按从小到大累计每个值对应的文档数的占比，返回指定占比比例对应的值。
>
>默认统计百分比为**[ 1, 5, 25, 50, 75, 95, 99 ]**
>
>```json
>{
>        "aggs": {
>             "avg_user": {
>                 "percentiles": {
>                     "field": "age"
>                 }
>             }
>        }
>}
>```

#### percentile_ranks

>percentiles是通过百分比求出文档某字段，percentile_ranks是给定文档中的某字段求百分比。
>
>```json
>{
>        "aggs": {
>             "avg_user": {
>                 "percentile_ranks": {
>                     "field": "age",
>                     "values": [18, 30]
>                 }
>             }
>        }
>}
>```

#### top_hits

>top_hits可以得到某条件下top n的文档。
>
>```json
>{
>        "aggs": {
>             "avg_user": {
>                 "top_hits": {
>                     "sort": [
>                         {
>                             "age": {
>                                 "order": "asc"
>                             }
>                         }
>                     ],
>                     "size": 1
>                 }
>             }
>        },
>        "size": 0
>}
>```

### 6.2、Bucket(桶)

类似于分组的概念。

#### terms	

>根据给定的filed分组，返回每组多少文档。
>
>```json
>{
>        "aggs": {
>             "avg_user": {
>                 "terms": {
>                     "field": "city"
>                 }
>             }
>        }
>}
>```

#### ranges

>根据区间分组
>
>```json
>{
>        "aggs": {
>             "price_ranges": {
>                 "range": {
>                     "field": "age",
>                     "ranges": [
>                         {
>                             "to": 20
>                         },
>                         {
>                             "from": 20,
>                             "to": 30
>                         },
>                         {
>                             "from": 30
>                         }
>                     ]
>                 }
>             }
>        }
>}
>```