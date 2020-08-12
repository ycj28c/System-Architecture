## Designing an API Rate Limiter 笔记

API Rate Limiter就是用来限制用户的请求数量的。

### 1.What is Rate Limit?
就是有大量请求，但我们服务能力有限，需要限制请求数量。Rate limiter就是通过在固定时间窗口内限制user，device，IP之类。比如：  
*用户每秒只能发一个信息  
*用户每天最多3次失败的credit card transactions  
*一个IP每天最多新建20个账号

### 2.Why do we need API rate limiting?
好处很多，比如防止Denail-of-service(DOS)攻击，穷举密码，穷举信用卡转账等。这样攻击和真实用户很想，很难detect却能轻松让你宕机。Rate limiting还能减少基础设施投入，停止spam等等。下面是Rate limiter的应用场景：
  
1)Misbehaviing clients/scripts：减少可能是内部或者外部的大量不当请求。  
2)Security：限制用户穷举密码（2-factor auth的情况）。  
3)To prevent abusive behavior and bad design practices：可以优化客户端的开发，防止程序总是重复请求同样的信息。  
4)Revenue：可以设置不同limit的销售策略，创造营收。  
5)To eliminate spikiness in traffic：在高峰期可以尽可能让每个人都能使用。

### 3.Requirements and Goals of the System
Functional Requirements：  
1)需要能够限制API单位时间请求量，比如15requrest/seconds  
2)APIs可以是在cluster内，所以rate limit必须考虑不同服务集合。在一个服务器超过限制或者服务集合超过限制都应该返回错误信息。

Non-Functional Requirements：  
1)系统需要高可用，因为rate limiter还有安全保护作用  
2)必须低延迟

### 4.How to do Rate Limiting?
通过在APIs或者Application级别增加Throttling，当超过throttle，服务器返回HTTP错误429-Too many requests。

### 5.What are different types of throttling?
几种throttling的类型：  
1)Hard Throttling：固定数量的API请求限制。  
2)Soft Throttling：百分比的机制，比如限制每分钟100条信息，10%的超额限制，也就是最多110条信息每秒。  
3)Elastic or Dynamic Throttling：弹性限制，只要系统有额外资源，就允许用户超过100条信息限制。

### 6.What are different types of algorithms used for Rate Limiting？
主要有两种： 

1)Fixed Windows Algorithm：固定时间，比如0s-59s是一个窗口，60s-119s是一个窗口。
  
2)Rolling Window Algorithm：就是滑动窗口啦

### 7.High level desgin for Rate Limiter
就是画图了（略），Rate Limiter在Web Server之后，在API Server之前，连接了Stroage和Cache Server，目的是先判断是否超Limit，如果没超过才能访问API Server

### 8.Basic System Design And Algorithm（Fixed Window）
如果是Fixed Window，因为每分钟重置计时，我们使用这个Map的结构：  
```
  Key: UserID
Value: {Count, StartTime}  
```
问题在于在每个时间块的中间，用户可以迅速的使用两个时间块的limit，这一点我们可以用滑动窗口算法解决。  
另外如果是分布式系统，或者多线程情况，会导致Atomicity问题，如果我们使用Redis存储key-value，可以使用Redis Lock（？？），也可以使用Memcached，不过增加锁会减慢处理速度。如果是在code里处理，比如使用Hashtable，可以自定义locking来解决原子性问题。  

How much memory would we need to store all of the user data?  
假设UserID是8bytes，Count是2bytes，time是4bytes，不过如果只存分钟和秒，只需要2byte，那么一个用户就是8+2+2=12bytes。假设hashtable有20bytes的overhead，如果我们需要跟踪1million用户，就需要(12+20)bytes*1million >=32MB。假如我们还需要4byes来锁住每个用户，那么一共约36MB内存。  
当规模大的时候，不可能在一台机器运行rate limiter，假设每个用户10个request每秒，那么就有10million的QPS。

### 9.Sliding Window algorithm
我们可以将每个请求的timestamp存放在sortedmap，比如Redis的Sorted Set（？？）
```
  Key: UserID
Value: Sorted Set{time1, time2, time3}  
```
如果有新的request过来，就先删除历史事件，再判断sorted set大小，如果可以就添加，比较容易理解。

How much memory would we need to store all of the user data for sliding window?  
假设UserID是8bytes，epoch time是4bytes，假设限制是500个请求每小时，hashtable有20bytes的overhead，而sorted set也有20bytes的overhead，那么我们一共需要：
```
8+(4+20(sorted set overhead)) * 500 + 20(hash-table overhead) = 12KB
```
如果需要跟踪1million用户，12KB*1million需要约12GB的内存。

sliding window相对Fixed Window算法需要更多的内存，这是一个scalability问题。

### 10.Sliding Window with Counters
这是一种折中的做法，就是把时间60等分，比如每小时500条信息，然后每分钟不能超过10条。那么60等分后就是1分钟，每分钟统计一次request counter，然后1小时的就是前60分钟的counter总和。  
可以使用Redis Hash（？？？）来存放counters，据说100个key以内特别高效

How much memory we would need to store all the user data for sliding window with counters?   
假设UserID是8bytes，epoch time是4bytes，couter是2bytes，假设500条信息限制每小时。hashtable和redis hash都有20bytes的overhead，因为我们以分钟为单位记录counter，所以每个user最多有60个entries，所以一共需要1.6KB每个user：
```
8+(4+2+20(Redis hash overhead)) * 60 + 20(hash-table overhead) = 1.6KB
```
如果1million用户就是1.6GB内存。  

这样的sliding window with counters算法减少了大概86%的内存使用量，相对于简单的sliding window算法。

### 11.Data Sharding and Caching
在大规模情况下，可以使用UserId来分布用户数据，使用Consistent Hashing来解决fault tolerance和Replication问题。如果每个APIs使用不同的限制，那么就用userId+API的combo来分布数据。

//TODO 还有部分没看

### 12.Should we rate limit by IP or by user?
如果limit by IP：  
如果多个用户使用一个public IP（比如手机用户使用同一个咖啡厅网关）就会出现问题。还有一个问题就是IPv6的地址很长，如果黑客用大量的IPv6攻击，很容易out of memory。

如果limit by User：  
用户登录后就可以应用rate limiter。有一个问题就是login API也需要rate limiter，否则黑客也能用DOS炸掉login。

How about if we combine the above two schemes?  
使用Hybrid的方法： 
可以同时使用两种方法进行rate limiter，因为两种情况单独使用都有weakness缺点。

## 总结
比我想象的要复杂，不是一个sliding window就能解决，还要考虑内存的因素。规模化肯定是要考虑的，这里提到的Shard和具体的方案比较少。另外，这里提到了很多redis的技术点，比如sorted set，redis hash之类不知道具体是啥，还需要专门研究。