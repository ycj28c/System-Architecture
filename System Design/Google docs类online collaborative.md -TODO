## Google Docs System design 笔记

[Google Docs System design | Part 1| Operational transformation | differentail synchronisation](https://www.youtube.com/watch?v=2auwirNBvGg)  
[Google Docs System design | part 2| System components explanation micro services arcitecture](https://www.youtube.com/watch?v=U2lVmSlDJhg)  

### 1.什么是online collaborative
就是会有多个人同时编辑同一个文档，比如Google Docs, Etherpad, Mockingbird等

### 2.需求
1）需要多人编辑，满足Concurrency（这个是难点）
2）要高性能，低延迟

### 3.Concurrency方案
可以用lock解决吗？  
不行，因为多人编辑，锁了其他人就无法一起编辑了。所以必须lock-free。

Pessimistic Concurrency control？

Optismistic Concurrency control？  
用类似version的机制来控制，也就是A在修改的时候，B也能看到A的修改。这样就要解决Sync的问题（conflicts resolution）。

Sync Strategis  
1）Event passing机制，可以用Character by Character sync，也可以用line by line sync。  
Character by Character就是每当用户修改，就实时发送。问题是这要求对所有的event，比如增删改都需要sync，而且要发送给所有人，显然要处理的数据量会很大。Line by line就是隔一段时间获取A对某个line的操作，发送给其他人，但是如果两人对某行进行大量修改，可能就会让这一样错乱。  
2）differential Sync机制（和git的diff很类似）  
隔一段时间发送所有A的diff到服务器，用version进行控制。

有时候会两种方案都用。Google docs是用event passing（Operational Transformation - OT）。

### 4.Operational Transformation细节
event会有很多，比如该字体，颜色等等。这里主要就讨论了增删改。比如：
```
Insert(A, 1)
Delete(T, 10)
Update(X, Y, 20)
```
考虑一行的场景，比如一个共享文档有"AT"，所以这个时候用户1和用户2都有"AT"。当用户1进行删除Delete(T,1)->"A"，而B在进行Insert(H,0)->"HAT"。当同步以后会出现问题：
```
用户1:
Delete(T,1)-> "A" -> get [Insert(H,0)] -> "HA"
用户2：
Insert(H,0)->"HAT" -> get [Delete(T,1)] -> "HT"
```
可以发现结果不同步了。（所以拳皇街机模拟器网络运行会乱，就是这个问题）

实际场景会有更多的用户，所以还会有一个中央服务器来汇总和处理。另外，要应用transform来处理event。
```
xform(a,b) = a'b'

用户1:
Delete(T,1)-> get [Insert(H,0)]，进行转换 xfor(D, I)就变成了D(T,2)，I(H,0)，这样再操作"AT"就是 "HA"

用户2：
Insert(H,0)->"HAT" -> get [Delete(T,1)]，进行转换 xfor(I, D)就变成了I(H,0)，D(T,2)，这样再操作"AT"就是 "HA"

大概是这样。。。
```

关于OT control在wiki也有详细解释：[Operational transformation](https://en.wikipedia.org/wiki/Operational_transformation)

### 5.Differential Sync细节
比如用户1有文档c和拷贝cc，server有文档s和拷贝sc。
```
用户1（修改）：
增加一个character，c和cc进行diff，发送给服务器

server：
将用户1的patch和sc进行diff，结果放入s（在修改时候lock住server的s）。结果要广播给所有其他的用户，然后每个用户进行diff和merge。
```
实际情况和git一样，可能会有冲突啊，这样可能就需要用户决定是否要discard目前的修改，或者进行conflict修改等等了。google docs还会直接覆盖的。

### 6.Operationial Transform和Differential Synchronization优劣？
OT：  
不是很稳定，在处理conflict时候不是很可靠。会发送大量的message，服务器压力大。

Diff：  
diffing的成本也是挺高的，修改一个标点可能也需要diff全文（或者某个分片），在文档大的时候，diff很明显减慢。

具体处理都需要通过server，server会需要将操作进行存储，比如cassandra是个不错的选择。如果网速较慢，用户可能发送了多个修改，但是只有第1个server进行了处理，后续的还没有处理到，这种情况用户就需要等待第1个操作返回结果。

### 7.高级设计
主要是图，可以参考一下youtube上的https://www.youtube.com/watch?v=U2lVmSlDJhg  
整体上可以采用Microservice的架构。
好处包括：
1.便于分工，解藕，module隔离  
2.单个功能相对简单，便于理解和开发快速上手  
3.可以使用不同语言来写，用RPC连接  
4.方便scale，哪个server有瓶颈就扩展哪个

包括了：  
API gateway：作为client连接的入口。  
Authendication permission：连接到了API gateway，进行用户验证。 
Comments： 就是comment功能，因为数据量大，类型单一，估计是存放在nosql db。  
Email/GCM Notification：连接到API gateway，就是第一时间返回用户提醒。  
AppServer（Export Doc/Import Doc）：导入导出doc，比如转换为PDF，连接到API gateway。  
RDBMS：存放用户信息和文档的meta data信息。连接到Authentication，AppServer和Session Server。  
Nodejs Websockets：就是主要的用户编辑doc的功能部分。  
Operations queue：也就是用户对doc的更新，都通过queue传递到Session Server去。  
Cloud Storage：用来存放binary文件，比如export的pdf。  

### 8.Websockets详解  
如果每次用HTTP连接，又hand shake啥的太浪费时间，所以直接用websocket的长连接。这样每次用户编辑或者服务器有更新就可以real-time进行更新。其他用户的操作比如cursor用不同的颜色表示，也是因为websocket的支持才能实现。如果结合redis？还能实时进行和其他用户的chat。

### 9.API Gateway详解  
（这里有点歧义，之前看到的微服务架构client是直接联系到server的，微服务其实是有网关和无网关的设计，有网关的偏多） 
对于microservices来说，API Gateway是极其重要的。因为所有的用户entry就是API gateway，比如Netflix zuul，API gateway需要让正确的server来contact和处理用户请求。包括了几个知识点：  
1）Single Entry point，比如用户一个登陆，API gateway就请求了notification啊，Authendication啊等服务，而对于用户就一个login请求。  
2）Security，这个很容易理解，因为隔离了服务器和用户。  
3）Dynamic Service discovering：就是服务发现，比如netflix的eureka之类，用来注册和管理动态的server连接。  
4）Service partition hidden：比如Amazon商品的rating和reviews服务要分拆，直接拆分microservice并进行重新注册即可（注册服务通过key来管理）  
5）Hiding microservice：具体架构对于用户是透明的。用户通过界面交互即可。  
6）circuit breaking熔断：比如一个login需要ABC 3个服务都返回结构才算完成。那么如果一个不这么重要的小部件C宕机了，那么使用async，其他服务结果仍然返回结果。然后API rating会注销C部件直到C注册服务重新恢复。  

### 10.front end设计  
主要就是HTML奇数，富文本使用类似canvas来做。视频中没有详细解释。
