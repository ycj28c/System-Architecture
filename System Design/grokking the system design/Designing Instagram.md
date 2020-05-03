## Designing Instagram 笔记

### What is Instagram?
Instagram是个社交网络服务，允许用户上传和分享照片和视频。用户可以选择分享到public或者private，如果是public的就可以所有人看到，private则只有设定的用户可见。Instagram也允许用户分享给其他网络平台比如Facebook，Twitter，Flickr和Tumblr

### Requirement and Goals of the System
先确认需求

Functional Requirements  
1.用户可以upload/download/view photos  
2.用户可以根据photos/video的title进行搜索  
3.用户可以follow其他用户  
4.系统可以生成和显示follow用户的最Top图片

Non-functional Requirement  
1.我们的服务需要highly available，因为不怎么宕机  
2.News Feed生成的系统延迟最多200ms
3.一致性不是最重要，用户可以忍受图片不马上显示  
4.系统应该highly reliable，意味着上传的图片和视频不丢失  

### Some Design Considerations
这是一个read-heavy系统，设计上应该重视如果快速检索图片。

1.用户可以上传无限多的图片。所以需要有效的管理存储。  
2.当观看图片的时候，要求low latency  
3.数据100%可靠，用户上传的图片不丢失  

### Capacity Estimation and Constraints
*假设我们有500M用户，每天有1M活跃用户。  
*有2M的新图片每天，每秒大概23个新图片
*平均图片大小>=200k  
*可以计算出每天空间需要2M * 200KB >= 400GB  
*可以计算出10年需要400GB * 365 * 10year \~= 1425TB

### High Level System Design
此处需要画图  

需要支持两个场景，一个是上传图片，另一个是查看和搜索图片。我们的服务器需要Object storage来存储照片，当然也需要数据库来存放图片的metadata信息。

### Database Schema
我们需要存储用户数据，上传图片，follow的信息。Photo table存放所有和图片有关的信息，需要一个（PhotoID，CreationDate）的index用来快速存取，因为我们优先获取最近的图片。

*Photo*  
PhotoID: int (PK)  
PhotoPath: varchar(256)  
PhotoLatitude: int  
PhotoLongtitude: int  
UserLatitude: int  
UserLongtitude: int  
CreationDate: datetime  

*User*  
UserID: int (PK)  
Name: varchar(20)  
Email: varchar(32)  
DateOfBirth: datetime  
CreationDate: datetime  
LastLogin: datetime  

*UserFollow*  
UserID1: int  
UserID2: int
PK: (UserID1, UserID2)

最直接的方法就是用RDBMS比如MySQL，因为我们需要joins。但是关系型数据在scale方面比较困难，详细可以看SQL vs. NoSQL部分。我们可以存放上面的结构到分布式Key-value存储，所有的metadata可以放入table，key是PhotoID，value就是图片相关的Object，比如创建时间等。

我们可以将photos存放到分布式文件存储比如HDFS或者S3

我们需要存放将用户和图片的关系，也需要存放用户follow的list，这两个表我们都可以使用一个wide-column数据库比如Cassandra。对于UserPhoto表，key可以是UserID，value就是该ID所有的PhotoIDs列表，存放在不同的columns里面，同样的schema样式也可以用于UserFollow表。Cassandra或者key-value数据存储通常都会维护一定数量的replicas提供reliability。而且，这些数据存储删除都不是直接发生，数据会保存一定的天数然后永久删除。(TODO)

### Data Size Estimation
假设需要存放10年数据，请计算需要多少存储能力。  
*User:*  
假设int和dateTime都是4bytes，那么每行数据需要  
UserID(4bytes)+Name(20bytes)+Email(32bytes)+DateOfBirth(4bytes)+CreationDate(4bytes)+LastLogin(4bytes)=68Bytes (TODO)  

假如我们有500million用户，就需要32GB总存储。  
500million*68\~=32GB

*Photo:*  
每行Photo大概284bytes，  
PhotoID(4bytes)+UserID(4bytes)+PhotoPath(256bytes)+PhotoLatitude(4bytes)+PhotoLongitude(4bytes)+UserLatitude(4bytes)+UserLongitude(4bytes)+CreationDate(4bytes)=284bytes  

假如每天有2M新照片，那么我们需要0.5GB存储每天：  
2M * 284bytes \~=0.5GB per day  
如果10年就是1.88TB存储。

*UserFollow:*  
每一行UserFollow数据是8bytes，如果我们有500million用户，平均每人follow 500个users。那么我们需要1.82TB的空间。  
500 million users * 500 followers * 8bytes \~=1.82TB  

如果10年那么需要3.7TB数据  
32GB + 1.88TB + 1.82TB \~=3.7TB

### Component Design
照片上传和写入可能比较慢，因为他们需要走disk，而读操作会快一些，特殊是可以加上cache。上传用户可能消耗掉所有可用连接，这意味着读取操作也会受到写操作影响。请记住Web服务器是由连接数限制的，假设1个web服务器有500个连接，那么总共不能超过500的读和写操作。我们需要将读写分布到不同的服务器中，我们需要读写分离。  
此处有图。

### Reliability and Redundancy
系统要求不能丢失文件，所以我们需要多个文件拷贝，一旦某个存储死了还可以从其他数据源获取。这个套路也适用于其他系统组成部分。如果我们想要高可用Availability，我们就需要多个replicas，这样的Redundancy冗余则会删除单点系统故障。




