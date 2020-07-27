## Load Balancing 笔记

Load Balance的好处：  
1.帮助分流traffic，提高avalibility和responsiveness  
2.帮助宕机时候自动转移traffic，放置single point of failure  

Load Balacing算法：  
How does the load balancer choose the backend server？  
首先是心跳检查保证服务可用，然后应用预先设置的算法来进行balance。Health Checks包含了多种算法：  
1.Least Connection Method - 分配流量到最少的连接数服务器  
2.Least Response Time Method - 分配流量到最少连接数以及最低平均响应时间服务器  
3.Least Bandwidth Method - 通过测量网络Mbps流量来分配流量  
4.Round Robin Method - 按顺序将流量发给下一个服务器，到end了再从第1个服务器开始，如此循环。适用于每个服务器配置差不多，而且连接非固定的情况  
5.Weighted Round Robin Method - 同Round Robin，不过weight高会分配更多traffic  
6.IP Hash - 通过用hash划分client IP来分配到不同服务器

Redundant Load Balancers：  
load balance本身也会宕机，所以也需要backup，可以主备或者cluster。