忘了可以回顾一下
TODO

### HTTP
基于TCP/IP，默认端口80，无连接无状态。  
基本方法GET（查），POST（增），PUT（该），DELETE（删）  

规范把 HTTP 请求分为三个部分:状态行、请求头、消息主体。类似于下面这样：
```
<method> <request-URL> <version>
<headers>

<entity-body>
```
GET请求的例子
```
 GET /books/?sex=man&name=Professional HTTP/1.1
 Host: www.example.com
 User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
 Gecko/20050225 Firefox/1.0.1
 Connection: Keep-Alive
```

响应reponse包括状态行，响应头（Response Header），响应正文  
常见状态码包括：  
```
200 OK 客户端请求成功
301 Moved Permanently 请求永久重定向
302 Moved Temporarily 请求临时重定向
304 Not Modified 文件未修改，可以直接使用缓存的文件。
400 Bad Request 由于客户端请求有语法错误，不能被服务器所理解。
401 Unauthorized 请求未经授权。这个状态代码必须和WWW-Authenticate报头域一起使用
403 Forbidden 服务器收到请求，但是拒绝提供服务。服务器通常会在响应正文中给出不提供服务的原因
404 Not Found 请求的资源不存在，例如，输入了错误的URL
500 Internal Server Error 服务器发生不可预期的错误，导致无法完成客户端的请求。
503 Service Unavailable 服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常。
```
HTTP响应的例子
```
HTTP/1.1 200 OK

Server:Apache Tomcat/5.0.12
Date:Mon,6Oct2003 13:23:42 GMT
Content-Length:112

<html>...
```

HTTP Keep-Alive（长连接） 简单说就是保持当前的TCP连接，避免了重新建立连接。

### HTTPS SSL/TLS
HTTPS 即 HTTP over TLS，是一种在加密信道进行 HTTP 内容传输的协议。  
有3个角色：  
CA： 数字证书认证机构（Certificate Authority，简称CA）  
服务端： 一些web服务器之类  
客户端： 就是我们浏览器使用者  

简略过程：  
1.客户端根据自己信任的CA列表，验证服务器端证书可信。  
2.客户端然后生成一串随机数，用服务端的公钥加密它，发给服务端。  
3.服务端使用这个随机数生成对称主密钥。  
4.两者用协议号的对称密钥对整个TLS会话进行加密，传输HTTP内容。  

CA的验证涉及到Root CA，是操作系统和浏览器自带的。

### How is virtual address translated to physical address
TLB，MMU具体如何工作的?

### TCP slow start
TCP(Transmission Control Protocol)，避免网络拥塞算法。拥塞窗口在每接收到一个确认包时增加，每个RTT内成倍增加，当然实际上并不完全是指数增长，因为接收方会延迟发送确认，通常是每接收两个分段则发送一次确认包。发送速率随着慢启动的进行而增加，直到遇到出现丢失、达到慢启动阈值（ssthresh）、或者接收方的接收窗口进行限制。如果发生丢失，则TCP推断网络出现了拥塞，会试图采取措施来降低网络负载。

慢开始算法的思路就是，不要一开始就发送大量的数据，先探测一下网络的拥塞程度，也就是说由小到大逐渐增加拥塞窗口的大小

### TCP Fast Retransmit 快重传
首先对于接收方来说，如果接收方收到一个失序的报文段，就立即回送一个 ACK 给发送方，而不是等待发送延时的 ACK。  
所谓失序的报文是指，用户没有按照顺序收到 TCP 报文段，比如接收方收到了报文 M1, M2, M4，那么 M4 就称为失序 报文。  
快重传算法规定，如果发送方一连收到 3 个重复的确认，就应当立即传送对方未收到的报文 M3，而不必等待 M3 的重传计时器到期。

### TCP fast recovery 快恢复
ssthresh初始16

一旦出现超时重传，TCP 就会把慢启动门限 ssthresh 的值设置为 cwnd 值的一半，同时 cwnd 设置成 1. 但是快恢复算法不这样做。  
一旦出现超时重传，或者收到第三个重复的 ack 时（快重传），TCP 会把慢启动门限 ssthresh 的值设置为 cwnd 值的一半，同时 cwnd = ssthresh （在有些版本中，会让 cwnd = ssthresh + 3）。  

### TCP vs Udp
UDP： user data protocol  
* UDP 缺乏可靠性。UDP 本身不提供确认，序列号，超时重传等机制。UDP 数据报可能在网络中被复制，被重新排序。即 UDP 不保证数据报会到达其最终目的地，也不保证各个数据报的先后顺序，也不保证每个数据报只到达一次  
* UDP 数据报是有长度的。每个 UDP 数据报都有长度，如果一个数据报正确地到达目的地，那么该数据报的长度将随数据一起传递给接收方。而 TCP 是一个字节流协议，没有任何（协议上的）记录边界。  
* UDP 是无连接的。UDP 客户和服务器之前不必存在长期的关系。UDP 发送数据报之前也不需要经过握手创建连接的过程。  
* UDP 支持多播和广播。  

### Three-way Handshake
是指建立一个 TCP 连接时，需要客户端和服务器总共发送3个包。  
三次握手的目的是连接服务器指定端口，建立 TCP 连接，并同步连接双方的序列号和确认号，交换 TCP 窗口大小信息。在 socket 编程中，客户端执行 connect() 时。将触发三次握手。  
1.客户端发送求请求  
2.服务端接收到并确 --- 完成第一次握手  
3.服务端发送ACK  
4.客户端接送到ACK --- 完成第二次握手  
5.客户端发送确认包ACK  
6.握手结束，连接建立完毕，可以开始发数据 ---完成第三次握手  

还有4次回收，在服务器发送ACK以后还要发送一个Fin用来说我的数据发完了。

TCP Triple Ack?

### Traffic control & congestion control
发送方维持一个叫做拥塞窗口cwnd（congestion window）的状态变量。拥塞窗口的大小取决于网络的拥塞程度，并且动态地在变化。发送方让自己的发送窗口等于拥塞窗口，另外考虑到接受方的接收能力，发送窗口可能小于拥塞窗口。

### TCP sliding window, window size是如何确定的
为了防止cwnd增长过大引起网络拥塞，还需设置一个慢开始门限ssthresh状态变量。ssthresh的用法如下：

1.当cwnd<ssthresh时，使用慢开始算法。  
2.当cwnd>ssthresh时，改用拥塞避免算法。  
3.当cwnd=ssthresh时，慢开始与拥塞避免算法任意。  

拥塞避免算法让拥塞窗口缓慢增长，即每经过一个往返时间RTT就把发送方的拥塞窗口cwnd加1，而不是加倍。这样拥塞窗口按线性规律缓慢增长。

发送方的窗口上限值=min{接收方的窗口rwnd,拥塞窗口cwnd}

### AIMD的机制，以及MIMD的比较
和性增长/乘性降低（英语：additive-increase/multiplicative-decrease、AIMD）算法是一个反馈控制算法，最广为人知的用途是在TCP拥塞控制。

???

### OSI 7 layer
OSI（Open System Interconnect），即开放式系统互联。 一般都叫OSI参考模型，是ISO（国际标准化组织）组织在1985年研究的网络互连模型。

* 7应用层（Application）： 各种应用程序协议，HTTP，FTP，SMTP之类  
* 6表示层（Presentation）： 信息的语法语义和关联，比如加密解密，压缩解压缩，Telnet，SNMP之类  
* 5会话层（Session）： 不同机器上用户之间建立和管理会话，SMTP，DNS之类  
* 4传输层（Transport）： 接受网络层数据，并交给会话层，保证数据有效到达，TCP，UDP之类  
* 3网络层（Network）： 控制逻辑编码，路由选择等，IP，ICMP，ARP之类  
* 2数据链路层（Data Link）： 物理寻址，将比特流转换问逻辑传输线路，Ethernet，PPP之类  
* 1物理层（Physical）： 机械电子通信信道上原始比特流传输，IEEE 802.1A等协议  

### TCP/IP四层概念模型 
应用层，传输层，网络层，数据链路层  

TCP属于传输层，提供可靠的字节流服务。所谓的字节流，其实就类似于信息切割。比如你是一个卖自行车的，你要去送货。安装好的自行车，太过庞大，又不稳定，容易损伤。不如直接把自行车拆开来，每个零件上都贴上收货人的姓名。最后送到后按照把属于同一个人的自行车再组装起来，这个拆解、运输、拼装的过程其实就是TCP字节流的过程。

IP协议的作用在于把各种数据包准确无误的传递给对方，其中两个重要的条件是IP地址和MAC地址（Media Access Control Address），举一个现实生活中的例子，IP地址就如同是我们居住小区的地址，而MAC地址就是我们住的那栋楼那个房间那个人。

### 命令行
netstat -s命令： 显示所有网络连接和端口

TCPdump： 抓取端口数据包  

### Reference
1.[TCP协议](https://hit-alibaba.github.io/interview/basic/network/TCP.html)  
2.[OSI七层模型和TCP/IP四层模型](https://blog.csdn.net/xuedan1992/article/details/80958522?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=1328680.45225.16163518970289791&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)  
3.[tcp慢开始（Slow-Start）、拥塞避免（Congestion Avoidance）、快重传（Fast Retransmit）和快恢复（Fast Recovery）](https://blog.csdn.net/zengaixiao/article/details/77171951)  