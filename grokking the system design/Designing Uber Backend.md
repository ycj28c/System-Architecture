## Designing Uber backend笔记
Uber是高读高写的典型代表，难度相对较大，必须掌握

### 1.What is Uber?
打车软件，用户和司机都通过smartphone的Uber app联系

### 2.Requirements and Goals of the System
有两个角色，1）Driver 2）Customers  
有以下场景：  
1.司机持续的向服务器通知他们的当前位置和载客状态  
2.乘客可以看到所有附近的有效司机  
3.顾客可以请求a ride，附近的司机会被通知  
4.当司机和顾客接受a ride，他们可以持续的看到各自的当前位置知道旅程结束  
5.当到达目的地，司机完成状态并可以接收下一单

### 3.Capacity Estimation and Constraints
*假设有300M的乘客，1M的司机，每天有1M的活跃乘客和500K的活跃司机。  
*假设每天有1M的rides。  
*假设每隔3秒所有的司机都会发送当前位置。  
*当用户请求ride了，系统可以实时的联系司机。

### 4.Basic System Design and Algorithm
之前介绍过Yelp，使用的是QuadTree的地图数据方法，Uber和Yelp最大的不同是有大量的写。所以需要解决以下2个问题：  
1）因为driver每3秒就更新一下位置，我们也需要相应地更新我们的数据结构。但是如果每次司机坐标变化都要更新，实在是太耗费时间和资源了。我们必须根据driver之前的坐标来找grid，如果新坐标不在之前的grid里，就需要把司机从之前的grid删除并插入到当前的grid中。此外，如果新的grid到达了最大限制，还需要继续分区。  
2）我们需要一个快速机制将所有附近的司机发给任何当前区域的活跃用户。并且，如果a ride正在进行的话，系统还需要通知司机和乘客当前位置信息。  

还是可以用QuadTree，不过需要改进快速update的性能。

Do we need to modify our QuadTree every time a driver reports their location？  
QuadTree肯定是要更新，否则无法获得准确的driver信息。不过我们可以不这么频繁的更新QuadTree，而且将实时的driver信息存放在Hashtable中，每隔15秒才更新一下QuadTree。我们可以把这个Hashtable称为DriverLocationHT。  

How much memory we need for DriverLocationHT？  
需要存放的信息有：
1.DriverID(3bytes - 1 million drivers)  
2.Old latitude(8 bytes, 一个int是4byte，32bit)  
3.old longtitude(8 bytes)  
4.New latitude(8 bytes)  
5.New longitude(8 bytes) Total = 35bytes  
如果我们有1M的driver，需要内存为1M * 35bytes >= 35MB，很小啊

How much bandwidth will our service consume to receive location updates from all drivers?  
我们只需要接收DriverID和location信息（3+16=19bytes），如果每3秒接收一次500K的driver，需要3.2M/s带宽就足够了，比想象的要小吧。  

Do we need to distribute DriverLocationHT onto multiple servers？  
虽然我们的内存和带宽要求都不大，单机都能满足。但是为了scalability，performance和fault tolerance的考虑，还是需要将DriverLocationHT随机分布到多台服务器去。每个服务器需要做以下2件事情：  
1.每当服务器接收到driver的location，需要broadcast给所有的有兴趣用户。  
2.服务器需要通知特定的QuadTree刷新Driver的位置，大概10-15秒的频率。  

How can we efficiently broadcast the dirver's location to customers？  
我们需要一个Push Modal来让服务器给相关用户进行推送。可以使用publisher/subscriber模型的专用Notification服务器用来专职推送。  
客户端：用户登录Uber app，然后app开始去服务器进行query最近的driver信息。  
服务端：在将drivers返回给用户app之前，我们可以让用户subscribe订阅所有这些drivers的信息。这样就有一个subscribers的list，每个subscriber感兴趣一系列的drivers。每当这些drivers的DriverLocationHT发生了变化，就将最新的Driver信息广播给subscribers。这样就能保证用户总能接收到driver的最新位置。

How much memory will we need to store all these subscriptions？  
根据之前的条件，我们有1M的每日活跃用户和500K的每日活跃driver。假设平均1个driver有5个customers来subscribe。假设所有的信息都存放在hashtable，我们需要有customerIDs和DriverIDs来维护subscription的关联。假设DriverID是3bytes，CustomerID是8bytes，那么一共需要：  
(500K * 3) + (500K * 5 * 8) ~= 21MB  

How much bandwith will we need to broadcast the driver's location to customers？  
因为每个driver有5个订阅者，所以一共有subscribers数量：  
5 * 500K >= 2.5M  
每个subscriber需要发送DriverID（3bytes）和坐标（16bytes），一共需要：  
2.5M * 19 bytes >= 47.5MB/s  

How can we efficiently implement Notification service？   
可以使用HTTP long polling长连接的方式或者使用push notifications的方式（用户连接上后，server单方便的发）

How will the new publishers/drivers get added for a current customer？  
之前我们说过采用的是subscribed的模式来跟踪附近的driver，这只发生在用户刚登陆Uber app的第一次query。如果后面有新的driver进入了用户区域该如果添加？如果是要动态的添加driver，也需要知道并跟踪用户的area，这会让方案非常复杂。所以针对新driver的情况，可以考虑让客户端主动来从server端pull信息。  

How about if clients pull information about nearby drivers from the server？  
如果使用用户请求的方式。用户可以将自己的当前坐标发给服务器，然后服务器负责在QuadTree中找到所有附近的drivers，并将结果返回给用户。Uber app客户端收到信息就可以将车辆以及当前坐标显示在app的地图上，用户可以每5秒请求一次避免过于频繁。  

Do we need to repartition a grid as soon as it reaches the maximum limit？  
可以设置一个缓冲cushion，比如说10%，当QuadTree的容量超过和小于10%的指定容量，才进行partition/merge，这样能减少QuadTree的操作Traffic。  

这里有图，比较复杂。

How would "Request Ride" sue case work？  
1.用户请求ride  
2.一个Aggregator服务器会接受请求并让QuadTree服务器返回最近的drivers  
3.Aggregator服务器收集信息并按照ratings排序  
4.Aggregator服务器给最优的（3）个driver同时发单，最先接受订单的driver获得派单，其他的driver会收到取消。如果没有任何driver回复，Aggregator服务器会尝试从后面3个drivers  
5.一旦有driver接受了请求，用户会获得通知  

### 5.Fault Tolerance and Replication 
What if a Driver Location server or Notification server dies？  
首先每天服务器都需要replica啊，一旦primary server宕了，马上secondary server顶上。此外，还可以将数据存放到快速硬盘比如SSDs中（因为数据小啊），如果primary和secondary都宕的情况可以从persistent storage中快速恢复。  

### 6.Ranking
rank方法有很多，通常使用就近（proximity）来rank，但是还可以使用popularity和relevance来rank。

How can we return top rated drivers within a given radius？  
假设我们在QuadTree中跟踪每个driver的overall ratings，popularity可以用一系列的指标计算出来，比如driver有几颗星啊。当我们要在附近radius里面搜索top10的driver时候，可以从多个QuadTree节点中找出top10，并进行aggregate。

### 7.Advanced Issues 
1.How will we handle clients on slow and disconnecting networks?  
2.What if a client gets disconnected when they are a part of a ride? How will we handle billing in such a scenario?  
3.How about if clients pull all the information, compared to servers always pushing it?  

## 总结
Uber是个重写重读的系统，是比较复杂的，好在数据量比较小。组成很多，有QuadTree，hashtable，aggregate服务器，client和driver，notification等等，十分复杂，需要将各个知识点串起来。本篇文章讲解的并不是很详细，很多地方细想还是不懂，比如如何更新QuadTree和Hashtable，服务器之间的同步问题，还有persistent数据库的选择，还有driver和client的用户数据结构，还有API的设计都没有提到。还需要看看相关的文章才行。










