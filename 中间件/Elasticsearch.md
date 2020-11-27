[TOC]

转自：https://blog.csdn.net/makang110/article/details/80596017

# ES

## 1.1 ES定义

ES=elaticsearch简写， Elasticsearch是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。 
Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

## 1.2 Lucene与ES关系？

1）Lucene只是一个库。想要使用它，你必须使用Java来作为开发语言并将其直接集成到你的应用中，更糟糕的是，Lucene非常复杂，你需要深入了解检索的相关知识来理解它是如何工作的。

2）Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

## 1.3 ES主要解决问题：

1）检索相关数据； 
2）返回统计结果； 
3）速度要快。

## 1.4 ES工作原理

当ElasticSearch的节点启动后，它会利用多播(multicast)(或者单播，如果用户更改了配置)寻找集群中的其它节点，并与之建立连接。这个过程如下图所示： 
![这里写图片描述](../../../../Software/Typora/Picture/20160818205953345)

## 1.5 ES核心概念

### 1）Cluster：集群。

ES可以作为一个独立的单个搜索服务器。不过，为了处理大型数据集，实现容错和高可用性，ES可以运行在许多互相合作的服务器上。这些服务器的集合称为集群。-

### 2）Node：节点。

形成集群的每个服务器称为节点。

### 3）Shard：分片。

当有大量的文档时，由于内存的限制、磁盘处理能力不足、无法足够快的响应客户端的请求等，一个节点可能不够。这种情况下，数据可以分为较小的分片。每个分片放到不同的服务器上。 
当你查询的索引分布在多个分片上时，ES会把查询发送给每个相关的分片，并将结果组合在一起，而应用程序并不知道分片的存在。即：这个过程对用户来说是透明的。

### 4）Replia：副本。

为提高查询吞吐量或实现高可用性，可以使用分片副本。 
副本是一个分片的精确复制，每个分片可以有零个或多个副本。ES中可以有许多相同的分片，其中之一被选择更改索引操作，这种特殊的分片称为主分片。 
当主分片丢失时，如：该分片所在的数据不可用时，集群将副本提升为新的主分片。

### 5）全文检索。

全文检索就是对一篇文章进行索引，可以根据关键字搜索，类似于mysql里的like语句。 
全文索引就是把内容根据词的意义进行分词，然后分别创建索引，例如”你们的激情是因为什么事情来的” 可能会被分词成：“你们“，”激情“，“什么事情“，”来“ 等token，这样当你搜索“你们” 或者 “激情” 都会把这句搜出来。

## 1.6 ES数据架构的主要概念（与关系数据库Mysql对比）

![这里写图片描述](../../../../Software/Typora/Picture/20160818210034345) 
（1）关系型数据库中的数据库（DataBase），等价于ES中的索引（Index） 
（2）一个数据库下面有N张表（Table），等价于1个索引Index下面有N多类型（Type）， 
（3）一个数据库表（Table）下的数据由多行（ROW）多列（column，属性）组成，等价于1个Type由多个文档（Document）和多Field组成。 
（4）在一个关系型数据库里面，schema定义了表、每个表的字段，还有表和字段之间的关系。 与之对应的，在ES中：Mapping定义索引下的Type的字段处理规则，即索引如何建立、索引类型、是否保存原始索引JSON文档、是否压缩原始JSON文档、是否需要分词处理、如何进行分词处理等。 
（5）在数据库中的增insert、删delete、改update、查search操作等价于ES中的增PUT/POST、删Delete、改_update、查GET.

## 1.7 ELK是什么？

ELK=elasticsearch+Logstash+kibana 
elasticsearch：后台分布式存储以及全文检索 
logstash: 日志加工、“搬运工” 
kibana：数据可视化展示。 
ELK架构为数据分布式存储、可视化查询和日志解析创建了一个功能强大的管理链。 三者相互配合，取长补短，共同完成分布式大数据处理工作。

## 5. ES的应用场景是怎样的？

### 通常我们面临问题有两个：

1）新系统开发尝试使用ES作为存储和检索服务器； 
2）现有系统升级需要支持全文检索服务，需要使用ES。 
以上两种架构的使用，以下链接进行详细阐述。 
http://blog.csdn.net/laoyang360/article/details/52227541

### 一线公司ES使用场景：

1）新浪ES 如何分析处理32亿条实时日志 http://dockone.io/article/505 
2）阿里ES 构建挖财自己的日志采集和分析体系 http://afoo.me/columns/tec/logging-platform-spec.html 
3）有赞ES 业务日志处理 http://tech.youzan.com/you-zan-tong-ri-zhi-ping-tai-chu-tan/ 
4）ES实现站内搜索 http://www.wtoutiao.com/p/13bkqiZ.html

## 6. 如何部署ES？

* docker安装运行

  ```
  mkdir -p /mydata/elasticsearch/config
  mkdir -p /mydata/elasticsearch/data
  echo "http.host: 0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml
  elasticsearch:7.4.2 
  docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
  -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
  -v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
  -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
  -d elasticsearch:7.4.2
  ```



#### 6.1 ES部署（无需安装）



1）零配置，开箱即用 
2）没有繁琐的安装配置 
3）java版本要求：最低1.7 
我使用的1.8 
[root@laoyang config_lhy]# echo $JAVA_HOME 
/opt/jdk1.8.0_91 
4）下载地址： 
https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/zip/elasticsearch/2.3.5/elasticsearch-2.3.5.zip 
5）启动 
cd /usr/local/elasticsearch-2.3.5 
./bin/elasticsearch 
bin/elasticsearch -d(后台运行)

## 6.2 ES必要的插件

必要的Head、kibana、IK（中文分词）、graph等插件的详细安装和使用。 
http://blog.csdn.net/column/details/deep-elasticsearch.html

### 6.3 ES windows下一键安装

自写bat脚本实现windows下一键安装。 
1）一键安装ES及必要插件（head、kibana、IK、logstash等） 
2）安装后以服务形式运行ES。 
3）比自己摸索安装节省至少2小时时间，效率非常高。 
脚本说明： 
http://blog.csdn.net/laoyang360/article/details/51900235

## 7. ES基本使用

安装路径：/mydata/elasticsearch

##### 切换到镜像容器里：docker exec -it 335ca0f03e7c /bin/bash

```
查看索引
http://192.168.200.128:9200/_cat/indices
查看节点信息
http://192.168.200.128:9200/_cat/nodes
```



### 1、新建文档（类似mysql insert插入操作）

```
http://localhost:9200/blog/ariticle/1 put
{
"title":"New version of Elasticsearch released!",
"content":"Version 1.0 released today!",
"tags":["announce","elasticsearch","release"]
}123456
```

创建成功如下显示：

```
{
- "_index": "blog",
- "_type": "ariticle",
- "_id": "1 -d",
- "_version": 1,
- "_shards": {
    - "total": 2,
    - "successful": 1,
    - "failed": 0
- },
- "created": true

}1234567891011121314
```

![这里写图片描述](../../../../Software/Typora/Picture/20160717132135758)

### 2、检索文档（类似mysql search 搜索select*操作）

http://localhost:9200/blog/ariticle/1/ GET

检索结果如下：

```
{

- "_index": "blog",
- "_type": "ariticle",
- "_id": "1",
- "_version": 1,
- "found": true,
- "_source": {
    - "title": "New version of Elasticsearch released!",
    - "content": "Version 1.0 released today!",
    - "tags": [
        - "announce"
        - ,
        - "elasticsearch"
        - ,
        - "release"
    - ]
- }

}1234567891011121314151617181920
```

如果未找到会提示：

```
{

- "_index": "blog",
- "_type": "ariticle",
- "_id": "11",
- "found": false

}12345678
```

查询全部文档如下：
![这里写图片描述](../../../../Software/Typora/Picture/20160717132224477)
具体某个细节内容检索，
查询举例1：查询cotent列包含版本为1.0的信息。
http://localhost:9200/blog/
_search?pretty&q=content:1.0

```
{

- "took": 2,
- "timed_out": false,
- "_shards": {
    - "total": 5,
    - "successful": 5,
    - "failed": 0
- },
- "hits": {
    - "total": 1,
    - "max_score": 0.8784157,
    - "hits": [
        - {
            - "_index": "blog",
            - "_type": "ariticle",
            - "_id": "6",
            - "_score": 0.8784157,
            - "_source": {
                - "title": "deep Elasticsearch!",
                - "content": "Version 1.0!",
                - "tags": [
                    - "deep"
                    - ,
                    - "elasticsearch"
                - ]
            - }
        - }
    - ]
- }

}1234567891011121314151617181920212223242526272829303132
```

查询举例2：查询书名title中包含“enhance”字段的数据信息：
[root@5b9dbaaa1a ~]# curl -XGET 10.200.1.121:9200/blog/ariticle/_search?pretty -d ‘

```
> { "query" : {
> "term" :
> {"title" : "enhance" }
> }
> }'
{
  "took" : 189,
  "timed_out" : false,
  "_shards" : {
  "total" : 5,
  "successful" : 5,
  "failed" : 0
  },
  "hits" : {
  "total" : 2,
  "max_score" : 0.8784157,
  "hits" : [ {
  "_index" : "blog",
  "_type" : "ariticle",
  "_id" : "4",
  "_score" : 0.8784157,
  "_source" : {
  "title" : "enhance Elasticsearch!",
  "content" : "Version 4.0!",
  "tags" : [ "enhance", "elasticsearch" ]
  }
  }, {
  "_index" : "blog",
  "_type" : "ariticle",
  "_id" : "5",
  "_score" : 0.15342641,
  "_source" : {
  "title" : "enhance Elasticsearch for university!",
  "content" : "Version 5.0!",
  "tags" : [ "enhance", "elasticsearch" ]
  }
  } ]
  }
}123456789101112131415161718192021222324252627282930313233343536373839
```

查询举例3：查询ID值为3,5,7的数据信息：
[root@5b9dbaaa148a ~]# curl -XGET 10.200.1.121:9200/blog/ariticle/_search?pretty -d ‘

```
{ "query" : {
"terms" :
{"_id" : [ "3", "5", "7" ] }
}
}'
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
  "total" : 5,
  "successful" : 5,
  "failed" : 0
  },
  "hits" : {
  "total" : 3,
  "max_score" : 0.19245009,
  "hits" : [ {
  "_index" : "blog",
  "_type" : "ariticle",
  "_id" : "5",
  "_score" : 0.19245009,
  "_source" : {
  "title" : "enhance Elasticsearch for university!",
  "content" : "Version 5.0!",
  "tags" : [ "enhance", "elasticsearch" ]
  }
  }, {
  "_index" : "blog",
  "_type" : "ariticle",
  "_id" : "7",
  "_score" : 0.19245009,
  "_source" : {
  "title" : "deep Elasticsearch for university!",
  "content" : "Version 2.0!",
  "tags" : [ "deep", "elasticsearch", "university" ]
  }
  }, {
  "_index" : "blog",
  "_type" : "ariticle",
  "_id" : "3",
  "_score" : 0.19245009,
  "_source" : {
  "title" : "init Elasticsearch for university!",
  "content" : "Version 3.0!",
  "tags" : [ "initialize", "elasticsearch" ]
  }
  } ]
  }
}12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849
```

### 3、更新文档（类似mysql update操作）

http://localhost:9200/blog/ariticle/1/_update/ POST
{“script”:”ctx._source.content = \”new version 2.0 20160714\”“}

更新后结果显示：
{

- “_index”: “blog”,
- “_type”: “ariticle”,
- “_id”: “1”,
- “_version”: 2,
- “_shards”: {
  - ”total”: 2,
  - “successful”: 1,
  - “failed”: 0
- }

}

查询&验证更新后结果：（对比可知，版本号已经更新完毕）
http://localhost:9200/blog/ariticle/1/

```
{

- "_index": "blog",
- "_type": "ariticle",
- "_id": "1",
- "_version": 2,
- "found": true,
- "_source": {
    - "title": "New version of Elasticsearch released!",
    - "content": "new version 2.0 20160714",
    - "tags": [
        - "announce"
        - ,
        - "elasticsearch"
        - ,
        - "release"
    - ]
- }

}

注意更新文档需要在elasticsearch_win\config\elasticsearch.yml下新增以下内容：
123456789101112131415161718192021222324
script.groovy.sandbox.enabled: true
script.engine.groovy.inline.search: on
script.engine.groovy.inline.update: on
script.inline: on
script.indexed: on
script.engine.groovy.inline.aggs: on
index.mapper.dynamic: true
```



#### PUT和POST不同点

* PUT需要带 id才能更新，POST则不需要

* _update更新时需要加上“doc”，POST会对比原来数据，如果和原来一样就什么都不修改；若不加 `“ _update”和“doc”，`则会修改版本号

* ```
  http://192.168.200.128:9200/customer/external/1/_update
  {
  	"doc": {
  			"name":"2"
  	}	
  }
  // 结果：
  {
      "_index": "customer",
      "_type": "external",
      "_id": "1",
      "_version": 3,  // 乐观锁版本号
      "_seq_no": 7,  // 乐观锁字段，POST相同数据，该字段也不会修改
      "_primary_term": 1,
      "found": true,
      "_source": {
          "name": "2"
      }
  }
  ```

  

#### 带乐观锁修改

```
http://192.168.200.128:9200/customer/external/1?if_seq_no=0&if_primary_term=1
```



### 4、删除文档（类似mysql delete操作）

http://localhost:9200/blog/ariticle/8/  返回结果

```
{
- "found": true,
- "_index": "blog",
- "_type": "ariticle",
- "_id": "8",
- "_version": 2,
- "_shards": {
    - "total": 2,
    - "successful": 1,
    - "failed": 0
- }
}1234567891011121314
```

![这里写图片描述](../../../../Software/Typora/Picture/20160717132434415)

### API

```
// _bulk批量新增更新POST
http://192.168.200.128:9200/customer/external/_bulk   
{"index":{"_id":"1"}}
{"name":"John Doe"}
{"index":{"_id":"2"}}
{"name":"Jane Doe"}

```

# Query DSL

ES使用灵活的，容易表达的Query DSL，通过JSON接口暴露了Lucene大部分的功能，这也是让你在产品中使用的原因。他能使你的查询更灵活，更精确，更容易阅读，更容易调试。

使用query DSL，要在query参数中传递消息体：

```js
GET /_search
{
    "query": YOUR_QUERY_HERE
}
```

### 基本检索

```js
GET bank/_search  // bank: 索引名
{
    "query":{
        "match_all":{},
         // 全文检索字段用match，其他非text字段匹配用term
    	"match": {
            "address": "kings" // 倒排索引，模糊查询、精确匹配
            "address.keyword": "122 Madison Street"  // 短语查询，只匹配该完整的短语
        },
        "match_phrase": {
            "address": "mill lane"  // 也是短语查询，只需要全文包含该字段，与.keyword相比，keyword必须值包含keyword的短语
        }，
        “multi_match”: {
            "query": "mail",
            "fields": ["adddress", "city"]   // 多条件匹配，最少只需fields中一项包含“mail”
        }，
        "term": {   // 精确检索专用，全文检索用match或filter等
            "age": "20"  
        }
        
        "bool": {
            "must": [  // 必须满足的条件
                {"match": {
                    "gender": "F"
            	}},
                {
                    "range": { // 范围查询，18-30
                        "age": {
                            "gte": 18,
                            "lte": 30
                        }
                    }
                }
            ], 
            "must_not": [  // 必须不满足的条件
                {"match": {
                    "age": "15"  
            	}},
            ],
            "should": [  // 应该满足的条件
                {"match": {
                    "age": "15"  
            	}},
            ],
            "filter": {  // filter能做到以上所有事情，但没有相关性得分，即检索命中得分
                "range": {
                    "age": {
                        "gte": 18,
                        "lte": 30
                    }
                }
            }
        }
        
            
    },
       
    "sort": [
        {
            "balance": {
                "order": "desc"
            }
        }
    ],
     // 分页；从第1条记录拿5条记录    
    "from": 1,  
    "size": 5,
    ”source“: ["balance", "firstname"]  // 只查询某些字段
     
}
```

 ### aggregation聚合检索

```js
GET bank/_search  // bank: 索引名
{
    
	"aggs": {
        "ageAgg": {
            "terms": {
                "field": "age", 
                "size": 10 // 以age字段聚合，检索出10个可能结果
            }
        },
        "ageAvg": {   // 查询age字段平均值
            "avg": {
                "field": "age"
            }
        },
            
       // 聚合查询：查询满足terms条件，结果要求查age字段的平均值
        "ageAgg": {
            "terms": {
                "field": "age",
                "size": 10   // 以age字段聚合，检索出10个可能结果
            },
            "ages": {
                "ageAvg": {
                    "avg": {
                        "field": "balance"
                    }
                }
            }
        },
    }

}
```







### 结构化查询条款（structure of a query clause）

一个典型的结构化查询条款有如下形式：

```js
{
    QUERY_NAME:{
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}
```

或者，引用另外一个特定的field：

```js
{
    QUERY_NAME:{
        FIELD_NAME:{
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
```

例如，你能使用match查询找到tweets中tweet field中有”elasticsearch“的： 

```js
{
    "match":{
        "tweet":"elasticsearch"
    }
}
```

全部的搜索请求如下：

```js
GET /_search
{
    "query":{
        "match":{
            "tweet":"elasticsearch"
        }
    }
}
```

 

### 联合多条目查询（combining multiple clauses）

查询条目可以时简单的一个，也可以是相互联联合创建复杂的查询，条目可以如下组合：

*1:leaf clauses*（比如match）被用来与一个或多个field比较quert string

2:*compound* clauses用来和其他的查询想集合，例如一个bool查询允许你联合must匹配，must_not匹配，或者should匹配：

```js
{
    "bool":{
        "must":     {"match":{"tweet":"elasticsearch"}},
        "must_not":{"match":{"name":  "mary"}},
        "should":   {"match":{"tweet":"full text"}}
    }
}
```

很重要的一点就是compound clause即能联合其他的查询条目，也能包括其他的compound clause。也就是说compound clause能相互嵌套，表达更复杂的逻辑。

例如，下面的查询emails包括”business opportunity“并且要么starred是true，要么folder是inbox并且spam是true：

```js
{
    "bool":{
        "must":{"match":      {"email":"business opportunity"}},
        "should":[
             {"match":         {"starred":true}},
             {"bool":{
                   "must":      {"folder":"inbox"}},
                   "must_not":  {"spam":true}}
             }}
        ],
        "minimum_should_match":1
    }
}
```

### es8去掉type概念原因

7.0之后type可选，8.0之后去除

两个不同type下两个field，在同一个索引下会被认为是同一个field，你必须在不同的type中定义相同的field映射，否则不同type中相同字段名称会在处理中出现冲突情况，岛主Lucene处理效率下降。为了提高ES处理数据的效率



### 数据迁移

```js
POST _reindex
{
	"source": {
        "index": "bank",
        "type": "account"
    },
    "dest": {
        "index": "newbank"
    }
}
```





# ES原理

## 分词：

```
POST _analyze
{
	"analyzer": "standard",
	"text": "aaa bbb ccc"
}
```





### 倒排索引

- 反向索引又叫倒排索引，是根据文章内容中的关键字建立索引。
- 搜索引擎原理就是建立反向索引。
- Elasticsearch 在 Lucene 的基础上进行封装，实现了分布式搜索引擎。
- Elasticsearch 中的索引、类型和文档的概念比较重要，类似于 MySQL 中的数据库、表和行。
- Elasticsearch 也是 Master-slave 架构，也实现了数据的分片和备份。
- Elasticsearch 一个典型应用就是 ELK 日志分析系统。

# Kibana可视化分析

### 基本使用

GET/customer/external/_bulk

```
{"delete":{"_index":"website", "_type":"blog", "_id":"123"}}
```

POST /customer/external/_bulk

```
{"create":{"_index":"website", "_type":"blog", "_id":"123"}}   
{"title": "xxxxxx"}
{"update":{"_index":"website", "_type":"blog", "_id":"123"}}   
{"doc": "xxxxxx"}
```

PUT/customer/external/_bulk

```
{"index":{"_index":"website", "_type":"blog", "_id":"123"}}
{"title": "xxxxxx"}
```

DELETE /customer/external/_bulk   

```
{"delete":{"_index":"website", "_type":"blog", "_id":"123"}}
```



## 5.1、在索引blog上查询包含”university”字段的信息。

![这里写图片描述](../../../../Software/Typora/Picture/20160717132502244)

## 5.2、Kibana多维度分析

![这里写图片描述](../../../../Software/Typora/Picture/20160717132453634)

# Rest整合



```
	<dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>${elasticsearch.version}</version>
        </dependency>
```

```java
@Configuration
public class GulimalllElasticSearchConfig {
    @Bean
    public RestHighLevelClient esRestClient() {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("192.168.200.128", 9200, "http")));
        return client;
    }
}
```



```java
    @Test
    public void indexData() throws IOException {
        // 构造索引
        IndexRequest indexRequest = new IndexRequest("users");
        // 设置id
        indexRequest.id("1");
        // 添加索引数据
//        indexRequest.source("userName", "zhangsan", "age", 18, "gender", "男");
        User user = new User();
        user.setUserName("zhangsan");
        user.setAge(19);
        user.setGender("男");
        String jsonString = JSON.toJSONString(user);
        indexRequest.source(jsonString, XContentType.JSON);
        // 执行保存操作
        IndexResponse index = client.index(indexRequest, GulimalllElasticSearchConfig.COMMON_OPTIONS);
        System.out.println(index);
    }
```

索引数据迁移

```
POST _reindex
{
  "source": {
    "index": "product"
  },
  "dest": {
    "index": "gulimall_product"
  }
}
```



### 复杂应用实例

```json
GET gulimall_product/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "skuTitle": "华为"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "catalogId": "225"
          }
        },
        {
          "terms": {
            "brandId": [
              "1"
            ]
          }
        },
        {
          "nested": {  // 嵌入方式，即条件为满足attrId=18且attrValue=‘4K’
            "path": "attrs",
            "query": {
              "bool": {
                "must": [
                  {
                    "term": {
                      "attrs.attrId": {
                        "value": "18"
                      }
                    }
                  },
                  {
                    "terms": {
                      "attrs.attrValue": [
                        "4K"
                      ]
                    }
                  }
                ]
              }
            }
          }
        },
        {
          "term": {
            "hasStock": {
              "value": "true"
            }
          }
        },
        {
          "range": {
            "skuPrice": {
              "gte": 0,
              "lte": 6000
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "skuPrice": {
        "order": "desc"
      }
    }
  ],
  "from": 0,
  "size": 1,
  "highlight": {
    "fields": {"skuTitle": {}}, 
    "pre_tags": "<b style='color:red'>",
    "post_tags": "</b>"
  }
  
}
```

#### 聚合

```
GET gulimall_product/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "brand_agg": {
      "terms": {
        "field": "brandId",
        "size": 10
      },
      "aggs": {
        "brand_name_agg": {
          "terms": {
            "field": "brandName",
            "size": 10
          }
        },
        "brand_img_agg": {
          "terms": {
            "field": "brandImg",
            "size": 10
          }
        }
      }
    },
    "catalog_agg": {
      "terms": {
        "field": "catalogId",
        "size": 10
      },
      "aggs": {
        "catalog_name_agg": {
          "terms": {
            "field": "catalogName",
            "size": 10
          }
        }
      }
    },
    "attr_agg": {
      "nested": {
        "path": "attrs"
      },
      "aggs": {
        "attr_id_agg": {
          "terms": {
            "field": "attrs.attrId",
            "size": 10
          },
          "aggs": {
            "aggr_name_agg": {
              "terms": {
                "field": "attrs.attrName",
                "size": 10
              }
            },
            "aggr_value_agg": {
              "terms": {
                "field": "attrs.attrValue",
                "size": 10
              }
            }
          }
        }
      }
    }
  }
}
```

### 电商完整的搜索展示

```
GET gulimall_product/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "skuTitle": "华为"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "catalogId": "225"
          }
        },
        {
          "terms": {
            "brandId": [
              "1"
            ]
          }
        },
        {
          "nested": {
            "path": "attrs",
            "query": {
              "bool": {
                "must": [
                  {
                    "term": {
                      "attrs.attrId": {
                        "value": "18"
                      }
                    }
                  },
                  {
                    "terms": {
                      "attrs.attrValue": [
                        "4K"
                      ]
                    }
                  }
                ]
              }
            }
          }
        },
        {
          "term": {
            "hasStock": {
              "value": "true"
            }
          }
        },
        {
          "range": {
            "skuPrice": {
              "gte": 0,
              "lte": 6000
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "skuPrice": {
        "order": "desc"
      }
    }
  ],
  "from": 0,
  "size": 1,
  "highlight": {
    "fields": {
      "skuTitle": {}
    },
    "pre_tags": "<b style='color:red'>",
    "post_tags": "</b>"
  },
  "aggs": {
    "brand_agg": {
      "terms": {
        "field": "brandId",
        "size": 10
      },
      "aggs": {
        "brand_name_agg": {
          "terms": {
            "field": "brandName",
            "size": 10
          }
        },
        "brand_img_agg": {
          "terms": {
            "field": "brandImg",
            "size": 10
          }
        }
      }
    },
    "catalog_agg": {
      "terms": {
        "field": "catalogId",
        "size": 10
      },
      "aggs": {
        "catalog_name_agg": {
          "terms": {
            "field": "catalogName",
            "size": 10
          }
        }
      }
    },
    "attr_agg": {
      "nested": {
        "path": "attrs"
      },
      "aggs": {
        "attr_id_agg": {
          "terms": {
            "field": "attrs.attrId",
            "size": 10
          },
          "aggs": {
            "aggr_name_agg": {
              "terms": {
                "field": "attrs.attrName",
                "size": 10
              }
            },
            "aggr_value_agg": {
              "terms": {
                "field": "attrs.attrValue",
                "size": 10
              }
            }
          }
        }
      }
    }
  }
}
```

#### 使用es进行检索的步骤：

```java
    @Override
    public SearchResult search(SearchParam param) {
        // 动态构建出查询需要的dsl语句
        SearchResult result = null;
        // 准备检索请求
        SearchRequest searchRequest = buildSearchRequest();
        try {
            // 执行检索请求
            SearchResponse response = client.search(searchRequest, GulimalllElasticSearchConfig.COMMON_OPTIONS);
            result = buildSearchResult(response);

            // 分析响应数据，封装成需要的格式
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }
```













