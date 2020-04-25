## Designing Twitter 笔记

### What is Twitter?
在线社交服务，用户发送最多140字长的信息。没注册的话只能读，不能写信息。用户可以用过website，SMS和Mobile app访问Twitter

### Requirements and Goals of the System
先确认需求：

Functional Requirements  
1.用户能发新推特  
2.用户可以follow其他用户  
3.用户可以favorites推特  
4.服务可以创建和显示用户follow的用户发送的信息，以时间顺序  
5.推特也可以包含photo和videos

Non-functional Requirement  
1.服务需要高可用  
2.时间轴生成的可接受latency是200ms  
3.用了可用性可以牺牲部分一致性，一个用户没有在短时间看到推特可以接受

Extended Requirements  
1.搜索功能  
2.回复功能  
3.话题趋势，当前的热门话题/搜索  
4.可以Tagging标记其他用户  
5.有Notification  

### Capacity Estimation and Constraints
假设我们有10亿用户，每天有2亿活跃用户(DAU)，每天有1一条新推特，每个用平均follow 200个人。

*How many favorite per day？*   
平均，每个用户favorite 5条推特每日。  
200M users * 5 favorites => 1B favorites

*How many total tweet-views will our system generate?*    
也就是一共要创建多少条推特给所有用户，假设每个用户平均访问timeline 2次每天，并且访问5个其他用户的pages。 如果每个页面用户能看到20条tweets，那么我们的系统每天创建28B/day的tweet-views  
200M DAU * ((2 + 5) * 20 tweets) => 28B/day  

*Storage Estimates*  
每天tweet有140个Characters，一个Characters需要2bytes，那么一条tweet需要280bytes。 假设我们还需要30bytes来存放每条tweet的metadata（比如ID，timestamp，UserId等）。那么每天的总存储需要：  
100M * (280 + 30) bytes => 30GB/day  
此外我们还有media数据，假设每5条推特有1张图片，每10条推特有1个视频。 假设每个photo有200KB，每个Video的大小是2MB。那么每天需要的media存储是：  
(100M/5 photos * 200KB) + (100M/10 videos * 2MB) \~= 24TB/day  

*Bandwidth Estimates*  
每天总共24TB数据，所以每秒钟290MB数据，我们需要28B的tweet-views每天，还需要展示图片和视频。假设用户3个视频只看1个，那么：  
(28B * 280bytes)/86400s of text => 93MB/s  
+ (28B/5 * 200KB ) / 86400s of photos => 13GB/S  
+ (28B/10/3 * 2MB ) / 86400s of Videos => 22GB/s  
Total \~= 35GB/s

### System APIs
我们可以用SOAP或者REST APIs来实现。下面是一个Posting推特的API
```
tweet(api_dev_key, tweet_data, tweet_location, user_location, media_id)
```
api_dev_key(String)：这个是用来表示注册账号的。  
tweet_data(String)：这是推特文本，大概140个字。  
tweet_location(String)：可选项，该推特选择的GPS位置(longtitude, latitude)  
user_location(String)：可选项，用户的GPS位置(longtitude, latitude)  
media_ids(number[])：可选项，该推特关联的图片或者视频文件等，会单独上传

返回(String)是访问发送tweet的URL，否则返回HTTP error

### High Level System Design
此处需要画图  
我们需要一个系统能有效的存储新的tweet，100M/86400s => 1150 tweets per second and read 28B/86400s => 325K tweets per second。所以这是一个read-heavy的系统。  

我们需要多个application server来服务所有请求，appliation servers前面则需要load balance来负载均衡。在后端，我们需要一个高效的数据库存储所有推特，而且可以提供大量的读取。而且我们还需要一个file storage来存储图片和视频。

尽管我们的期望是1亿写和280亿的读取量每天，所以平均我们的系统将受到大概1160的新推特和325K的读取请求每秒。这些流量在一天中比较平均，但是在高峰期可能有几千的写请求和1M的读请求。在具体设计的时候要记住这一点。

### Database Schema
数据库主要有下列表：  
Tweet表：  
TweetId：int(PK) 
UserId：int  
Content: varchar(140)  
TweetLatitude: int  
TweetLongtitude：int  
UserLatitude: int  
UserLongtitude: int  
CreationDate：dateitme   
NumFavorites：int  

User：  
UserID: int (PK)  
Name: varchar(20)  
Email: varchar(20)  
DateOfBirth: datetime  
CreateDate: datetime  
LastLogin: datatime  

UserFollow:  
UserID1: int   
UserID2: int  
PK(UserID1, UserID2)  

Favorite:  
TweetID: int  
UserID: int  
CreateDate: datetime  
PK(TweetID, UserID)  

关于选择SQL数据库还是NoSQL数据库，请看Designing Instagram --!

## Data Sharding