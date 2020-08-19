其实在算法题中对string进行hash经常会用到，就类似一个sliding windows存放。不过bloom Filter有其他的应用场景。

1)垃圾邮件地址过滤(地址数量很庞大)  
2)爬虫URL地址去重  
3)解决缓存击穿问题  
4)浏览器安全浏览网址提醒  
5)google 分布式数据库Bigtable以及Hbase使用布隆过滤器来查找不存在的行或列，以减少磁盘查找的IO次数  
6)文档存储检索系统也可以采用布隆过滤器来检测先前存储的数据  

二、Bloom Filter的特点：
1>、不存在漏报（False Negative），即某个元素在某个集合中，肯定能报出来。  
2>、可能存在误报（False Positive），即某个元素不在某个集合中，可能也被爆出来。  
3>、确定某个元素是否在某个集合中的代价和总的元素数目无关。  

简单说就是用速度换准确度

## Reference
[bloom Filter布隆过滤器简介](https://www.jianshu.com/p/51e40483911f)  
[海量数据处理算法—Bloom Filter](https://blog.csdn.net/hguisu/article/details/7866173)
