## Designing Facebook Messenger 笔记

### 1.What is Facebook Messenger?
就是即时消息服务，可以使用手机或者网站进行chat。

### 2.Requirements and Goals of the System
Functional Requirements：  
1.Messenger支持one-on-one对话  
2.Messenger可以跟踪用户的online/offline状态  
3.Messenger可以持久化聊天历史

Non-functional Requirements：  
1.系统Latency必须很小  
2.系统必须高consistent，用户可以在任何设备看到他们的聊天历史  
3.Messager是高availablity的，不过满足高一致性是第一位 

Extended Requirements：  
Group Charts：可以多人同时聊天，就是聊天室啦。  
Push Notifications：如果对方不在，可以离线通知。  

### 3.Capacity Estimation and Constraints
假设每天有500M的活跃用户，平均每个用户每天发送40条信息。这样就有20B数据每天。  

Storage Estimation:  
假设每条消息100bytes，存储每日信息则需要2T的空间。  
20 billion messages * 100 bytes => 2 TB/day  
而存储5年的聊天数据，就需要3.6petabytes的数据空间。  
2 TB * 365 days * 5 years ~= 3.6 PB  
除此之外，我们还需要存储用户的信息，和message的metadata。

Bandwidth Estimation：  
一天的数据量是2TB，那么至少需要25MB的带宽：
2 TB / 86400 sec ~= 25 MB/s  
因为message是双向的，所以服务要接受一方的信息发送另外一方，所以upload和download带宽都是25MB，总共50MB。  

### 4.High Level Design
就是画图。  

实际的workflow要保证高consistent，所以还需要ACK，和TCP很像：  
1.userA通过chat server发送message给userB  
2.chat server收到message并发送ACK给userA  
3.server存储message到数据库，并且发送message给userB  
4.userB接收message并发送ACK给server  
5.server告知userA该message已经成功发送到userB  

### 5.Detailed Component Design
**a.Messages Handling** 
How would we efficiently send/receive message?  
聊天室的我们早就知道使用长链接了，看看这里的方案吧。两种选择：  
1.pull model：用户周期性问服务器有没有新消息  
2.push model：用户和服务器保持连接，服务器通知user有新消息  
如果用第一种方法，很明显有latency，而且大多数时间都是收到空消息，浪费流量。显然要采用第二种方法。  

How will clients maintain an open connection with the server？  
可以使用HTTP的Long Polling技术或者WebSockets。long polling就是客户端请求服务器，如果服务器有新数据就发回去，没有没有就一直等，直到有新数据就马上发回去，完成了这次请求。Websocket则是升级版的Long Polling了，只需要一次握手，就可以让客户端和服务器保持网络通路了，效果最好。

How can the server keep track of all the opened connection to redirect messages to the users efficiently?  
服务器可以维护一个hash table，其中的Key就是UserID，而value就可以是connection object。每当服务器收到请求，直接从hash table里面找对应连接就行。  

What will happen when the server receivers a message for a user who has gone offline？  
当接收方断线，服务器需要通知发送方发送失败。如果这只是个临时断开，那么需要等待用户重连。可以让发送方尝试重发消息，另外失败消息也应该存放在用户客户端，这样才不需要重新录入。（这里注意用户连接服务器后就能将连接信息存放在hash table）

How many chat servers we need？  
假设我们有500M的连接，一个当前的服务器大概可以搞定50K的同步连接，则我们需要10K台服务器。(真有这么多？）

How do we know which server holds the connection to which user？  
可以在chat server之前使用软件load balancer，可以根据userID进行重定向
//TODO 这是什么LB？

How should the server process a 'deliver message' request?  
服务器需要：  
1.将message存放到数据库（这个是异步进行）    
2.将message发送给receiver（每次用户登录，该用户连接server就存放到hash table）  
3.发送ACK给sender（通知事务完成）

How does the messenger maintain the sequencing of the messages?  
可以考虑在每个message加上timestamp，这个timestamp就是服务器收到信息的时间。但是特定情况下也无法保证顺序，比如A和B同时发信息，A发了A1，B发了B1，在A可能看到的顺序是A1，B1，而B看到的顺序是B1，A1。  
解决的方法就是对于每个client保持特定的sequence，这样对于每个用户来说，所有devices上的显示是一致的。不过其他用户看到的可能不一样了。

**b.Storing and retrieving the messages from the database**  
每当chat server收到信息，需要存放到数据库中去，有两种方案：  
1.启一个separate thread，用来存数据到数据库。  
2.发一个asynchronous的请求到数据库进行存储。  

设计数据库需要考虑以下的问题：  
1.怎么让database connection pool工作更有效  
2.怎么进行retry failed requests  
3.如果多次失败怎么log这些failed requests    
4.怎么重试这些logged requests

Which storage system we should use?  
我们需要一个能够支持高效多次的小updates，并且能够快速获得一个range的records的数据库。因为用户关心sequentially访问数据。  
所以我们不能使用RDBMS比如MySQL，也不能使用NoSQL比如MongoDB，因为如果每次用户收发信息都进行数据库操作会造成大量的latency和load。（TODO不能用内存存储吗？）  
这里推荐的存储是wide-column database列式数据库比如HBase。Hbase是column-oriented key-value NoSQL数据库，一个key可以存放多个values（SortedMap<String, SortedMap<String, String>>）。Hbase是基于Google's BigTable并且泡在Hadoop Distributed File System(HDFS)的。HBase会先将数据存放在内存buffer，一旦buffer满了，就dump数据到disk中。这样可以存储很多小数据，并且在读取的时候可以定位到某个key或者扫描一个range的rows。Hbase也适用于高效的存放可变大小数据，很适合message系统。

How should clients efficiently fetch data from the server?  
Clients使用paginate来从服务器获取数据（也就是range of rows）。每个用户的paginate不一样，比如手机端的屏幕小，显示少，就返回少量的messages。 

**c.Managing user's status**  
我们需要跟踪用户的online/offline状态，并且提示相关用户是否有状态变化。因为我们已经维护所有active用户的connection object（用于long polling），所以可以方便得知用户的状态。但是我们一共有500M在线用户，如果每次状态改变都发送，很消耗资源。可以这么优化：  
1.当用户客户端登录，pull所有他们的好友状态  
2.当A发送信息给B，而B下线了，发送failure给A并更新B状态  
3.当B上线，等待几秒看是否B又下线，避免重复检验  
4.客户端可以从服务器主动获取用户状态，不过不应过于频繁，主要是服务器来进行广播  
5.当用户和其他用户开始聊天，客户端可以请求状态

Design Summary：  
这里有图。  
1.用户连接到chat server  
2.服务器和用户建立连接，此时cache server开始mapping上long polling connection和server  
3.当用户发信息，存放到了HBase数据库  
4.服务器会广播用户状态给相关用户（用户好友）  
5.客户端可以在某些情况pull好友状态

### 6.Data partitioning
数据量很大(3.6PB 5年)，分区是必须的。有几种方案：

Partitioning based on UserID：  
假设使用UserID分区，假设一个DB shard是4TB，那么需要3.6PG/4TB~=900个shards。假设我们有1K个shards，我们可以用hash(UserID)%1000来进行分区。因为一个用户数据在一个shard，读取也是很快的。  
类似pintest的shard方法，一开始可以准备少量的物理服务器，每个服务器装有多个数据库，一个数据库还能有多个instance，等容量大了，就把instance移出来到单独的服务器去。  
假设我们历史数据无限大，可以使用大型logical partitions，每个partition对应多个物理服务器，当数据量变化，就增加物理服务器的配置（TODO，这里需要了解具体做法）

Partitioning based on MessageID:  
这个不好，因为搜索序列化的数据会很慢。不适合这个场景。

### 7.Cache
我们可以cache每个用户的viewport的数据，比如说15条，每次登陆能快速读取到。因为分区是按照userID，所以cache也是设置在每个服务器的。

### 8.Load balancing 
需要再chat servers前面增加load balancer，这个LB可以将UserID映射到当前存放该用户connection的服务器去。同样，在cache servers前面也可以增加LB（因为每个服务器有一个cache，用LB来定位cache服务器）

### 9.Fault tolerance and Replication
What will happen when a chat server fails？  
chat server存放着user的connection，如果宕机了，将connection转移是一件非常困难的事情（faliover TCP），简单点的做法就是让用户自动重连（TODO这样可以连接到没宕机的服务器？）

Should we store multiple copies of user messages？  
不能只有用户数据的一个copy，这样一旦该服务器宕了，数据就丢失了。这样就需要将多份copies存放在多个服务器，或者使用技术比如Reed-Solomon encoding来分布和备份。TODO

### 10.Extended Requirements
a.Group chat  
可以将group-chat objects独立出来。group-chat使用GroupChatID定义，并且维护这个group的参与者。load balancer可以通过GroupChatID来定向服务器，服务器就iterate所有用户来找到对应的服务器来处理这些connection。在数据库，也可以使用单独的表分区，可以用GroupChatID来分区（TODO，需要更多信息）

b.Push notifications  
上文的notification主要用来通知失败消息，不过也可以给离线用户发信息。比如手机收到的微信消息提示，这些Device（比如苹果手机或者browser）都有notification的机制可以调用。  
除此之外，我们需要设置单独的Notification server，用来处理和发送离线消息给manufacture的push notification服务器。

### 总结
需要了解wide-column database的应用场景。关于memory cache啊，variable size data和range scan之类的相关知识。
关于replicate这块也没看明白。group chat也讲得很粗略

### Reference
[WebSocket 是什么原理？为什么可以实现持久连接？](https://www.zhihu.com/question/20215561)







