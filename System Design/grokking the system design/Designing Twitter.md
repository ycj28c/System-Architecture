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

