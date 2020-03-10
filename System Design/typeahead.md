## AutoComplete 系统设计概述
Typeahead就是twitter的一个JavaScript的库，相当于1个搜索栏，具体请看[typeahead.js](https://twitter.github.io/typeahead.js/)，大概是2013年左右的项目。因为简单直观，用于系统设计讨论很好，这种叫做设计auto complete系统。

1.有默认的搜索项目
2.可以进行局部匹配，类似模糊匹配
3.有建议项

## 确认问题/需求
1.How many entries/user？10 Billions
如果用户少单机都可以搞定，用户越多需要逐步扩展。基本的scale就是从单机 -> 双机 -> 多机负载均衡 -> 100亿级别那在各个层次都需要大规模负载均衡，比如多数据中心。根据用户位置路由到最近机房。

2.Latency requirements： 100 ms 99 percentile
少的延迟意味着更快的返回结果，比如可以边计算边访问，需要在算法和硬件结构上配套。

3.RPS 系统的访问量
几千RPS，几万RPS之类，在入口服务器的选择上就会有区别。比如FB在用户多了以后就按照学校的IP来分机器，更多了就需要大规模集群了。另外在用户端，比如web端也需要做限制，比如延迟请求。

4.系统数据量/来源
系统需要处理的数据量，还有生成的大量log，TB，PB级别的log的话也需要集群存储。然后比如map reduce之类进行大量的log分析。根据数据保存时间，也可以像银行一样分为实时访问的近期数据，和历史数据存储（延时）。

5.VM spec？4 CPUS，16GB RAM，20GB Shared disk
如何做计算？ SLA(qps, latency, availability time)，同样要结合用户数，请求数量等其他条件计算来判断需要的VM数量。

6.可用性，分区，一致性要求
CAP原理，分区容错性Partition tolerance，可用性Availability，一致性Consistency不可兼得，最多符合两者。
一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）
可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）
分区容忍性（P）：以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。

一致性和可用性，为什么不可能同时成立？答案很简单，因为可能通信失败（即出现分区容错）。如果保证 G2 的一致性，那么 G1 必须在写操作时，锁定 G2 的读操作和写操作。只有数据同步后，才能重新开放读写。锁定期间，G2 不能读写，没有可用性不。如果保证 G2 的可用性，那么势必不能锁定 G2，所以一致性不成立

[CAP 定理的含义](https://www.ruanyifeng.com/blog/2018/07/cap.html)

## 总体设计  
1. 画图整体设计
源数据：logs -> map reduce，如果db -> query
算法： trie
前端：user -> type -> webpage fontend -> http request
后端：gateway -> lb 到service

下面对于每部分进行详细设计

## 数据获取部分设计
数据来源可以是离线日志处理系统，通过Map Reduce或者spark之类的批处理或者streaming处理系统，对日志分析，生成trie。

如果数据来自db，数据库设计（SQL/NOSQL，表和表关系）
//TODO

如何获取出现频率?
```
//TODO
```

## 分布式设计
大规模如果分布式sharding/replica，

consistent hashing，

可用性Availability

load balance来均分高


## 接口设计，前后端接口
前端：
每次用户在搜索框输入字母发送请求

如何减少系统qps？如何做缓存？
在keyup时间时候发送请求，不过增加一个延迟比如100ms，减少请求数量。使用browser 的local storage，对于进行过的请求，直接返回结果。
//TODO 研究local storage

后端API：

API设计：
GET请求(没必要POST因为长度有限），api/getByPrefix/text，或者api/getByPrefix?work=xxxxxx&&userId=xxxxx
方法：
getByPrefix(String prefix)来处理前端进行请求。

## OOP设计，类和数据结构的设计
本题因为只涉及到简单的字符和返回结果，不需要OOP

6. 优先资源优化，trade off，怎么减少log，cache。前端HTTP expire，browser localstorage来自动cache之类。比如前端设置last keyup才进行query处理，delay50ms ~ 100 ms。在前端的HTTP header增加Expires和Cache-Control(max-age=3600)来缓存请求3600秒

## 算法上设计
如何实现，使用Trie。 
画图，在TrieNode上应该使用Array或者hashtable来支持sparsity，unicode国际多元。 另外使用List来记录frequent of all childrens(Top K的结果），每级进行merge。 每周创建更新的offline TopK。
```
class TrieNode {
	Map<Character, TrieNode> children;
	List topWords;
}
```

BigO：在遍历的复杂度仅为O(L搜索单次的长度), Space是O(NL)

单机装不下一个完整的trie怎么办？
如果增加几天机器就够用，那么直接把Trie按照首字母拆分到几个机器去。更多的就需要集群存储，存储对使用者透明。

搜索词的prefix分布很不均衡怎么办？
很明显，A开头的words肯定比X开头的要多，按照占用空间分离开即可。

如何做用户个性化的结果？
个人认为在每个TrieNode可以准备多套TopK，根据用户的设置来返回不同的TopK

存储Trie的节点如果挂了怎么办？ 如何提高可用性？


高级设计：
各种关联问题，比如监控，KPI，

热门事件之类（trending queries）
可以个性化设置高权重query和TopK的值。如果


哇，细想的话涉及的东西真是不少

## Reference
1. [Google系统设计面试题--设计一个auto complete 系统(type ahead系统)](http://www.noteanddata.com/interview-problems-system-design-learning-notes-autocomplete.html)  
2. [Type ahead/Auto Complete 设计](https://www.jianshu.com/p/c7fc9092d9fe)