正常情况用Relation DB就行，通常是特定情况采用NoSQL。  

关键部分说3次：  
特定情况采用NoSQL!
特定情况采用NoSQL!
特定情况采用NoSQL!

### 使用NoSQL的场景：

1.明显的键值结构  
比如cache场景，使用Redis和MemcacheDB之类key-value存储，性能高（比如Redis超过10W的TPS）

2.明显的graph结构  
比如社交网的Relation结构业务，那么用Neo4J是很直观的  

3.海量数据分布式的场景  
数据分析，使用大数据的标配，Hbase，BigTable等，不怕大表，查询快  

4.业务简单关联少schemaless，但是数据结构变化迅速  
比如创业初期，大量schemaless场景。使用mongoDB，CounchDB之类文档数据库，什么attribute变化直接加到那个key上，而关系型数据库一加是一整个column。但是复杂查询join不行。  

5.性能要求较高/高并发，对consistent要求低的场景  
比如推荐/排行榜/粉丝关注之类的业务，NoSQL扩展方便，丢数据也没事。还有热点数据，可以充分利用NoSQL的性能。    

6.全文搜索引擎   
比如类似google搜索引擎的功能，用Elastic Search，Solr啊，技术核心是“倒排索引”（inverted index），就是通过关键词index文档。也是随便扩展的，缺点同样是没有事务，复杂查询慢。  

### 一定不能用NoSQL的场景：

1.要求事务，ACID的情况   
比如银行/支付/购物车（如果购物车显示允许最终一致可以NoSQL)，用户metadata等，不能丢，必须SQL。NoSQL只能evential consistent。

2.需要复杂关联查询   
比如一个CEO的compensation，要统计stock，option，salary，bonus等等收入，来源表很多，就不适合了。  
不过这一点可以通过denormalize数据库结构解决，而且在QPS高的情况下，MySQL等关系型数据库也要denormalize来进行shard，和NoSQL一样用法。

3.需要立刻返回处理结果的情况  
NoSQL很多是Eventual consistency，就是说插入的数据不是马上能看到，这样不适合实时回馈的场景比如订票。


## Reference
[NoSQL 还是 SQL ？这一篇讲清楚](https://juejin.cn/post/6844903654869172232)  
[NoSQL_系统设计笔记12](http://www.ayqy.net/blog/nosql/)
