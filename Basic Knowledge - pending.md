Table of Contents
=================
  * [CAP Theorem](#cap-theorem)
    * [Idea](#idea)
    
# CAP Theorem  
https://www.cnblogs.com/duanxz/p/5229352.html
http://www.ruanyifeng.com/blog/2018/07/cap.html
總的來説就是
Consistency
Availability
Partition tolerance
無法同時滿足

consistent hash 
virtual node
这一篇的介绍比较好
https://blog.csdn.net/lihao21/article/details/54193868

要解决的就是以下集群问题
1、平衡性(Balance)：平衡性是指哈希的结果能够尽可能分布到所有的缓冲中去，这样可以使得所有的缓冲空间都得到利用。很多哈希算法都能够满足这一条件。
2、单调性(Monotonicity)：单调性是指如果已经有一些内容通过哈希分派到了相应的缓冲中，又有新的缓冲加入到系统中。哈希的结果应能够保证原有已分配的内容可以被映射到原有的或者新的缓冲中去，而不会被映射到旧的缓冲集合中的其他缓冲区。 
3、分散性(Spread)：在分布式环境中，终端有可能看不到所有的缓冲，而是只能看到其中的一部分。当终端希望通过哈希过程将内容映射到缓冲上时，由于不同终端所见的缓冲范围有可能不同，从而导致哈希的结果不一致，最终的结果是相同的内容被不同的终端映射到不同的缓冲区中。这种情况显然是应该避免的，因为它导致相同内容被存储到不同缓冲中去，降低了系统存储的效率。分散性的定义就是上述情况发生的严重程度。好的哈希算法应能够尽量避免不一致的情况发生，也就是尽量降低分散性。 
4、负载(Load)：负载问题实际上是从另一个角度看待分散性问题。既然不同的终端可能将相同的内容映射到不同的缓冲区中，那么对于一个特定的缓冲区而言，也可能被不同的用户映射为不同 的内容。与分散性一样，这种情况也是应当避免的，因此好的哈希算法应能够尽量降低缓冲的负荷。



某些环节挂掉了，怎么处理。 
无非就是1. 要么replica， master slave, active-passive 或者 2.周期存snapshot 在磁盘上，然后存action log... 挂了可以重新恢复。。。


Service Mesh ？

Google MapReduce 加上 Google File System 这两篇论文可谓是大数据时代的开山之作，与 Google BigTable 并称 Google 的三架马车
GFS: google file system
這裏有詳細描寫，但是比較繁瑣了，較常用的是hadoop家族的HDFS  

Monolith VS Microservice  
Microservice: 易于scale，易于大项目分工，易于同步开发，不易设计，需要更多时间develop  
Monolith: 运行较快，难于新手入门，需要频繁deploy，一宕机就全宕



Long-Polling vs WebSockets vs Server-Sent Events
Long polling or short polling(client pull)
WebSocket(server push)
Server-Sent Events(server push)
https://blog.csdn.net/bell10027/article/details/89199762
https://mr-dai.github.io/gfs/

rating algorithm system
Glicko rating system: https://en.wikipedia.org/wiki/Glicko_rating_system

Firebase
Firebase 是一個同時支援 Android、iOS 及網頁的 app 雲端開發平台  
提供了以下功能，包含所有的后端，开发者只管写应用即可。  
实时数据库（Realtime database），用户认证（Authentication），自定义API（Cloud function），消息推送（Cloud messaging），静态网页（Hosting），云存储（Cloud storage），此外还提供了强大的用户分析，测试等功能，容易学开发快。

Monolith VS Microservice  
Monolith: 运行速度快，代码易于重用，适合小团队开发，需要频繁deploy，难于新人上手，一宕机全down  
Microservice: 易于scale，易于大项目分工，可以同步开发，模块多难于设计，需要更多develop时间，  
gc 算法
hashmap ，concurrentHashMap（put过程，怎么解决冲突的，sizeCtl 等关键成员变量的作用）
volatile
spring IOC 过程
如何自己设计IOC框架
设计一个高并发系统
cassandra和dynamodb的区别
JWT
IO多路复用
协程（结合网络编程说协程的优势）
乐观锁 悲观锁
如何用redis设计分布式悲观锁
gc算法，怎么判断对象可被回收
对CAS的理解，java中的CAS，如何不用unsafe实现CAS
假设公司有上万人，一则通知要快速的推送到每个人，如何通过kafka来实现
memtable怎么实现的(跳表)
给了链接，做了道题，双线程交替打印奇偶
spring @Autowired (@Resource, 类似） 怎么做的
uuid怎么做的，了解哪些UUID生成策略(雪花)
ACID
docker
浏览器输入url发生了什么（经典问题）
promise
