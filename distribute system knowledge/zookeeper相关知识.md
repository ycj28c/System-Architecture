Zookeeper学习笔记

https://zhuanlan.zhihu.com/p/210999265

## 为什么要用Zookeeper  
Zookeeper主要解决了分布式系统的强一致问题，在单机的并发可以用多线程框架来解决，但是在集群情况下是无法处理的，因为每天主机都是独立的，所以需要有zookeeper，所以zookeeper就是这样一个分布式协调系统。

不过分布式强一致事务会大大降低系统效率，在CAP里算是牺牲了Available来保证consistent。

zk是个文件系统，不过存储很小，主要是用来做协调的。就是通过文件系统来做管理。

## 具体功用(还是缺乏实战认识)
1、命名服务：命名服务是指通过指定的名字来获取资源或者服务的地址，利用zk创建一个全局的路径，即是唯一的路径，这个路径就可以作为一个名字，指向集群中的集群，提供的服务的地址，或者一个远程的对象等等。

2、配置管理（文件系统、通知机制）：程序分布式的部署在不同的机器上，将程序的配置信息放在zk的znode下，当有配置发生改变时，也就是znode发生变化时，可以通过改变zk中某个目录节点的内容，利用watcher通知给各个客户端，从而更改配置。  
一个分布式配置文件的例子：[基于zookeeper实现统一配置管理](https://blog.csdn.net/sinat_42483341/article/details/107224842)

3、集群管理：是否有机器退出和加入、选举master。对于机器的退出，所有机器约定在父目录下创建临时目录，对于新机器的加入，所有机器创建临时顺序编号目录节点。

4、分布式锁：分为两类，一个是保持独占：客户端需要的时候，就去通过createznode的方式实现，所有客户端都去创建/distribute_lock节点，用完就删除节点就行了。一个是控制时序，/distribute_lock已经预先存在，所有客户端在它下面创建临时顺序编号目录节点。主要流程是：客户端在获取分布式锁的时候在locker节点下创建临时顺序节点，释放锁的时候就删除，客户端首先调用createZnode放在在locker创建临时顺序节点，然后调用getChildren来获取locker下面的所有子节点，此时不用设置watch，客户端获取了所有子节点的path之后，反正最后要找到最小序号的那个节点，调用exist方法，同时对其注册事件监听器

5、队列管理：两种类型的队列，一种是同步队列，一个是按照FIFO方式进行入队和出队，第二种保证了队列消息的不会丢失，因为会在特定的目录下创建一个persistent_sequential节点，创建成功时watcher通知等待的队列，队列删除序列号最小的节点，此场景下，zk中的znode用于消息存储，znode存储的数据就是消息队列中的消息内容，sequential序列号就是消息的编号，按序列取出即可。

https://zhuanlan.zhihu.com/p/75161633

## Zookeeper原理
弱一致性：就是最终一致性，比如DNS，Gossip协议(Cassandra)  
强一致性：包括同步，Paxos，Raft，ZAB算法

数据不能存在单点上，分布式系统对fault tolorence一般就是state machine replication，所以这个强一致性就是要state machine replication的consensus(共识)算法。

强一致性要求：
1）顺序一致性
2）原子性
3）单一的系统映像
4）持久性

强一致性方式包括：
1）主从同步：一主一从，一主多从等，这种方式的可用性特别差。  
2）多数派：每次写和读都要保证N/2+1个节点成功，纯粹的多数派机制在并发情况下可能无法保证顺序，导致脏数据。  
3）使用类似Paxos的算法，下面详细讨论  

Basic Paxos：  
角色太多，有client，proposer，acceptor，learner等，不容易开发，流程也太复杂

Multi Paxos：  
减少了角色，不过还是较难理解，传统的数据库比如MySQL也可以通过paxos协议实现强一致集群。

Raft：  
简单版的Multi Paxos，划分3个子问题：
1）怎么选leader
2）怎么log Replicaion
3）怎么保证Safety
角色只包含了Leader，Follower，Candidate
主要Raft容易理解是因为有[动画演示](http://thesecretlivesofdata.com/raft/)，[另一个动画演示](https://raft.github.io/)

几个关键点：  
1）一个集群只有一个leader。  
2）集群有term的设置，表示leader是第几任的。  
2）leader之间和node时刻保持heart beat，只要超时node就会重新选择（quoram）。  
3）如果被分成了多个集群，也只会有1个集群大于n/2个节点，才能有效处理。  
4）leader宕机就是最先获得时钟的发动竞选，如果同时发动竞选，会失效等待下一次竞选，直到选出一个leader。  
5）如果宕掉的机器回复了，因为term的设置，会同步大term数据到小term节点去。  
6）数据只要超过N/2就算成功，返回client，剩余的节点会慢慢补上。  
7）节点要配置单数个，保证是多数派。  
8）写请求只能有leader提出，读可以读取任何node。 
9）当写请求完成，Leader再次向集群follower广播进行提交。    

注意还是有可能出现极端例子导致数据丢失的，不过一致性还是没有问题的。

ZAB：  
这是Zookeeper使用的算法，和Raft原理差不多。不过zab将leader周期称为epoch，raft称之为term。zab心跳方向是follower到leader，而raft心跳方向是从leader到follower。

## zookeeper监控原理
zeekpeer看着就是一个文件夹，和普通的文件夹一样。

1、zk类似于linux中的目录节点树方式的数据存储，即分层命名空间，zk并不是专门存储数据的，它的作用是主要是维护和监控存储数据的状态变化，通过监控这些数据状态的变化，从而可以达到基于数据的集群管理，zk中的杰点的数据上限时1M。  
2、zk中的wathc机制：client端会对某个znode建立一个watcher事件，当该znode发生变化时，这些client会收到zk的通知，然后client可以根据znode变化来做出业务上的改变等。

## 实践操作
Zookeeper：  
1）需要配置好文件系统path（每个节点不同的data文件夹）  
2）需要设置文件端口和心跳端口  
3）在每个server上需要配置好所有n个节点  
4）在每个文件夹里还需要设置myid标识  
5）启动命令例子  
服务端
```
## 启动每个zookeeper node
bin/zkServer.sh start zoo1.cfg
bin/zkServer.sh start zoo2.cfg
bin/zkServer.sh start zoo3.cfg

## 查看状态，是leader还是follower
bin/zkServer.sh status zoo1.cfg
```

客户端
```
## 客户端连接
bin/zkCli.sh -server 127.0.0.1:2181

## 进入zookeeper命令行后，简单的命令
## 显示所有现有文件夹
ls /
## 创建test文件夹
create /test 111
create /test/test11 sss
## 获取值，会获得sss
get /test/test11
```

Etcd：  
服务端：略  
客户端：
```
## 创建test2文件夹，值为aa
/usr/local/bin/etcdctl put test2 "aa"
## 从test2获取value，得到aa
/usr/local/bin/etcdctl get test2
```

## 类似产品
etcd，google强推的和k8s相关

只知道在性能测试上etcd已经超过了ZK，最主要的应该是一致性算法的优势，zk基于从paxos改进而来的zab，而etcd基于和paxos十分不同的raft

https://zhuanlan.zhihu.com/p/96690890

