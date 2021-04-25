Introduce
---------
现在的数据库种类很多，除了传统数据库，还有文档数据库，图形数据库，竪型数据库等等，本文讨论的是用Elastic Search作爲数据库。我们知道Elastic Search是很广汎应用的全文搜索，比较典型的例子就是Google的搜索bar（当然人家有自己的），同类的搜索引擎还有Solr之类。因爲ES也可以作爲一个数据库，其高性能并行处理（砸钱堆机器），高可用性（砸钱堆机器），免费使用的特性让他作爲一个数据库也充满亮点。

Replace Relation DB
-------------------
传统数据库适用与业务逻辑複杂，各种join，表于表之间联係紧密，预算紧张的情况。但是如果财大气粗，这些都可以用Elastic Search解决，而且ES的横向扩展性很好，空间不够，性能不佳的情况直接加入更多的Node和内存即可。这里介绍下具体的转换：

1. 数据的存放  
ES是基于lucene，大量的数据都是存放在内存中。ES会有很多的node，数据分佈应该通过一定的Hash算法，每个node均匀存放部分数值范围，load balance在每个node上，命中了直接返回，没命中则经过master来定位。是一种空间换时间的方式，每个shad存放所有需要的数据。

2. 数据分佈结构
比如有Organiztion和Person两个Shad，虽然说这个Organization A里头有Person A，这个Person A也包含Organization A的信息，但是这确实两个分开的Shad，不可能通过Join来联合查询。因爲複杂联合速度会很慢，所以会将需要的数据尽量存放在1个index中去，需要好好的设计数据结构。

3. 等价数据查询
ES基本上把关系型数据库的sql语言使用另外的算法再整了一边，比如Group By:  
SQL的Group By
```
SELECT model,COUNT(DISTINCT color) color_count FROM cars GROUP BY model HAVING color_count > 1 ORDER BY color_count desc LIMIT 2;
```
对应Elasticsearch就是Terms Aggregation，即分桶聚合（也就是mapreduce之类的概念）
```
GET cars/_search
{
  "size": 0,
  "aggs": {
    "models": {
      "terms": {
        "field": "model.keyword"
      },
      "aggs": {
        "color_count": {
          "cardinality": {
            "field": "color.keyword"
          }
        },
        "color_count_filter": {
          "bucket_selector": {
            "buckets_path": {
              "colorCount": "color_count"
            },
            "script": "params.colorCount>1"
          }
        },
        "color_count_sort": {
          "bucket_sort": {
            "sort": {
              "color_count": "desc"
            },
            "size": 2
          }
        }
      }
    }
  }
}
```
而COUNT(DISTINCT color) color_count需要使用一个指标类聚合Cardinality Bucket Filter实现having condition。ORDER BY color_count desc LIMIT 3是用Bucket Sort算法实现。所以，ES就是集群算法版本的数据库。 
 
关于关係型数据库和ES查询的转换可以参考[Elasticsearch如何实现SQL语句中 Group By 和 Limit 的功能](https://segmentfault.com/a/1190000014946753)

ES vs Relation DB
-----------------
__优点：__  
* 高并发。实测es单机分配10g内存单实例，写入能力1200qps，60g内存、12核CPU起3个实例预计可达到6000qps。 
* 横向Scale方面做的很出色，满足大数据下实时读写需求，无需分库（不存在库的概念）。数据之间无关系，这样就非常容易扩展。
* 速度快，ES直接返回数据，没有relational db那些步骤，基本就是IO的速度。

__缺点：__  
* es没有事务，而且是近实时。
* 吃硬件，几乎靠吃内存提高性能，成本也比传统数据库高。
* 传统数据的多表关联操作，在es中处理会非常麻烦。比如有4，5张表想要join下然后group by order by就不容易搞定。原因在于：传统数据库设计的初衷在于特定字段的关键词匹配查询；而es倒排索引的设计更擅长全文检索。
* ES的权限这块还不完善。
* 浪费空间，ES是基于Lucene开发的，它的许多局限从根本上都是由Lucene引入的。例如，为了提高性能，Lucene会将同一个term重复地index到各种不同的数据结构中，以支持不同目的的搜索，基于你选用的分析器，最终index数倍于原本的数据大小是有可能的。内存方面，ES的排序和聚合（Aggregation）操作会把几乎所有相关不相关的文档都加载到内存中，一个Query就可以很神奇地吃光所有内存
* 最逆天的是，mapping不能改，需求变化时候用es你会想死，数据量多你能想像一下几亿的数据慢慢scan然后全部重建的痛苦吗。
* 因为使用桶聚合，不知道桶有多大，数据太多就爆了，导致OOM，全靠花钱堆。但是因为快，所以oom不一定发生。  

此外，ES团队不推荐完全采用ES作为主要存储，缺乏访问控制还有一些数据丢失和污染的问题，建议还是采用专门的 DB存储方案，然后用ES来做serving。可以用ES + HDFS主要用来做查询等，但速度没达到毫秒级，几十亿数据查询3秒左右，ES官方说是毫秒级 但没说数据量级别。elasticsearch是分布式的存储，比如你有5个分片，你的数据是分散在5个分片实例上的，但凡是分布式数据存储，使用join等关联性操作就会复杂化，因为你无法确保需要关联的数据在同一个分片上，因此elasticsearch引入了parent_children数据关系类型(可以做到join等操作)；数据库分库分表也是需要考虑数据存储路由规则的,另外索引机制，使得不同类型数据在同一index下，会造成不必要的搜索压力。

ES vs Other NoSql
-----------------

__关于nosql种类：__    
* 键值(Key-Value)存储数据库： redis，voldemort
* 列存储数据库： Cassandra，Hbase，Riak。 这部分数据库通常是用来应对分布式存储的海量数据（大表）。键仍然存在，但是它们的特点是指向了多个列。这些列是由列家族来安排的。
* 文档型数据库： couchDB，MongoDb。 文档型数据库可以看作是键值数据库的升级版，允许之间嵌套键值。而且文档型数据库比键值数据库的查询效率更高。
* 图形数据库： Neo4J, InfoGrid, orientDb。 connection使用graph db来做有优势。

__优点(和MongoDB比较）：__    
* 免费使用,当初mongoDB太贵，所以用ES，目前还在用orientDb，neo4j（connection都是用graph db做的，有优势）
* 容错能力比mg强。比如1主多从，主片挂了从片会自动顶上（硬件上）
* 支持较复杂的条件查询，group by、排序都不是问题（mongo不行）
* ES还多出个全文索引功能，当然mongo3也有，但和es比就是个玩笑。es现在更多的也被归入nosql一族了而非ir系统

__缺点(和MongoDB比较）：__  
* MongoDB比较简单，轻量级，易用


Addition Resource
-----------------
关于ElasticSearch-Hadoop和家族，简单来说，Hadoop还是Hadoop，Elasticsearch还是Elasticsearch，而Elasticsearch-Hadoop在中间用来连接这两个系统，大量的原始数据可以存放在Hadoop里面，通过Elasticsearch—Hadoop可以调用Hadoop的Map-Reduce任务来创建elasticsearch的索引，数据进入elasticsearch之后，就可以使用elasticsearch的搜索特性来进行更加高级的分析，比如基于Kibana来作快速分析。

Elasticsearch支持多种类型的gateway，有本地文件系统（默认），分布式文件系统，Hadoop的HDFS和amazon的s3云存储服务。HDFS就是分布式文件存储系统，在读写一个文件时，当我们从 Name nodes 得知应该向哪些 Data nodes 读写之后，我们就直接和Data node 打交道，不再通过Name nodes。

当然这些年由于云比如AWS，Azure的兴起，数据库也走云服务了，如果真的不差钱，完全可以直接用云服务，省去了基础设施的顾虑。

Reference
---------
[5分钟深入浅出 HDFS](https://zhuanlan.zhihu.com/p/20267586)  
[elasticsearch（lucene）可以代替NoSQL（mongodb）吗？](https://www.zhihu.com/question/25535889)  
[Elasticsearch--- mapping是什么](https://www.jianshu.com/p/7cf6af033823)  
[Elasticsearch如何实现SQL语句中 Group By 和 Limit 的功能](https://segmentfault.com/a/1190000014946753)  
[一文读懂非关系型数据库（NoSQL）](https://www.jianshu.com/p/2d2a951fe0df)  
