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

### Reference
[WebSocket 是什么原理？为什么可以实现持久连接？](https://www.zhihu.com/question/20215561)







