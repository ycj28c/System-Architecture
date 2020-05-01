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



