MySQL和NoSQL型数据库的Replica比较

SQL   
“自带” 的 Replica 方式是 Master Slave，通过WAL实现   
“手动” 的 Replica 方式也可以在 Consistent Hashing 环上顺时针存三份   

NoSQL   
“自带” 的 Replica 方式就是 Consistent Hashing 环上顺时针存三份   
“手动” 的 Replica 方式：就不需要手动了，NoSQL就是在 Sharding 和 Replica 上帮你偷懒用的！   

Replica通常的做法是：一式三份（重要的事情说三遍）  
Replica同时还能分摊读请求  
