## Proxies 笔记
Proxy就是client和后端服务器的中间层了，可以是硬件或者软件。

通常，proxies用来filter requests，log requests，增加删除header，加密解密压缩，cache等等。这个解释感觉任何防火墙，路由之类都算是proxies。

### Proxy Server Types  
proxy可以在本地服务器，也可以在用户和远程服务器之间，一些著名的代理：

Open Proxy  
一个Open proxy就是一台代理服务器，可以被任何Internet用户访问。通常一个代理服务只允许在特定网络组内来存储和转发，比如DNS或者web pages来减少和控制组内的带宽。但是如果使用了open proxy，任何用户都能用此转发。有两种open proxy：  
1.Anonymous Proxy - 这个代理显示了自身是服务器，但不公开初始的IP。虽然这些代理服务器很容易被发现，但是对某些用户可以隐藏他们的IP  
2.Transparent Proxy - 这类代理服务器也显示自身，并且通过支持HTTP headers，可以传递最初的IP。这类服务器好处是可以cache网站。

Reverse Proxy  
服务器根据客户端的请求，从其关系的一组或多组后端服务器（如Web服务器）上获取资源，然后再将这些资源返回给客户端，客户端只会得知反向代理的IP地址，而不知道在代理服务器后面的服务器集群的存在。说白了就是类似Load balancer，tomcat之类的存在。

### 自我总结
总之这章没讲什么有用的，就介绍个代理的概念，而且一点也不具体。