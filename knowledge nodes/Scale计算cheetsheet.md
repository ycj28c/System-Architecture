
## 基础数值
QPS（Query Per Second） ：服务器每秒可以执行的查询次数  
TPS（Transaction Per Second） ：服务器每秒处理的事务数  

通常的峰值QPS是常规的5倍  
如果考虑未来高速增长的设计，要至少提高10倍QPS  

大写和小写的是8倍的区别： 
B=Byte  
b=bit  
1 Kb = 1024 bit  
1 KB = 1024 Byte  
1 Mb = 1024 Kb  
1 MB = 1024 KB  
1 Byte = 8 bit  
1 MB = 8Mb  
1 Mb = 0.125 MB  

## 存储空间数值
1个英文字母占用多少空间和编码有关。 

1个字节Byte，就是8bit

ASCII码: 1个英文1个Byte  
UTF-8: 1个英文1个Byte  
Unicode: 1个英文两个Byte  

比如地址长度100个英文，就是100Byte，然后1M的用户就是100 * 1M = 100MByte = 100MB

## Linux服务器数值
Socket: TCP，单机最多60,000左右pool，不过考虑高内存和带宽，支持3000就不错了

## 网络数值
Webservice: 大概500 - 1000 QPS  
路由带宽：100M  

通常，运营商说的1M宽带的M是指Mb/s，也就是Mbps，换算一下的话，1M宽带下载速度也就是125KB/s，再去掉损耗的话就是120KB/s左右。以此类推，10M宽带的最快下载速度是1.25MB/s，100M的宽带最快下载速度是12.5MB/s。

## 中间件数值
Nginx: 最高300,000 QPS  
Tomcat：20,000 QPS  

## 数据库数值
Mysql: 1000 QPS (max 4k)    
NoSQL Database(比如Cassandra): 10K QPS  
Redis: 80,000 QPS (max 100,000)   
Memcached Cache: 1,000,000 QPS  
Elastic search: 搜索QPS 1700 (2000)  

## Queue性能
Rocket MQ: 10,000 QPS  
Kafka: 单机TPS写入1M  

## Reference
1.[一文看懂 Mbps、Mb/s、MB/s 有什么区别](https://www.ithome.com/0/437/491.htm)  
2.[bit、byte、位、字节、汉字的关系](https://blog.csdn.net/bigapple88/article/details/5601295)  
