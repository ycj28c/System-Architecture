## Designing Youtube or Netflix 笔记

### 确认需求
功能需求：  
1. 用户需要能上传视频
2. 用户需要能够分享和查看视频
3. 用户需要能够根据视频名称搜索
4. 我们服务能够记录视频状态，比如like/dislike，total views等
5. 用户能够增加评论

非功能性需求：  
1. 系统高可靠，视频上传不丢失
2. 系统高可用，一致性可能受影响
3. 用户实时看视频，感受不到lag

### 容量估算和约束
假设1.5billion用户，800million每日活跃用户。加入每个用户每日平均看5个视频，那么每秒 
``` 
800M * 5 / 86400 = 46k videos/sec
```
假设我们的上传观看率是1：200，每个视频有200观看数，那么需要至少上传230视频每秒
```
46K / 200 = 230 videos/sec
```

存储估算：假设每分钟有500小时的视频上传到youtube，平均每分钟的视频需要50mb的空间（需要存放为多种格式），那么总共空间需要
```
500 hours * 60 min * 50MB >= 1500GB/min(25GB/sec)
```
这个估计是在忽略了压缩和复制备份的前提下

带宽估算：每分钟500小时的视频，假设每个视频占用10MB/min带宽，我们需要300G的上传带宽每分钟
```
500 hours * 60 mins * 10MB >= 300GB/min(5GB/sec)
```

### 系统API设计
我们可以用SOAP或者REST APIs来展示服务功能，比如

上传函数：
```
uploadVideo(api_dev_key, video_title, vide_description, tags[], category_id, default_language, 
                        recording_details, video_contents)
```
这个参数可以自己设计，返回值是String，如果接受返回HTTP 202(request accepted)，当视频encode结束就发送email。

搜索函数：
```
searchVideo(api_dev_key, search_query, user_location, maximum_videos_to_return, page_token)
```
参数可以自由发挥，返回值可以是JSON，包含了视频列表，每个视频可以有title，thumbnail，create date等

Stream函数：（跳播）
```
streamVideo(api_dev_key, video_id, offset, codec, resolution)
```
返回的是stream，也就是在某个offset的视频chunk

### 高级设计
这个其实说白就是要画图说明整个流程：  
1. Processing Queue：每个上传的视频都会被发送到queue里面等待被deque和encoding，thumbnail generation和storage
2. Encoder：将视频encode为多重formats
3. Thumbnails generator：生成快照视频用于快速展示
4. Video and Thumbnail storage：将视频和快照视频存储到分布式存储系统
5. User Database：存储用户信息比如name，email，address之类
6. Video metadata storage： 视频元数据库，记录格式化的数据比如title, file path，uploading user, total views等，还有所有的comments

### 数据库设计
主要是metadata storage的设计，可以如下：

Videos表:  
VideoID  
Title  
Description  
Size  
Thumbnail  
Uploader/User  
Total number of likes  
Total number of dislikes  
Total number of views  

Comments表：  
CommentID  
VideoID  
UserID  
Comment  
TimeOfCreation  

User表：
UserId  
Name  
Email 
Address  
Age  
Registration Detail  

### 详细设计
因为服务的读取比重很高，所以需要我们要设计一个能快检索视频系统。预期是200：1读写比例。

Where would videos be stored？视频存储在文件系统比如HDFS和GlusterFS(这个没有深入了解过，TODO)

How should we efficiently manage read traffic？ 我们要分离读和写。因为我们会有视频的多个拷贝，所以我们可以将读写分布到不同的服务。对于metadata，可以使用master-slave设置，这样writes就会先到master然后应用到全部的slaves去。这样的设置可能会导致视频过期问题，比如，当一个新视频加入了，metadata先插入了master，在应用到slave之前slave都是无法看到该视频的； 因此slave不会返回realtime数据。 但是这个staleness对于这个视频系统是可以接受的，因为这个同步的时间会很短。

Where would thumbnails be stored？ 系统将生成比元视频数量更多的thumbnails。 如果我们假设每个视频有5个thumbnails，我们就需要一个非常高效的存储系统来处理大量的读取流量。 主要有两个考虑点：  



