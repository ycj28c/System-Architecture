## Designing Dropbox 笔记

### 1.Why Cloud Stroage?
现在人们要从多个设备比如手机，电脑，平板等访问文件，cloud很有必要。主要好处有：  

Availablity：任何时间地点都可以访问文件  
Reliability and Durability：提供100%的持久和可靠性，提供了多点容灾  
Scalability：只要付钱就能扩展容量

### 2.Requirements and Goals of the System
1.用户能上传下载文件到他们的设备  
2.用户可以分享文件给其他用户  
3.提供自动synchronization服务  
4.系统最大能存放1GB的大文件  
5.ACID-ity必须，也就是Atomicity，Consistency，Isolation和Durability  
6.提供离线编辑，用户可以在本地修改，连线后会自动sync

额外需求：  
系统提供数据的snapshot，用户可以回退到之前版本

### 3.Some Design Considerations
* 预期有大量的读写操作  
* 读写比例类似  
* 文件被拆分为很多chunks(比如4MB)，这样失败后只需要尝试少部分失败文件  
* 更新操作也只需要更新有变化的chunks  
* 删除重复chucks，节省空间和带宽  
* 在客户端存放一些metadata，比如文件名，大小等，避免和服务器过度交互  
* 微小的操作用户可以只上传diff而不是chunk

### 4.Capacity Estimation and Constraints
* 假设我们一共有500M用户，其中100M日活用户(DAU)  
* 假设平均每个用户有3个设备  
* 假设每个用户平均200个文件，那么一共就有100B个文件  
* 假设平均文件大小100K，那么一共有100B*100KB=>10PB  
* 假设我们每分钟有1M的连接数  

### 5.High Level Design
此处需要画图。  
这里需要有服务器处理用户从Cloud Storage上传下载(Block Server)，专门服务器处理metadata关于文件和用户信息(存放于SQL或者NoSQL数据库)，还需要通知所有用户任何更新文件的更新(Synchronization server)。
```
         / Block Server ------------ Cloud Storage
		/                       \
Clients  --- Metadata Server --- Metadata Storage
        \         |
		 \ Synchronization Server
```

### 6.Component Design

#### a.Client  
1.上传和下载文件  
2.在workspace中发现文件变化  
3.处理offline文件修改conflict或者concurrent的同步文件

How do we handle file transfer efficently？  
将大文件拆分为4MB每块的chucks，具体块的大小和device，网络带宽，平均文件大小有关。每个chuck都要记录metadata信息。

Should we keep a copy of metadata with Client？  
在用户本地记录metadata的拷贝，可以让用户离线更新，在sync时候节省时间

How can clients efficiently listen to changes happening with other client？  
一种是periodically向服务器请求是否有更新。很显然浪费带宽，影响服务器性能。  
另一种就是HTTP long polling，在客户端登陆后和服务端保持连接，一旦有更新才会发送客户端。

可以将客户端行为分为下面4个部分：  
1.Internal Metadata Database：客户端内置数据库跟踪所有文件，chucks还有它们的versions以及文件位置信息。  
2.Chucker：该组件可以拆分文件，也可以组合文件，并且可以检测文件变化。  
3.Watcher：监控本地workspace，通知indexer任何用户的增删改操作。并且接收服务器端的synchronization服务器的broadcast  
4.Indexer：这个组件比较复杂。从watcher获取event信息，更新本地metadata数据库关于chuck变化信息。当chuck成功提交下载，indexer还会请求synchronization server广播变化给其他用户，并更新远程metadata数据库。就是其他的活都indexer干了。

How should clients handle slow servers?  
客户端可以尝试retry，使用指数delay时间增长来retry。

#### b.Metadata Database  
Metadata数据库维护了不同版本的metadata，包括Files，chucks，user，devices和workspace。可以使用MySQL(SQL)或者DynamoDB(NoSQL)。Synchronization service需要提供文件的一致信息，尤其多个用户同时操作同一个文件也要保持一致。SQL数据库天然支持ACID，如果使用NoSQL则需要在Synchronization service增加同步逻辑层。

#### c.Synchronization Service
服务端的Synchronization Service负责处理客户端提交的文件修改，并将修改应用到subscribe的其他用户。也会将用户本地metadata数据库和remote metadata数据库进行同步，是整个系统最重要的部分。  
Synchronization Service需要尽量减少数据传输，所以需要有一套diff algorithm来减少文件的传输量。具体就是我们把文件拆成4MB每个小chuck，而且只发修改的chuck。服务器和客户端可以计算chuck的hash(比如SHA-256算法)来判断local copy是否需要更新。更新保存的不是全新的copy，只是diff的部分。  
为了高效和规模化的处理同步协议，可以考虑在client和synchronization service之间增加一层middleware，也就是queue来平衡负载。

#### d.Message Queuing Service
Queue将包括两个部分：  
1.Rquest Queue是global queue，对所有client共享。所有用户metadata请求都会发送到Request Queue，到synchronization service进行更新。  
2.Response Queue是在文件更新结束后，发送update messages到每个客户端。因为message在用户接收到后就被删除，所以还需要为每个subscirbe的clients创建response queue。

#### e.Cloud/Block Storage
Cloud/Block Storage是存放实际文件和chucks数据的地方，用户直接和Storage联系进行发送和接收操作。metadata数据是单独存放，这样隔离metadata和文件的方法让我们可以使用在cloud或者in-house都能操作文件。  
这里有Detailed compnent design for Dropbox的图。

### 7.File Processing Workflow
当client A更新了与B，C共享的文件，会通过Response queue发送，如果B，C不在线，message queuing就会将update notification一直保留在separate response queue直到B，C上线。  
1.Client A upload chucks to cloud storage。  
2.Client A updates metadata and commits changes。  
3.Client A 得到成功确认，并且关于change的notification会发送给B和C。  
4.Client B和C获得metadata修改信息，并下载更新的chucks

### 8.Data Deduplication
Deduplication专门用来消除重复，提高存储空间利用率。可以通过对每个chuck进行hash计算判断文件是否相同。我们可以用两种方式进行deduplication：  

a.Post-process deduplication  
就是先把新chuck存到storage去，然后进行分析找出duplication。这样客户端不用等待hash结果，可以保证storage的性能。缺点drawback是1）在短时间内存放了重复数据，2）重复数据的传输占用带宽

b.In-line deduplication  
可以实时进行deduplication hash计算。当某个chuck被判断为已经存在，只记录和发送metadata用于reference。

### 9.Metadata Partitioning
在scale metadata db有两种方式：  
1.Vertical Partitioning：就是分表，比如把user相关表放到数据库A，文件相关表放到数据库B。不过有以下问题，当有trillion的chunks的时候仍然有单表过大的问题，而且join两个表会非常慢。  
2.Range Based Partitioning：可以通过首字母分表，比如A开头和B开头的放到不同的表去，这样会导致表大小不平衡，比如E开头的就特别多。  
3.Hash-Based Partitioning：通过Hash功能，比如把FileID平均hash到[1-256]的区间去

### 10. Caching
在处理热门files/chunks使用类似Memcached的键值cache，一个商业cache大概有144GB的内存，这样的服务器可以cache36k的chunks。类似的cache机制有LRU之类。

### 11. Load Balancer (LB)
在client和Block servers之间，还有client和Metadata server之间都可以加LB。通常的LB可以使用Round Robin，服务器轮流接受请求，不过可能存在服务器已经满负荷仍然向其发送请求的情况，所以可以配置更高级的LB机制（根据服务器负载来或者机器性能来等等）。

### 12. Security, Permissions and File Sharing
用户需要将文件存放到cloud，所以安全security和隐私privacy很重要。尤其是用户将文件public或者共享给其他用户的场景，我们需要再metadata DB存放每个文件的permission来决定哪个文件可以显示给哪个用户。

### 总结
具体indexer的使用，还有本地的详细设计

### Reference







