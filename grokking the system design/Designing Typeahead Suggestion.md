## Designing Typeahead backend笔记
类似于auto suggestion

### 1. What is Typeahead Suggestion?
就是输入后提示建议啊

### 2. Requirements and Goals of the System
Functional Requirements: 
当用户输入query，服务提供前10个建议  

Non-function Requirements:   
建议应该马上显示，和google那种一样，我们说200ms以内显示吧

### 3. Basic System Design and Algorithm
就是用Trie数据结构，利用离线周期性更新

Should we have case insensitive trie? 为了简化问题，这里不管大小写

How to find top suggestion? 就是把所有以当前节点结尾的count都记录在本节点。根节点应该很大啊。

Given a prefix, how much time will it take to traverse its sub-tree? 不知道，log(N)效率

Can we store top suggestions with each node? 如果全存上需要大量内存，只存top10还行。

How would we build this trie? 就是trie算法

How to update the trie? 
就是trie算法，假设每天有5billion的search，那么qps是60k/s，所以肯定不能在线更新。会非常影响读取。  
正确的做法是离线，可以使用Map reduce的做法统计短语的频率，周期可以是1小时，统计前一个小时的高频词。形成一个snapshot。具体做法有：  
1）创建原来机器的trie副本，更新完毕后switch一下
2）利用主备服务器，更新完备机切换一下

How can we update the frequencies of typeahead suggestions? 这里不是很懂。如果是全新建的，不用担心，如果只是更新原来的trie，那么需要根据count更新top10的值。

How can we remove a term from the trie? 一种是从trie数中删除，一种是加上filter layer过滤掉结果。

What could be different ranking criteria for suggestions? 还可以用freshness，user location，language等来细分rank

### 4. Permanent Storage of the Trie
How to store trie in a file so that we can rebuild our trie easily - this will be needed when a machine restarts?   
这里很关键，我们可以将整个trie存放在文件里，作为一个snapshot。这是一个serializing和deserializing的操作。文中提到层次存放，比如C2,A2,R1,T,P,O1,D。  

这里提到了top suggestion的处理，相当于backtracking，是先build完毕子节点，然后把所有children的top10汇总一下，一路到root的。

### 5. Scale Estimation
假设一天有5billion的search，相当于60k qps，假设20%的是唯一的，并且我们只想index top50%（10%？）的数据，那么大概是100million的数据需要index

Storage Estimation: 
如果一个短语30个Character/bytes，那么总共需要100million*30bytes=3GB  
这里还假设每天成长2%的terms，那么一年后3GB+(0.02 * 3GB * 365days) = 25GB

### 6. Data Partition
a. Range Based Partitioning:  
就是用第一个字母分区，主要问题就是unbalanced。‘E’就有特别多的单词。  

b. Partition based on the maximum capacity of the server:  
就是根据内存容量来分区，比如：  
Server 1, A-AABC  
Server 2, AABD-BXA  
Server 3, BXB-CDA  
如果用户type了A，就找server1，不过如果type了AA，就需要同时找server1和server2了。 

这里可以用一个load balance（估计是自定义的软件lb）在所有trie server前面，存放这个分区关系和redirect traffic。

如果需要访问多个node的情况，最后要合并结果，可以在server端也可以在client端进行合并。

不过这种做法还是有可能hotspots，就是个别短语访问量特别大。

c. Partition based on the hash of the term:   
也可以把每个term进行hash，然后根据hash建Trie树，这样比较能避免hotspot。但也有缺点，就是搜索一个词，几乎要遍历所有服务器了。

### 7. Cache
对于热搜，可以做cache，就是把最热门的search terms单独拿出来放到trie服务之前。甚至还可以用ML来辅助predict热搜。

### 8. Replication and Load Balancer
需要replicas给load balance和trie servers。

### 9. Fault Tolerance
What will happen when a trie server goes down? 首先由主备机，然后还有snapshot

### 10. Typeahead Client
还有一些客户端的优化：  
1.按键间隔50ms后才发送请求  
2.如果用户一直type，就可以取消正在执行的请求  
3.用户可以从服务器存放一些预处理数据？  
4.用户可以存放最近历史，极大可能reuse  
5.在用户打开website就建立连接，这样用户开始type以后可以快速开始？  
6.服务器可以push一些cache到cnd和internet service provider（isp），具体怎么操作？  

### 11. Personalization
根据用户的搜索历史，localtion等信息可以生成个性化信息。可以存放在本地或者在独立的服务器。




