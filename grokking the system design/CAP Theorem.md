## CAP Theorem 笔记

P(Partition tolerance)分区容错性:  
个别节点down的情况下还能正常工作。  
大多数分布式系统都分布在多个子网络。每个子网络就叫做一个区（partition）。分区容错的意思是，区间通信可能失败。比如，一台服务器放在中国，另一台服务器放在美国，这就是两个区，它们之间可能无法通信。一般来说，分区容错无法避免(down个节点总会发生），因此可以认为 CAP 的 P 总是成立。CAP 定理告诉我们，剩下的 C 和 A 无法同时做到。

C(Consistency)一致性:   
表示同一时间能看到一样的数据。通过锁定数据实现。  

A(Availability)可用性:  
只要收到用户的请求，服务器就必须给出回应。通过replicate数据实现。  
用户可以选择向 G1 或 G2 发起读操作。不管是哪台服务器，只要收到请求，就必须告诉用户，到底是 v0 还是 v1，否则就不满足可用性。


不管什么数据库，最多满足其中两项

比如RDBMS传统数据库满足AC，但是不满足P  
Cassandra和CouchDB满足AP，但是不满足C  
BigTable，MongoDb和HBase满足CP，但是不满足A  

一致性和可用性，为什么不可能同时成立？答案很简单，因为可能通信失败（即出现分区容错）。如果保证 G2 的一致性，那么 G1 必须在写操作时，锁定 G2 的读操作和写操作。只有数据同步后，才能重新开放读写。锁定期间，G2 不能读写，没有可用性不。  

在什么场合，可用性高于一致性？举例来说，发布一张网页到 CDN，多个服务器有这张网页的副本。后来发现一个错误，需要更新网页，这时只能每个服务器都更新一遍。

We can only build a system that has any two of these three properties. Because, to be consistent, all nodes should see the same set of updates in the same order. But if the network suffers a partition, updates in one partition might not make it to the other partitions before a client reads from the out-of-date partition after having read from the up-to-date one. The only thing that can be done to cope with this possibility is to stop serving requests from the out-of-date partition, but then the service is no longer 100% available.这个意思也类似，想要一致就得锁其他的，那么被锁的时候就不可用了。

## Reference
[CAP 定理的含义](https://www.ruanyifeng.com/blog/2018/07/cap.html)