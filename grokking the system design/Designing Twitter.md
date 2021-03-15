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
因为我们每天都有大量的新推特，所以读的load特别高，必须将负载分布到多个机器中去。下面是集中分片方案：  

*Sharding based on UserID*  
我们可以存储所有某UsedID的数据在一个服务器。包括该用户的tweets，favorites，follows等，直接对userID使用hash function就可以，但是问题也很明显：  
1.如果一个用户热门，那么对该服务器的读取量会很大，影响性能  
2.有些用户的数据量可能特别多，造成维护困难  

*Sharding based on TweetID*  
就是针对TweetID进行hash，分布到随机服务器。在搜索的时候，需要查询所有服务器，每个服务器返回满足条件的结果，然后centralized server将结果进行aggregate并返回用户。这里用timeline生成为例：  
1.首先application server找到这个用户的所有follows  
2.App server发送query给数据库服务器，找到那些follows的所有tweets
3.每个数据库找到这些推特，按照recency排序返回top的推特  
4.App server合并结果和排序，并返回top结果给用户  

这种方式解决了hot用户问题，但是比起Sharding by UserID，我们需要查询所有数据库分区来找到一个用户的推特，可能会导致高latencies。此外，我们还可以为hot推特进入cache（部署在数据库前）

*Sharding based on Tweet creation time*  
按照时间分片可以让我们更容易更快的找到top推特，因为我们只需要查询少量服务器。而问题就是traffic load过于集中，无法分摊。比如，在写数据时候，所有的推特都会写入1个服务器。

*What if we can combine sharding by TweetID and Tweet creation time?*  
（这一段比较重要）  
可以同时按照推特creation time和TweetID来分片，这样可以快速的获得latest推特。所以这样要保证每个tweetID是universally unique的，并且每个推特数据都需要含有timestamp。  

所以这里的tweetID就需要这么设计：包含两部分，第一部分表示epoch seconds，第二部分是自动生成的递增序列。这里存放epoch time是精确到秒，所以需要31bits来记录（为什么，因为86400 sec/day * 365 (days a year) * 50 (years) => 1.6B，而1.6B就是31bits)。对于auto incrementing sequence，我们期望1150条新推特每秒，所以可以分配17bits用来存储递增序列，2^17>=130K，完全足够，而且每秒钟都可以重置auto incrementing sequence。这样31bits+17bits就是48bit的TweetID。

### Cache
我们可以在数据库层级前面增加cache服务器来cache热门的tweets和users。

*Which cache replacement policy would best fit our needs?*  
很显然Least Recently Used(LRU)是最适合的

*How can we have a more intelligent cache?*  
同样可以使用80-20 rule，20%的tweets生成了80%的traffic，所以我们需要cache 20%的每日数据(在每个shard分片)

*What if we cache the lastest data?*  
对于Tweet来说很适合，80%的用户只看最近3天的数据，所以我们可以只cache最近3天的数据。如果我们有专门的cache，用来cache所有用户3天的数据，那么光推特就有300million或者90G的数据量，这意味着需要100G的内存。这个数据量可以很简单的放入到一个服务器中，不过我们需要一个replicate减少单机负载。每当我们生成用户的timeline，都可以先请求cache服务器。  

我们的cache和Hash table一样，key是OwnerID，value则是一个doubly linked list包括用户的所有推特。因为我们需要先显示最近的，所以新推特插入到双链的head，删除时候就删除双链的tail

### Timeline Generation
关于timeline生成，参考Designing Facebook's Newsfeed

### Replication and Fault Tolerance
因为是read-heavy系统，所以每个DB分区都需要secondary database，这个secondary server只用来读取，分担流量。所有写操作首先进入primary server，然后replicated到secondary servers。这样也能提供fault tolerance，因为只要primary server goes down，就可以failover到secondary。

### Load Balancing
我们可以在3个地方增加LB：  
1.在用户和Application servers之间  
2.在Application servers和数据库之间  
3.在Aggregation servers和cache server之间  
通常，使用简单的Round Robin方法就可以了，这种方法很容易应用且不会带来额外开销，而且当一个服务器down的时候，LB会吧traffic转移其他的服务器。而这种方法的问题在于Round Robin不会考虑各个服务器的负载，所以更聪明的LB会定期发送queries到服务器，检查各个服务器的load并且动态调整traffic。

### Monitoring
监控系统也很重要，我们需要收集下列metrics/counters，来把握系统性能：  
1.多少个新推特每天/秒，峰值是多少  
2.Timeline的delivery状态，多少推特被发送每天/秒  
3.平均的latency是多少当用户刷新timeline  
根据这些性能指标，我们就可以知道是否应该调整replication，LB或者cache

### Extended Requirements
*How do we serve feeds*  
发送的推特的多少和用户follows的数据有关，也可以pre-generate feed来提高性能，具体请看Ranking and timeline generation在Designing Instagram部分

*Retweet*   
对于已经发送的Tweet Object，没必要存放所有实体，只需要存放ID就行

*Trending Topics*  
我们可以cache最近N秒内最流行的Hashtags或者search queries，保持更新状态每M秒。我们可以rank这些趋势话题，根据他们的frequency或者retweet或者likes，也可以根据需求调整权重weight。

*Who to follow? How to give suggestions?*
就是follow的推荐系统，可以用图找出2级或者3级的热门人数（follows多的人）进行推荐。如果可以推荐的人选不多，可以使用Machine Learning（ML）来shuffle和re-prioritize。ML的信号可以是最近某人的followship增加了，或者共同的following，或者共同的location和interest。

*Moments*  
从其他网站获取最近1-2小时的top news，搞清楚相关联的推特，并且进行prioritize和categorize，然后通过ML-supervised learning（专家训练）或者clustering提供Trending topics in Moments

*Search*  
搜索包括indexing，ranking还有retrieval(检索)，一种类似的方法可以参考Design Twitter Search

### 其他
这里有一段tweet的模拟代码，挺有趣，来源[新鲜事系统](https://marian5211.github.io/2018/01/27/%E3%80%90%E4%B9%9D%E7%AB%A0%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1%E3%80%91%E6%96%B0%E9%B2%9C%E4%BA%8B%E7%B3%BB%E7%BB%9F/)
```
import javax.jws.soap.SOAPBinding;
import java.util.*;
public class DesignTwitter {
    class Twitter {
        HashMap<Integer,User> UsersList;
        int tweetTimeStamp = 0;
        class Tweet{
            int id;
            int userId;
            int timeStamp;
            Tweet(int id,int useId,int timeStamp){
                this.id = id;
                this.userId = useId;
                this.timeStamp = timeStamp;
            }
        }
        class User{
            int uid;
            List<Tweet> tweets = new ArrayList<>();
            List<Integer> followers = new ArrayList<>();//被谁关注
            List<Integer> followees = new ArrayList<>();//关注了谁
            User(int uid){
                this.uid = uid;
            }
        }
        /** Initialize your data structure here. */
        public Twitter() {
            UsersList = new HashMap<>();
        }
        /** Compose a new tweet. */
        public void postTweet(int userId, int tweetId) {
            if(!UsersList.containsKey(userId)){
                User user = new User(userId);
                UsersList.put(userId,user);
            }
            Tweet tweet = new Tweet(tweetId,userId,tweetTimeStamp);
            UsersList.get(userId).tweets.add(tweet);
            tweetTimeStamp++;
        }
        /** Retrieve the 10 most recent tweet ids in the user's news feed. Each item in the news feed must be posted by users who the user followed or by the user herself. Tweets must be ordered from most recent to least recent. */
        public List<Integer> getNewsFeed(int userId) {
            List<Integer> feeds = new ArrayList<>();
            if(!UsersList.containsKey(userId)){
                User user = new User(userId);
                UsersList.put(userId,user);
                return feeds;
            }
            Comparator<Tweet> cmp = new Comparator<Tweet>(){
                public int compare(Tweet t1,Tweet t2){
                    return t2.timeStamp - t1.timeStamp;
                }
            };
            Queue<Tweet> heap = new PriorityQueue<>(10,cmp);
            //先处理自己的新鲜事
            List<Tweet> ownTweets = UsersList.get(userId).tweets;
            for(int i = ownTweets.size()-1;i >= ownTweets.size() - 10 && i >= 0;i--){
                heap.add(ownTweets.get(i));
            }
            //处理关注的人的新鲜事
            List<Integer> followees = UsersList.get(userId).followees;
            for(Integer followee : followees){
                List<Tweet> tweets = UsersList.get(followee).tweets;
                for(int i = tweets.size()-1;i >= tweets.size() - 10 && i >= 0;i--){
                    heap.add(tweets.get(i));
                }
            }
            int k = 10;
            while (k > 0 && !heap.isEmpty()){
                feeds.add(heap.poll().id);
                k--;
            }
            return feeds;
        }
        /** Follower follows a followee. If the operation is invalid, it should be a no-op. */
        public void follow(int followerId, int followeeId) {
            //如果没有用户，新建用户
            if(!UsersList.containsKey(followeeId)){
                User user = new User(followeeId);
                UsersList.put(followeeId,user);
            }
            if(!UsersList.containsKey(followerId)){
                User user = new User(followerId);
                UsersList.put(followerId,user);
            }
            if(followeeId == followerId || UsersList.get(followerId).followees.contains(followeeId)){return;}
            //更新用户关注和被关注列表
            UsersList.get(followeeId).followers.add(followerId);
            UsersList.get(followerId).followees.add(followeeId);
        }
        /** Follower unfollows a followee. If the operation is invalid, it should be a no-op. */
        public void unfollow(int followerId, int followeeId) {
            if(followeeId == followerId || !UsersList.containsKey(followeeId) || !UsersList.containsKey(followerId) ||
                    !UsersList.get(followerId).followees.contains(followeeId) || !UsersList.get(followeeId).followers.contains(followerId)){
                return;
            }
            Iterator<Integer> iterator =  UsersList.get(followeeId).followers.iterator();
            while(iterator.hasNext()){
                int i = iterator.next();
                if(i == followerId){
                    iterator.remove();
                }
            }
            Iterator<Integer> iterator2 =  UsersList.get(followerId).followees.iterator();
            while(iterator2.hasNext()){
                int i = iterator2.next();
                if(i == followeeId){
                    iterator2.remove();
                }
            }
        }
    }
    public void test(){
        Twitter obj = new Twitter();
        obj.postTweet(1,5);
        List<Integer> param_2 = obj.getNewsFeed(1);
        obj.follow(1,2);
        obj.postTweet(2,6);
        List<Integer> param_3 = obj.getNewsFeed(1);
        obj.unfollow(1,2);
        List<Integer> param_4 = obj.getNewsFeed(1);
    }
    public static void main(String[] args){
        DesignTwitter test = new DesignTwitter();
        test.test();
    }
}
```

