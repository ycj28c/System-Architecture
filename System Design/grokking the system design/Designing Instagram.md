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

我们需要存放将用户和图片的关系，也需要存放用户follow的list，这两个表我们都可以使用一个wide-column数据库比如Cassandra。对于UserPhoto表，key可以是UserID，value就是该ID所有的PhotoIDs列表，存放在不同的columns里面，同样的schema样式也可以用于UserFollow表。



