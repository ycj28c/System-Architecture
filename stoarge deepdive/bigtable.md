## 简介
BigTable是Google的一个论文，Google BigTable是一个分布式，结构化数据的存储系统，它用来存储海量数据。该系统用来满足“大数据量、高吞吐量、快速响应”等不同应用场景下的存储需求。

具体的实现比如HBase，是列式数据库，本质上是一个超级大的map（key value存储）。具体的就是使用sortedMap<String, sortedMap<String, String>>来存储每行数据，这样数据可以方便的添加新属性，也可以方便的按照列hash，实现分布式。

这类nosql的用法都一样，就是高吞吐，可以并行做一项操作（也就是map），然后合并结果（也就是reduce）。和多线程是一个道理，不过将垂直的机器power变成了水平的机器power，这样通过加机器就行实现吞吐量的提升。

一个场景就是存放历史网页，一个url一行数据，但是一个属性比如title可以存放多个历史版本。

Google官方的bigtable，[Cloud Bigtable](https://cloud.google.com/bigtable），其实也和Hbase一样，都是按照白皮书写的。另外，Hbase是一个产品，而Google Bigtable是直接提供服务了，就不用考虑服务器啥的了，直接用就行。

具体参考指南https://cloud.google.com/bigtable/docs/quickstarts

Bigtable将数据统统看成无意义的字节串，客户端需要将结构化和非结构化数据串行化再存入Bigtable。所以在youtube的设计提到用Bigtable存储缩略图，因为是列式的，所以缩略图存储位置接近。

## 和GFS的关系
因为都是分布式的，所以BigTable实际上也是存放在GFS上的。

|   | GFS  | BigTable  | 
|---|---|---|
| 本质  | 分布式文件系统  | 分布式NoSQL Database   | 
| 开发公司  | Google  | Google  | 
| 是否开源  | 否  | 否  | 
| 类似开源产品  | HDFS  | HBase  | 
| 基本操作  | 读 & 写文件  | 增删改查记录  | 
| 类似什么  | 家用电脑的文件系统  | 大数据版的Excel  | 

## 数据操作细节
单节点存放为多个文件Block，前面的Block都是有序排列好的数据，最后一个block则是在内存中。  
一个block中包含了bloom filter端（用来

增删改：  
不删旧数据，当有新数据，会放到最后一个block中，当内存存满（256MB），就排好序，存放到disk去。  
然后定期进行多路归并之前的block，删除旧数据。

查：  
先全遍历最后一个block O(N)，最后一个block是内存数据，采用跳表（skip table）实现。  
如果不存在则从后往前遍历block，因为越后面的block数据越新，这些disk中的block是通过sstable实现，因为都是有序，检索效率为O(logN)。此外包含了bloom filter，检索速度应该是更快一些的。

## 管理过程
bigtable和GFS一样，也是master slave架构。是通过consistent hashing进行水平shard的（会用zookeeper代替）。  

读：  
client在请求数据的时候请求master，比如请求数据key A，master包含了consistent hash map，知道这个key属于哪个slave，然后就进入了slave本地的查询过程。（先找最后一个跳表 -> 然后倒序找有序表bloomer filter -> 然后二分找数据)

写：  
client在进行写数据的时候，同样请求master，比如写入key A：3，请求master后告知在slave1，于是client练到了slave1上写入跳表，如果跳表慢了就输出到disk。  

如果解决节点宕机问题？答案就是存到分布式文件系统GFS上，那么就可以放心用了。

## 分布式锁
在读写同时发生会发生Race Condition资源竞争问题，这就需要分布式锁来解决了。  

Chubby和Zookeeper是比较常见的分布式锁。引入了zookeeper之后，master就不需要额外的consistent hashmap了，直接交给锁服务器来管理数据分布和锁。

具体过程就是：  
1.client要写入/读数据，先请求锁服务器  
2.锁服务器锁定这个数据的key，并返回slave server（tablet server）的serverId  
3.client就可以到该serverId写入/读数据  
4.写入成功/找到数据后返回结果给client，然后client会让lock服务器unlock这个key  

## 总结
是由client，master，tablet server（slave），distributed lock

1.client的作用就是请求read和write  
2.tablet server的作用就是维护key value的具体数据  
3.master的作用是shard文件，以及管理tablet server的health  
4.distributed lock用于update metadata（维护consistent hashing），还有锁定全局key的作用

通常master就一个，然后感觉分布式锁会是性能瓶颈呢？

## Reference
[HBase实操 | 如何使用HBase存储图片](https://yq.aliyun.com/articles/670092)
[理解BigTable](https://lianhaimiao.github.io/2018/03/19/%E7%90%86%E8%A7%A3BigTable/)
