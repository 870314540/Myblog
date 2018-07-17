ES基础之一


###简介

ES：Elasticsearch（elastic + search） 

ElasticSearch是一个基于Lucene的开源搜索引擎。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口.

Elasticsearch是面向文档(document oriented)的。在Elasticsearch中，你可以对文档（而非成行成列的数据）进行索引、搜索、排序、过滤。


Elastic Search是一个实时的高可用的分布式全文搜索和分析引擎，它可以快速地对PB级的数据进行搜索、分析、快速返回结果；它是建立在全文搜索引擎Lucene基础上的搜索引擎，提供多种语言的API，使用简单，可以方便快捷地进行扩容以满足不断增长的业务数据需求。
         
分片 复制  冗余 


         
与传统数据库比较：

	Relational DB -> Databases -> Tables -> Rows      -> Columns
	Elasticsearch -> Indices   -> Types  -> Documents -> Fields
	
	
###基本概念

- 接近实时搜索 NRT 

- 集群  节点

群集是一个或多个节点（服务器）的集合，它们共同保存您的整个数据，并提供跨所有节点的联合索引和搜索功能。群集由唯一名称标识

节点是属于集群一部分的单个服务器。它存储数据并参与群集索引和搜索功能。

一个运行中的 Elasticsearch 实例称为一个 节点，

集群是由一个或者多个拥有相同 cluster.name 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

- 索引 index

索引就像关系数据库中的“数据库”。它有一个定义多种类型的映射。索引是逻辑名称空间，映射到一个或多个主分片，并且可以有零个或多个副本分片。


- 类型 type

类型是索引的逻辑类别/分区，

- 分片和复制 

主分片 复制分片

默认情况下，Elasticsearch中的每个索引被分片`5个主分片和1个复制`，这意味着，如果你的集群中至少有两个节点，你的索引将会有5个主分片和另外5个复制分片（1个完全拷贝），这样的话每个索引总共就有10个分片。

<b>分片很重要，主要有两方面的原因： 
1）允许你水平分割/扩展你的内容容量。 
2）允许你在分片（潜在地，位于多个节点上）之上进行分布式的、并行的操作，进而提高性能/吞吐量。</b>

一个分片可以是 主 分片或者 副本 分片。 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。


###集群内容

- 集群健康

status 字段中展示为 green 、 yellow（主分片全部正常，副本分片不正常） 或者 red

`curl XGET Http://localhost:9200/_cluster/health`

put /index/_settings 动态调整副本分片数目


###输入和输出

在 Elasticsearch 中 文档是指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID。

元数据元素： _index,_type,_id

`PUT /{index}/{type}/{id}`
`POST /{index}/{type}  缺省id,系统自动帮助创建`

> 1.索引名:名字必须小写，不能以下划线开头，不能包含逗号.
> 2._type :可以是大写或者小写，但是不能以下划线或者句号开头,不应该包含逗号,并且长度限制为256个字符.
> 3.自动生成的 ID 是 URL-safe、 基于 Base64 编码且长度为20个字符的 GUID 字符串


获取部分文档url：
`/website/blog/123?_source=title,text`

只需source里字段
`/website/blog/123/_source`


在 Elasticsearch 中文档是 不可改变 的，不能修改它们。 相反，如果想要更新现有的文档，需要 重建索引 或者进行替换

在内部，Elasticsearch 已将旧文档标记为已删除，并增加一个全新的文档。 尽管你不能再对旧版本的文档进行访问，但它并不会立即消失。当继续索引更多的数据，Elasticsearch 会在后台清理这些已删除文档。
（标记后，内部后台清理）



### 分布式文档存储

路由一个文档到分片  `shard = hash(routing) % number_of_primary_shards`

主分片与副本分片交互

当发送请求的时候， 为了扩展负载，更好的做法是轮询集群中所有的节点。

一致性保证：必须有合适的分片数目

规定数目 ：  (primary + number_of_replicas) / 2  + 1

基于文档的复制：不会转发更新请求，转发完整的文档


###document的crud原理

1、客户端发送请求到任意一个node，成为coordinate node 
2、coordinate node对document进行路由，将请求转发到对应的node，此时会使用**round-robin随机轮询算法**，在primary shard以及其所有replica中随机选择一个，让读请求负载均衡 
3、接收请求的node返回document给coordinate node 
4、coordinate node返回document给客户端 
5、特殊情况：document如果还在建立索引过程中，可能只有primary shard有，任何一个replica shard都没有，此时可能会导致无法读取到document，但是document完成索引建立之后，primary shard和replica shard就都有了。





### mapping ：

类似于数据类型，组成有analyzer，filter

mapping的作用就是执行一系列的指令将输入的数据转成可搜索的索引项

###全文检索

全文检索，一般使用的是倒排索引的算法来实现的。 用到了Lucene









###倒排索引

倒排索引(Inverted Index)：倒排索引是实现“单词-文档矩阵”的一种具体存储形式，通过倒排索引，可以根据单词快速获取包含这个单词的文档列表。倒排索引主要由两个部分组成：“单词词典”和“倒排文件”。



###主备机制

ES存在主备机制，shard 和 replia机制

```
（1）index包含多个shard
（2）每个shard都是一个最小工作单元，承载部分数据，lucene实例，完整的建立索引和处理请求的能力
（3）增减节点时，shard会自动在nodes中负载均衡
（4）primary shard和replica shard，每个document肯定只存在于某一个primary shard以及其对应的replica shard中，不可能存在于多个primary shard
（5）replica shard是primary shard的副本，负责容错，以及承担读请求负载
（6）primary shard的数量在创建索引的时候就固定了，replica shard的数量可以随时修改
（7）primary shard的默认数量是5，replica默认是1，默认有10个shard，5个primary shard，5个replica shard
（8）primary shard不能和自己的replica shard放在同一个节点上（否则节点宕机，primary shard和副本都丢失，起不到容错的作用），但是可以和其他primary shard的replica shard放在同一个节点上
```

###容错机制
Elasticsearch容错机制：master选举，replica容错，数据恢复。

一个例子 ：
a、原来node1是一个master，假如因为某种原因node1宕机了，node中是R0，R1和R2全部丢失。
b、进行master选举，自动选举另外一个node成为新的master，承担起master的责任来，例如现在全部选举了node2为master。
c、新的master，将丢失的primary shard的某个replica shard提升为primary  shard。此时集群的状态就会变为yellow，原因是虽然目前所有的primary  shard全部变成active，但是replice shard少了，也就并不是所有的replica  shard都是active。
d、重启故障的node1，新的master(node2)会将缺失的副本都copy一份到这个node1上面去，而且这个node1会使用之前已经有的shard数据，只是同步一下宕机之后发生的修改，此时，集群是status再次变回green。


###es并发控制

乐观锁：不加锁。一般是version对比后，进行操作；

悲观锁：常见于关系型数据库。在各种情况下都上锁，就只有一个线程可以操作这条数据，不同的场景下，有行级锁、表级锁、读锁和写锁。

悲观锁的优点：方便，对应用程序来说透明，缺点：并发能力低。 
乐观锁：并发能力高，都需要重新比对版本号，然后可能需要重新加载数据，再写，这个过程，可能需要重复好几次。

es后台都是多线程异步的，多个修改请求，是没有顺序的；

es内部的多线程异步并发是基于自己的verison版本号进行乐观锁控制的

es还提供了一个feature，就是说，你可以不用它提供的内部_version版本号来进行并发控制，可以基于你自己维护的一个版本号来进行并发控制。






1、问题

如何存储

如何分片

如何启动 相应


如何分词索引

分片故障 如何处理



---

5.*之后，把string字段设置为了过时字段，引入text，keyword字段

这两个字段都可以存储字符串使用，但建立索引和搜索的时候是不太一样的

keyword：存储数据时候，不会分词建立索引

text：存储数据时候，会自动分词，并生成索引（这是很智能的，但在有些字段里面是没用的，所以对于有些字段使用text则浪费了空间）。

--- 

JestClient 操作 es 





参考：
[es 参考1](https://mp.weixin.qq.com/s/XEYsgYOcI7Wv0PZq4Sf-Hw)
[es 参考2](https://blog.csdn.net/sdksdk0/article/details/78469190)