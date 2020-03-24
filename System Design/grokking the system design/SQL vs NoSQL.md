## SQL vs. NoSQL 笔记

### 基本概念

SQL:    
使用row和column来存储数据，  
常见有MySQL, Oracle, MS SQL Server, SQLite, Postgres, and MariaDB  

NoSQL:  
主要分为四种：  
* Key-Value Stores -- 就是key-value pair，常见的有Redis, Voldemort, and Dynamo  
* Document Databases -- 数据以document形式存放，每个文档包含整个entity，常见的有CouchDB and MongoDB  
* Wide-Column Databases（宽列存储） -- 说白了普通行式数据通用SortedMap<String, String>存放，而列式数据库就是用类似Sorted<String, Sorted<String, String>>来存放。常见的有Cassandra和HBase  
* Graph Databases -- 用图结构存储数据，有vertax,node等。应用场景主要是递归类型的，比如朋友的朋友的朋友的朋友，关系越多，相对普通关系型数据库越快。在商品数据库中，我们查询某个客户买了哪些商品通常效率比较高，但是我们要查询”那些客户买了这个商品”就会比较慢，因为需要全表遍历。常见的有Neo4J和InfiniteGraph。


### SQL/NoSQL高级别区别

Schema:  
SQL必须定义好column，比如存放car，那么column要先定义好是Color，Make，Modal。  
NoSQL的是动态的，可以随时插入，而且每个car的属性不用都完全相同。  

Querying:  
SQL使用SQL(structured query language)来查询，非常强大。  
NoSQL的query主要针对collection，不同类型数据库语法不同，查询力度较弱。  

Scalability:  
SQL通常只能垂直扩展，比如增加horsepower(马力)，比如更多CPU，高性能Memory等。有集群的MySQL之类的解决方案，但是比较有挑战和耗时。  
NoSQL可以非常方便的进行水平扩展，我们可以通过增加更多的服务器来处理更多的流量(注意是更多，未必更快)。  

Reliability(可靠性) or ACID compliancy:  
SQL数据库基于ACID compliancy(Atomicity原子性, Consistency一致性, Isolation隔离性, Durability持久性)。
NoSQL数据库会牺牲部分ACID特性来保证性能和扩张性。对于分布式事务的特性BASE，则是反这个标准的，即基本可用（Basically Availble）, 软状态/柔性事务（Soft-state）, 最终一致性（Eventual Consistency）

还有基础的CAP理论:CAP理论是Brewer教授提出的：一个分布式系统不能同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Tolerance of network Partition）。鱼和熊掌不可兼得。

### SQL VS NoSQL 选择哪个？
按情况而定。

选择SQL：   
1.在需要保证ACID的情况下，比如很多e-commerce和financial的应用，要保证数据准确。  
2.如果数据是结构化的并且不怎么变化，也是SQL更好。  

选择NoSQL:  
NoSQL可以保证没有性能瓶颈  
1.大容量数据总是有一点非结构化的数据，NoSQL数据库可以方便的增加和修改类型。比如document数据库，可以加入任意type  
2.可以使用便宜的cloud-based computing硬件设备，比如Cassandra就是设计为可以扩张到多个data center的情况  
3.特别适合快速开发迭代quick iteration of your system的情况，因为需要频繁修改  


## Reference
[图数据库：概览](https://zhuanlan.zhihu.com/p/64962725)
