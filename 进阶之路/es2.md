##es2


###mac 安装

1、安装elasticsearch

`brew install elasticsearch`

2、安装完，查看版本 `elasticsearch --version`

Version: 6.2.4, Build: ccec39f/2018-04-12T20:37:28.497551Z, JVM: 1.8.0_171

3、启动与停止

`
brew services start elasticsearch
brew services stop elasticsearch
`

4、启动后，访问http://localhost:9200

```
{
  "name" : "Aot7YdN",
  "cluster_name" : "elasticsearch_***",
  "cluster_uuid" : "olODMrg-TpiIxwx5taIUjQ",
  "version" : {
    "number" : "6.2.4",
    "build_hash" : "ccec39f",
    "build_date" : "2018-04-12T20:37:28.497551Z",
    "build_snapshot" : false,
    "lucene_version" : "7.2.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}

```
5、安装logstash

`brew install logstash`

6、logstash查看版本 `logstash --version`

logstash 6.2.4

7、logstash启动,进入logstash的安装目录，输入下面的命令启动

`bin/logstash -e 'input { stdin { } } output { stdout {} }'`

8、logstash启动成功 http://localhost:9600
`{"host":"*****-Pro.local","version":"6.2.4","http_address":"127.0.0.1:9600","id":"f21fc0ba-2bc4-4ef9-b70c-ef27f626bd41","name":"*****-Pro.local","build_date":"2018-04-12T22:29:17Z","build_sha":"a425a422e03087ac34ad6949f7c95ec6d27faf14","build_snapshot":false}
`
9、安装kibana
`brew install kibana`

10、查看版本号`kibana --version`

11、到kibana目录运行./kibana，启动后访问：http://localhost:5601/ , 
看到kibana界面后会提示”Configure an index pattern”。

12、安装chrome elasticSearch head 插件

chrome://extensions/   搜索安装即可



###1 rest api 



向Elasticsearch发出的请求的组成部分与其它普通的HTTP请求是一样的：

curl -X<VERB>'<PROTOCOL>://<HOST>/<PATH>?<QUERY_STRING>' -d'<BODY>'

• VERB HTTP方法：GET , POST , PUT , HEAD , DELETE

• PROTOCOL http或者https协议（只有在Elasticsearch前面有https代理的时候可用）

• HOST Elasticsearch集群中的任何一个节点的主机名，如果是在本地的节点，那么就叫localhost

• PORT Elasticsearch HTTP服务所在的端口，默认为9200

• QUERY_STRING 一些可选的查询请求参数，例如 ?pretty 参数将使请求返回更加美观易读的JSON数据

• BODY 一个JSON格式的请求主体（如果请求需要的话


---

curl -XGET 'http://localhost:9200/_count?pretty' -H 'Content-type':'application/json' -d '
{
    "query": {
        "match_all": {}
    }
}
'


```

 curl -XPUT "http://localhost:9200/movies/movie/1" -H 'Content-type':'application/json' -d'
{
    "title": "The Godfather",
   "director": "Francis Ford Coppola",
    "year": 1972
}'
     
{"_index":"movies","_type":"movie","_id":"1","_version":2,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":1,"_primary_term":1}

```
```
curl -XGET "http://localhost:9200/movies/movie/1"

{"_index":"movies","_type":"movie","_id":"1","_version":2,"found":true,"_source":
{
    "title": "The Godfather",
    "director": "Francis Ford Coppola",
    "year": 1972
}
}

```

curl -XDELETE "http://localhost:9200/movies/movie/1"



版本号(_version)可用于跟踪文档已编入索引的次数


###2 ElasticSearch具有和端点(_bulk)用于用单个请求索引多个文档
 
 _search端点：使用_search端点，可选择使用索引和类型。也就是说，按照以下模式向URL发出请求：<index>/<type>/_search
 
搜索请求正文和ElasticSearch查询DSL :ElasticSearch自己基于JSON的域特定语言，可以在其中表达查询和过滤器
 
 curl -XPOST "http://localhost:9200/_search" -H 'Content-type':'application/json' -d'
{
    "query": {
        "query_string": {
            "query": "kill"
        }
    }
}'
 
fields属性用来要搜索的字段数组：

 "query": {
        "filtered": {
            "query": {
                "query_string": {
                    "query": "drama"
                }
            },
            "filter": {
                "term": { "year": 1962 }
            }
        }
    }









 ---
 
### ik 用法

curl -XPUT http://localhost:9200/index 
 
 {"acknowledged":true,"shards_acknowledged":true,"index":"index"}
 
 
 curl -XPOST http://localhost:9200/index/fulltext/_mapping -H 'Content-Type:application/json' -d'
{
        "properties": {
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word"
            }
        }

}'

{"acknowledged":true}

```
curl -XPOST http://localhost:9200/index/fulltext/_search  -H 'Content-Type:application/json' -d'
{
    "query" : { "match" : { "content" : "中国" }},
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "content" : {}
        }
    }
}
'
```

 在这个文件里配置字典IKAnalyzer.cfg.xml
 
  
[ik示例](https://github.com/medcl/elasticsearch-analysis-ik) 
 
 
 
 
 ---
 
 
 
 
### 几个重点部分
结果聚合

索引的各种操作

集群的api 及操作
 
 ```
 curl -XGET http://localhost:9200/_cluster/health
 
{"cluster_name":"elasticsearch_cuiyt","status":"yellow","timed_out":false,"number_of_nodes":1,"number_of_data_nodes":1,"active_primary_shards":5,"active_shards":5,"relocating_shards":0,"initializing_shards":0,"unassigned_shards":5,"delayed_unassigned_shards":0,"number_of_pending_tasks":0,"number_of_in_flight_fetch":0,"task_max_waiting_in_queue_millis":0,"active_shards_percent_as_number":50.0}
 ```
 
  elasticsearch模块

 
 ---
 问题
 1、score是怎么算的 （有待深入）
 
 Elasticsearch 默认是按照文档与查询的相关度(匹配度)的得分倒序返回结果的. 得分 (_score) 就越大, 表示相关性越高.

 TF/IDF算法
 
 Term Frequency ：某单个关键词(term) 在某文档的某字段中出现的频率次数

 tf(q in d) = sqrt(termFreq)
 
 
 Inverse Document Frequency：某个关键词(term) 在索引(单个分片)之中出现的频次； 出现频次越高, 这个词的相关度越低
 
 idf = 1 + ln(maxDocs/(docFreq + 1))

Field-length Norm：字段长度, 这个字段长度越短, 那么字段里的每个词的相关度也就越大. 某个关键词(term) 在一个短的句子出现, 其得分比重比在一个长句子中出现要来的高.

具体计算公式是 norm = 1/sqrt(numFieldTerms)
 
 单关键字term搜索，可以`tf*idf*norm`,如果多个搜索关键词，还要有Vector Space Model
 
 
 2、hits中参数
 
 
 
 参考：
 [yiibai教程](https://www.yiibai.com/elasticsearch/elasticsearch-getting-start.html)
 [安装参考](https://blog.csdn.net/callmepls1/article/details/79441505)