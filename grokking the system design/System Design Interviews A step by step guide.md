## System Design Interviews: A step by step guide 笔记

system design interview(SIDs)现在越来越多了，因为光刷题难以区别面试者了（大家都刷leetcode），而系统设计因为没有标准答案，可以挖掘很深，而且本身需要多年经验，所以更容易刷掉candidates。这里是几个steps：

Step 1: Requirements clarifications  
首先当然需要讨论好题目，了解这个系统需要由哪些功能。以twitter为例，下面几项可能都是需要讨论到的：  
* 用户是否可以发送twitter和follow其他用户  
* 我们需要设计显示用户的timeline不  
* tweets包含photos和videos不  
* 我们只涉及backend还是也要开发前端？  
* 用户可以search tweet不  
* 我们需要显示热门trending topics吗  
* 当有新推特会push notification功能不

Step 2: Back-of-the-envelope estimation  
下面就是要量化系统规模。通常涉及到scaling，partitioning，load balancing和caching。  
* 系统的预期规模（比如每天的新tweet数量，tweet的浏览量，每秒的timeline生成量等）  
* 需要多大的存储？如果tweets包含图片视频会需要不同的储存需求  
* 网络带宽预期？通常带宽和流量控制和load balance息息相关  

Step 3: System interface definition  
定义APi，这一步主要作用是让接口更清晰，确保是符合requirement的。比如twitter的API有： 
postTweet(user_id, tweet_data, tweet_location, user_location, timestamp, …)  
generateTimeline(user_id, current_time, user_location, …)   
markTweetFavorite(user_id, tweet_id, timestamp, …)  

Step 4: Defining data modal  
设计数据存储（数据表，也有可能是Json结构等），主要是用来阐明system components之间的flow，同时也能指引了后续的数据分区和管理。比如Twitter：  
User: UserID, Name, Email, DoB, CreationData, LastLogin, etc.  
Tweet: TweetID, Content, TweetLocation, NumberOfLikes, TimeStamp, etc.  
UserFollow: UserdID1, UserID2  
FavoriteTweets: UserID, TweetID, TimeStamp  
同时也应该决定是使用SQL还是NoSQL，或者混合的方案。是否需要图片和视频存储等。

Step 5: High-level desgin  
这一步是整体设计，需要画图说明需要的模块以及各模块之间的联系。比如twitter：  
twitter需要多个应用服务器来服务所有read/write requests，会需要load balance来负载traffic，因为是read heavy，所以单独服务器处理。在后端，也需要保证高效数据系统来support大量读。这意味着一个分布式的文件存储系统来存储图片视频。  
主要就是画图。

Step 6: Detailed design  
这一步就是重点环节的详细设计，需要非常详细的说明pros和cons，tradeoffs等。  
* 因为要存放大量数据，怎么分区和分布到多个数据库？是将一个用户所有数据存放在同一个数据库吗？有什么问题？  
* 怎么处理热门用户（发很多tweet或者follow很多人）  
* 因为用户timeline包含了最近的tweets，我们是否用某种方式存放来优化最新的tweets？  
* 在哪里引入cache来加快速度？  
* 哪个部分需要更好的负载均衡？

Step 7: Identifying and resolving bottlenecks  
尝试针对尽可能多的瓶颈问题使用不同的解决方案。  
* 系统中是否存在single point failure？怎么mitigate it？
* 我们有足够多的的replicas吗？当一些服务器宕机的时候，是否可能保证还能提供服务？  
* 同样的，我们是否能保证个别服务器的failure不会导致整个系统shutdown  
* 如果监控系统性能？在重要部分宕机或者性能差的时候是否有预警？




