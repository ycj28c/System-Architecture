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


