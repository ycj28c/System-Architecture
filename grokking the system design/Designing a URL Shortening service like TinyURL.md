## Designing a URL Shortening service like TinyURL 笔记

### 1.Why do we need URL shortening?
就是让url缩短，容易记忆的大小，甚至自定义。同时也起到隐藏初始url的作用。

### 2.Requirements and Goals of the System
要确保设计的就是面试人想要的，问清楚。

Functional Requirements：  
1.给一个URL，能生成一个短的独一无二的alias  
2.当用户访问link，服务能redirect到其初始link  
3.用户可以选择一个custom的link  
4.这些links在一个固定时间后会失效，用户必须设定过期时间

Non-Functional Requirement：  
1.系统高available，因为一旦宕机，依赖的URL redirections也失效了  
2.URL redirection应该延迟很小  
3.Shortened links不应该被猜到

Extended Requirement：  
1.分析；比如redirection发生次数的分析  
2.最好还能为其他服务提供REST APIs 

### 3.Capacity Estimation and Constraints
系统将是read-heavy的。假设这个比例是100：1读和写。  

Traffic estimates：假设有500M的新URL shortenings每月，根据100：1的读写比例，将会有50B的redirections每月。

What would be Query Per Second（QPS）for our system？
```
# 写
500 million / (30 days * 24 hours * 3600 seconds) = ~200 URLs/s
# 读
100 * 200 URLs/s = 20K/s
```

Storage estimates: 假设存放5年的URL shortening请求，那么
```
500 million * 5 years * 12 months = 30 billion
```
假设每项数据需要500bytes，那么需要total storage：
```
30 billion * 500 bytes = 15 TB
```

Bandwith estimate：因为我们期望200个新URL每秒，所以带宽是
```
# 写
200 * 500 bytes = 100 KB/s
# 读
20K * 500 bytes = ~10 MB/s
```

Memory estimates：假如我们需要cache部分热门URLs，那么需要多少内存？我们以天为单位进行cache，根据80-20法则
```
# 读总共每天
20K * 3600 seconds * 24 hours = ~1.7 billion
# 需要的cache
0.2 * 1.7 billion * 500 bytes = ~170GB
```
实际内存应该小于170GB，因为很多请求是重复的。

### 4.System APIs
可以使用SOAP或者REST APIs，例子
```
createURL(api_dev_key, original_url, custom_alias=None, user_name=None, expire_date=None)
```
api_dev_key(string)：就是注册账户所用的API developer key，可以用来进行throttle。 
original_url(string)：原始URL  
custom_alias(string)：可选项，用户自定义URL  
user_name(string)：可选项，用户名可以用来encoding  
expire_date(string)：可选项，每个shorten URL的过期时间

return(string)：成功时候返回shortened URL，否则返回error code

```
deleteURL(api_dev_key, url_key)
```
成功删除的返回“URL Removed”

How do we detect and prevent abuse？恶意用户可以消耗掉所有的URL keys，所以需要通过api_dev_keyl来限制每个用户的URL创建，还有redirections的次数（在一定周期内，可以看rate limiter那一部分内容）
