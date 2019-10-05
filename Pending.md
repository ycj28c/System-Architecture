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
https://mr-dai.github.io/gfs/
