## Caching 笔记
Cache嘛，大家都懂。用快速的内存存放可以快速读取的内容。

### Application server cache
应用服务器的cache通常就是存放内存或者本地硬盘(比网络硬盘快)，不命中就读硬盘。  
在分布式环境下，会有多台应用服务器，每天服务器可以有自己的cache。但如果LB是随机读取节点的，会增加cache的miss rate。解决的方法是使用global cache或者使用distribute cache。

### Content Distribution Network(CDN)
CDN主要针对于大文件类型的cache，比如大量的图片和视频等。因为CDN的工作机制也是先找本地，如果没有则从后台搜索，机制和cache是一样的。  
注意如果系统没有那么大，也没有必要使用CDN，可以把静态文件放在subdomain，比如(static.yourservice.com)，使用轻量级HTTP server比如Nginx提供专门的文件服务。

### Cache Invalidation 
Cache的主要问题就是数据不一致，比如数据库更新了，但是Cache没有更新。解决方法就是invalidation，有3种场景：  

Write-through cache：  
就是写的时候同时写入数据库和cache，这样直接可以获得高速读，而且数据已经持久化不会丢失。缺点是latency，因为要写操作要写两次。

Write-around cache：  
就是写的时候只写入数据库(估计还需要invalid cache)，好处是写的速度快一点，缺点是会导致cache miss，在读的时候会有high latency。  

Write-back cache：  
就是写的时候只写入cache，然后在特定条件或者固定间隔后才写入数据库。好处是速度快，用户读取也快，缺点是有数据丢失的风险，特别是断电之类的情况。

### Cache eviction policies
常见的cache替换方案：  
1.First In First Out(FIFO):先进先出  
2.Last In First Out(LIFO):先进后出  
3.Least Recently Used(LRU):最早使用的先出  
4.Most Recently Used(MRU):最晚使用的先出    
5.Least Frequently Used(LFU):使用最少的先出  
6.Random Replacement(RR):随机出

### 自我总结
这章讲的比较粗略，可能分布式情况下的cache还需要自己在深入研究下