//TODO

## 基础知识

NoSQL使用cluster作为集群，不过它们的集群方式并不同。

比如是cassandra是Master，而mongo database是文本数据库

sync replication
同步复制，这个可以保证强一致，不过follower多的情况下，延迟太大，一般很少使用
async replication
异步复制，这个可能造成读不一致，但是写入效率高
semi sync replication
半同步，一般采用quorum的机制，即当写入的节点个数满足指定条件，即算写入成功，然后通过并发请求多个node来满足读取的一致性


HDFS的运行依赖于NameNode，如果NameNode挂了，那么整个HDFS就用不了了，因此就存在单点故障(single point of failure)；其次，如果需要升级或者维护停止NameNode，整个HDFS也用不了。为了解决这个问题，采用了QJM机制(Quorum Journal Manager)实现HDFS的HA（High Availability）。
Quorum机制。每次写入JournalNode的机器数目达到大多数(W)时，就认为本次写操作成功了。

Quorum是一种简单有效的管理副本的机制，我们先建立如下定以（[2]中也有描述）：

N = 存储数据副本的节点的数量
W = 更新成功所需的副本更新成功的数量
R = 一次数据对象读取要访问的副本的数量

更新时，只有至少W副本更新成功时才算更新操作成功；读取时，至少要读取R个副本的数据。这样当W + R > N时，对于同一个数据对象，更新集合与读取集合一定有重叠，保证了读取的数据中一定有最近更新的值。

Quorum机制：W + R > N 保证一致性，在任意replica上写，用vector lock记录version，读数据时进行reconsiliation；N越大越不怕data center failure，W越小越可写，R越小越可读，只要W+R>N则为强一致。

## Reference
[聊聊replication的方式](https://www.jianshu.com/p/0abdb2fbe48c)  
[DDIA 5. 复制](https://github.com/Vonng/ddia/blob/master/ch5.md)  