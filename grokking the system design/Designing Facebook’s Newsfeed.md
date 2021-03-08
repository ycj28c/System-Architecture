## Designing Facebook's Newsfeed 笔记

### 1.What is Facebook’s newsfeed?
就是Facebook主页的更新，包括了状态更新，图片，videos，links，app activity，还有like的people，pages，groups和follows等等。

### 2.Requirements and Goals of the System
Functional Requirements:  
1.根据follow的用户，pages还有groups的posts更新来发送Newsfeed  
2.一个user可以有很多freinds，也可以follow很多pages和groups  
3.Feed可以包含images，videos或者只是text  
4.当有新post马上发送给所有active user

Non-functional requirements:  
1.要生成newsfeed real time，最高latency不应该超过2s  
2.当有获取新newsfeed的请求，5s内用户就能获得数据

### 3.Capacity Estimation and Constraints
假设一个用户有300个friends，follow 200个pages

假如300M日活用户，平均每天刷5次timeline，那么就有1.5B的请求，大概17500/s QPS。大概20台机器了

假如每个用户要存放500个post在内存，假如每个post 1KB，那么需要500KB每个用户。每天就是150TB内存，如果一台服务器100GB，那么需要1500台机器来支持。

存在内存？

### 4.System APIs
getUserFeed(api_dev_key, user_id, since_id, count, max_id, exclude_replies)  
使用Json

since_id：这个相当于created_time的作用  
count：默认一次尝试读取200个比如  
max_id：和since_id对应，返回一个范围内的数据  
exlude_replies：boolean，就是禁止comment的功能

### 5.Database Design
主要需要3个对象：  
1.User： 
2.Entity：包括了groupd和pages  
3.FeedItem：这里假设只有user能够发送feed  

User：
```
userID: int pk
name: varchar(20)
email: varchar(32)
dateofbirth: datetime
creationdate: datetime
lastlogin: datetime
```

Entity:
```
entityId: int pk
name: varchar(20)
type: tiny int
description: varchar(512)
creationdate: datetime
category: samllint
phone: varchar(12)
email: varchar(20)
```

UserFollow:
```
userid - entityOrFriendId (combine pk)
type: tiny int
```

FeedItem:  
注意有like，可以是user发的，也可以是来自entity，location数据？
```
FeedItemId: int pk
UserId: int
contents: varchar(256)
EntityID: int
locatonLatitude: int
locatioLongtitude: int
CreationDate: datetime
Numlikes: int
```

FeedMedia：
```
FeedItemID - MediaID combine pk
```

Media:  
怎么这个也需要location？
```
MediaID: int pk
type: smallint
Description: varchar(256)
paty: varchar(256)
LocationLatitude: int
LocationLongtitude: int
CreationDate: datetime
```

### 6.High Level System Design

##### Feed generation生成feed  
就是当user和entities在有post时候生成。当系统收到生成feed请求，就做了如下事情：  
1.检索所有用户follow的userId和entities  
2.找到最新的那些有更新的ids  
3.rank一下，不知道为啥rank  
4.将这feed放入cache，并且只返回前20条  
5.当用户移动到页面最下，显示后面20条（用了pagenation）

如果用户在线，就大概5分钟跑一次获取，然后将新生成数据假如到当前cache中。

##### Feed publishing
可以是用户pull，也可以是server push。

需要下面的组成部分：  
1.Web Servers，就是facebook主服务器了，这个是前端服务器    
2.Application server，用来泡workflow和存放数据库，这个是后端总服务器    
3.Metadata database and cache，存放Users，Pages和Groups的metadata  
4.Posts database and cache，存放发的post的数据，这个独立出来写了？    
5.Vidoe and photo storage，and cache，就是文件系统存储  
6.Newsfeed generation service，收集posts，并存到cache，也用来增加news feed到user timeline  
7.Feed notification service，给用户发通知有新feed的服务器，啊

画图（略）

总的来说，就是newsfeed只放在cache中啊，然后feed notification service从里头读

### 7.Detailed Component Design
a.Feed generation  
标准的sql的获取方法
```
(SELECT FeedItemID FROM FeedItem WHERE UserID in (
    SELECT EntityOrFriendID FROM UserFollow WHERE UserID = <current_user_id> and type = 0(user))
)
UNION
(SELECT FeedItemID FROM FeedItem WHERE EntityID in (
    SELECT EntityOrFriendID FROM UserFollow WHERE UserID = <current_user_id> and type = 1(entity))
)
ORDER BY CreationDate DESC 
LIMIT 100
```
不过有一些问题：  
如果post多速度慢，如果在线处理，每个更新都需要更新所有followers导致大量backlogs。要改进，需要预先生成timeline和存放到memory

Offline generation for newsfeed：  
改进方法就是离线也周期性生成用户newsfeed并存放到memory中。当用户请求，直接给他们生成好的。要记录lastGenerated时间，然后排序使用LinekdHashMap或者TreeMap<FeedItemID, FeedItem>在内存排序。就可以方便的移除老数据和翻页了。

Should we generate (and keep in memory) newsfeeds for all users?  
很多用户可能并不经常登录，就不用一直离线生成了，可以用LRU删除不频繁用户的离线feed任务。更高级的则是比如AI找出用户登录习惯。

b. Feed publishing  
The process of pushing a post to all the followers is called fanout。就是pull或者push的选择了。  

1."Pull" model or Fan-out-on-load:   
就是用户主动pull，很多时候没更新就浪费流量

2."Push" model or Fan-out-on-write:  
就是long poll，有一个问题就是当用户有millions of followers(celebrity-user)，那么服务器一次要push给很多人。

3.Hybrid:  
和twitter timeline一样，对于celebrity这些hot user，采用pull，而普通人就使用push来推送news feed。还有一种方式就是只发送给用户的在线朋友。

Should we always notify users if there are new posts available for their newsfeed?   
手机端可能需要流量，所以可以让用户只pull来refresh，而不是自动更新。

### 8.Feed Ranking
就是排序一下，比如根据number of likes啊，comments数量啊，shares数量啊，更新时间啊等等来排序。

### 9.Data Partitioning
这里主要说是如何partition那些news feed的数据。

重要

a.posts和metadata的分片
比较难，大致是和twitter一样，通过postID和create time同时分片，具体看Diesining Twitter

b.news feedd的分片
比较简单，因为都是在memeory，通过userID来分片，因为每个userID最多500条，所以不会出现某用户数据量过大不能放在一台服务器的情况。  
未来的扩展和replication则是使用consistent hashing，要具体思考一下。具体就是通过hash fuction将feed分不到不同的内存服务器，某个内存服务器宕机不用，会平衡转移到下一台virtual node。












