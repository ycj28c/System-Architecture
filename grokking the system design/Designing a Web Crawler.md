## Designing an Web Crawler 笔记
不同的web crawler要求不一样，有可能P2P要用distribute hashing table的（没学过）。这是本文的主要设计，各个节点独立。

如果centralized architecture那就不一样，主要由中央服务器分配任务。技术上大约是用kafka存放url，然后消费者读取kafka做bfs。判重则直接使用redis实现。

### 1.What is Web Crawler?
就是爬虫，都知道

### 2.Requirements and Goals of the System
这里就是Non-function requirement：  
Scalability:因为数据量很大。  
Extensibility:可扩展性

### 3.Some Design Considerations
问问题尝试搞清楚ambiguous。

Function：  
1.爬虫只爬HTML，还是也爬image，video等等？本文只爬HTML  
2.爬什么协议？HTTP，FTP链接或者其他类型？本文只关注HTTP  
3.要爬多少pages？是不是有个URL database？本文假设1个月爬1B数据，因为链接pages还有链接，所以可能会有15B的数据  
4.某个RobotsExclusion的知识点，就是专门给爬虫准备的robot.txt

关键是要想到是否已知要爬的URL，还有是否爬URL的URL

### 4.Capacity Estimation and Constraints
做计算了，要算出QPS，才有概念多大规模系统设计

QPS：
15B / (4 weeks * 7 days * 86400 sec) ~= 6200 pages / sec

存储：
假设每个HTML page 100K，此外每个记录500Bytes（500个英文）的metadata  
15B * (100KB + 500) ~= 1.5 petabytes

注意使用70% capacity modal，1.5petabytes/0.7 ~= 2.14 petabytes

### 5.High Level design 
爬虫基本过程就是获取要爬的URL，连接URL到page，进行爬，发现新的URL也爬，重复。

具体怎么爬要思考一下，通常使用bfs避免过度爬，也可以dfs如果不是从root进去的。另外关键就是关联路径，比如(http://foo.com/a/b/page.html), 那么就要尝试爬/a/b, /a/, /.的路径

画图：（略）  
关键的一点是不要重复爬，所以这里有个Duplicate Eliminator部分，此外还有一个Extractor是用来解析HTML documents的

### 6.Detailed Component Design
画图：（略）  
这里不是系统组成，更多的是单个service内部的各个components的连接。

1.The URL frontier：这是个数据结构包含了所有剩余URLs，然后FIFO来读就行，类似Queue结构。每个service应该会发过来一个queue files，遍历这个就行。    
2.The fetcher module：这个就是用来下载Http Page的模块。  
3.Document input stream：据说用来避免重复下载的。不是很清楚。  
4.Document dequpe test：也是用来避免重复下载的，可以用MD5或者SHA来编码document来做checksum。  
5.URL filters：可以处理一些blacklist，限制domian，protocol之类。  
6.Domain name resolution：用来map那个IP和address的。可能需要本地自定义。  
7.URL dedupe test：还是用来去重。在每个service内部需要一个in memory cache被所有thread共享。  
8.Checkpointing：记录点，可以周期性记录当前解析状态，即使某个node失败重启就能继续跑。

这里的设计主要就是一个service内部的设计，基本就涉及到了获取url和去重。 
 
本地也需要几个存储部分：
1.分配的URL文件（文件）
2.已经处理过的URL（数据库）
3.用于内容判重用的checksum（数据库）
4.文件存储（s3）存储解析完毕的HTML内容

没有全部nodes整合后的处理啊，我想所有文件解析完毕要汇总到一个中央数据库吧，这个数据库的设计估计是：  
url pk  
ip  
checksum  
s3link  
created_date  
update_date  

### 7.Fault tolerance
这里比较重要。

是用类似consistent hashing的方式，每个机器都有环上其他机器的信息。然后每个机器map到多个virtual node，然后尽量均匀地hash到一个ring上，每个crawling job也map到这个ring上，这样workload能尽量平均。而且宕机了就当处理文件交给vitural node的下一个节点处理。

然后每个server会定期的跑checkpoint并把分析好的存入disk。

负载均衡的延伸思考：
https://www.1point3acres.com/bbs/thread-436948-1-1.html
负载均衡的做法也是类似。简单的做法，每个url一个hash，hash的取值范围就是1 到 1billion。把hash分成10K个组，每个组一个range。这个range就是一个partition，或者叫shard。每台机器就负责一个shard。
更好的做法可以参考consistency hash，或者直接用cassandra存取这些URLs。可以到几百上千的node，支持10K机器没啥问题。

### 8.Data Partitioning 
这里比较重要。

主要要考虑3个数据存放：  
1.要访问的URL  
2.URL的checksum用于dedupe（重复数据删除）  
3.Document的checksum用于dedupe（重复数据删除）  

我们通过hostname来distributing URLs，发过去以后这些数据就保存在了每个service上。因为用了consistent hashing的方式，所以某台机器宕机后就把那台机器对应hash的url发给环上下一个node。

然后各个service定期将结果dump（文件）发回到远程中央服务器。避免重复获取。

### 9.Crawler Traps
很多坑，比如网站的spammer扫描，被block等等。

### 其他
补充内容来源[爬虫系统与搜索建议系统](https://marian5211.github.io/2018/03/10/%E3%80%90%E4%B9%9D%E7%AB%A0%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1%E3%80%91%E7%88%AC%E8%99%AB%E7%B3%BB%E7%BB%9F%E4%B8%8E%E6%90%9C%E7%B4%A2%E5%BB%BA%E8%AE%AE%E7%B3%BB%E7%BB%9F/)

##### 用dfs还是bfs爬？
用bfs，因为dfs容易爬过深，爆栈。bfs是放到queue里面，比较容易并行操作。

##### 怎么单机多线程爬虫
多个线程都会从队列里向外拿URL，相当于对队列的一次写操作，这时候就会发生冲突，也就是互斥

需要注意的是，争取共享资源，就要考虑以下三个机制：  
1）sleep —— 就是睡一会，设置一定的时间，然后回来看看资源能用了不。但问题是：在道资源可以使用的第一时间知道，效率很低  
2）condition variable 信号量（实现互斥）—— 相当于所有的线程都在等着，然后信号量通知他们能用的时候，就都立马去抢占用资源，谁抢到算谁的  
3）semaphore —— 就像门上挂上五把钥匙一样，每次可以进去5个人，出来的时候还钥匙，也就是允许同一时间有多个线程访问用一个资源。  

思考一个问题，既然多线程这么快，是否可以尽量多的开线程，比如开一万个？  
不行，原因如下：  
1）线程之间来回切换(context switch)还是有花费（需要保存线程执行状态，还需要切换二级缓存等）的，因此线程不能太多，线程过多效率会很低  
2）一个CPU在同一时间只能处理一个线程，如果机器只有一个CPU线程依然在排队   
3）线程端口数目是有限的（TCP/IP协议中，端口只有2个字节，也就是65536个端口，操作系统还会预留一些端口给其他服务）  
4）网络带宽有限制，线程很多，单台的带宽是不能够满足的  

所以需要改进为分布式爬虫

##### 怎么分布式爬虫
（这里假设是使用中央queue的概念，当然我们把分配好的爬虫url以文件形式直接发给每天机器也行）  
分多台机器爬取，就可以突破单台机器的限制了！此时分布式爬虫依然共享一个内存中的URL queue.

分布式爬虫虽然解决了线程不能太多的问题，但是又带来了一个问题：URL队列在内存中放不下了（假设1trillion，差不多要40T的内存）！这是不行的。那么我们考虑一下把URL queue存在硬盘里，也就是数据库里。

但是放到数据库里之后，又有一个很要命的限制是：没有办法控制网页抓取的顺序和优先级啊！

解决：给数据库加入一个优先级的列，再加一个频率的列，把我之前的数据库表改造一下，变成如下：  
url pk  
ip  
checksum  
s3link  
created_date  
update_date  
priority：优先级  
available_time：控制抓取频率，也就是这个时刻之后再进行抓取。如果本次有更新，那么就把时间设置地更近一些；如果本次没有更新，那么就把时间设置地更远一些  

##### Scale的考虑
（这里主要考虑了中央库task table的设计，这个库包含了所有要处理的url队列）  
拆表sharding，加速访问，有一个细节需要注意，需要一个scheduler，用来安排去哪里要数据。shard可以是根据网站的域名来划分。

还有如果是全球性的爬虫，也可以按照区域分片task table，然后定期和全世界的数据库进行同步。

### 10.Other
这里也有一些讨论，在10000台分布式机器的问答  
https://leetcode.com/problems/web-crawler-multithreaded/discuss/617672/Anyone-have-answers-to-the-follow-up

