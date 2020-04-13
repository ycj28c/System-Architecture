具体的需要做Hello World来感受 TODO
[10 分钟教程](https://aws.amazon.com/cn/getting-started/tutorials/)

## Amazon EC2
全名為 Amazon Elastic Compute Cloud，就是一个Server虚拟主机啦。

## Amazon VPC
全名為 Amazon Virtual Private Cloud, 直譯翻譯的話接近Virtual(虛擬) Private(私有) Cloud(雲)。 就是将多台主机连接起来像局域网一样使用，对外可以就提供一台EC2的服务器端口。

## Amazon S3
全名為 Simple Storage Service，就是大型网络存储，通过API访问文件，可以设置相应的权限之类。

## Amazon RDS
全名為Amazon Relational Database Service，就是网络关系型数据库，当做MySQL来用吧。RDS服務主要有6個資料庫種類可以選擇，除了常見的MySQL、PostgreSQL、MariaDB，也有大型商業公司常用的Oracle與微軟旗下的MSSQL。當然Amazon也很喜歡用自己的規格綁架使用者，所以製作出了Amazon Aurora這個與MySQL共容的資料庫。

## Amazon DynamoDB
Amazon最具代表性的NoSQL服务。

## Amazon SES
全名為 Amazon Simple Email Service，主要是寄送Email信的服務。 這個服務如果從EC2呼叫是免費的，每個月有62,000封免費，每天最高可以寄送五萬封Email。 据说不好用，没有深入研究。

## Amazon SNS 
全名為 Amazon Simple Notification Service，這個服務也是常存在各種服務之中。主要去監聽系統活動，並且發送通知訊息。 這服務有兩個比較明顯的概念，第一個就是通知使用者，比如发送Email通知，另外一個是驅動其他程式，比如说自动化触发其他AWS服务。

## Amazon CloudFront
这个算是CDN版本的S3，将S3的文档自定义在全球节点分享。

## AWS IAM
IAM除了可以限制使用者，還可以管理使用者權限，比較特別的是他的使用者權限管理得很細，讀寫可以分離。 每個AWS的服務的權限是分開的，也有User使用者與Role角色的設定方式，也可以單獨廢棄掉某個使用者或是某個授權的Key(如果不小心洩漏的話)。總之非常的詳盡。

## Amazon Cognito
也是用来做身份验证，主要是针对mobile的

## Elastic Load Balancer / Auto Scaling
現在的分散式架構為了解決時間週期性的大小流量變化，使用了叫做Auto Scaling的技術。簡單的來說就是在流量大的時候多開幾台機器，流量小的時候關掉多餘的機器。   
但是在實現Auto Scaling之前會先碰到要如何把流量平均分配給多台機器這個問題，那就是Load Balancer。 方法就是在所有的機器前面在建立一台機器先接收所有的流量，但是不處理Reqeust，而是分發給其他的機器Server處理，有點類似總機的概念。一般如果要自己建立的話，可以使用Nginx再加上一些設定與調整。但是如果你用的是AWS Load Balancer的話，基本上就是隨開即用。AWS的Load Banlancer還有一個特別的服務，就是加上HTTPS不用錢。

## AWS ElasticCache
相当于Redis。 In memory cache會使用系統的memory，會擠壓到Application能使用的資源，又因為Auto Scaling的關係，所以每台Cache都有可能會重複或是不完整，因此拉出來成為獨立的服務是一個很自然的解法，可以自己选择Redis或是Memcached。

## AWS Certificate Manager
就是提供HTTPS服务，只要你使用的服務是Elastic Load Balancing、Amazon CloudFront 及 Amazon API Gateway這些整合型的服務，那你都可以免費使用AWS 提供的SSL 憑證，而且還是類似Wildcard形式的，連子網域都包含進去。

## AWS Lambda
serverless的运行脚本文件，支持多种Node.js，Ruby，Java，Go等多种脚本语言，通过HTTP直接运行。相当于在线跑code吧，用 Amazon Lambda 可搭建一套无 EC2 的 Web / App 架构（无服务器架构）：  
核心计算：Amazon Lambda（编程模型 node.js、java、python）  
数据库存储：Amazon DynamoDB  
域名管理：Amazon Route 53  
前端 Web 资源存储：Amazon S3  
前后端 REST：Amazon API Gateway  
网络加速：Amazon CloudFront - CDN  
身份验证：Amazon IAM、Amazon Cognito

[带您玩转Lambda，轻松构建Serverless后台！](https://aws.amazon.com/cn/blogs/china/lambda-serverless/)

## Amazon API Gateway
Amazon API Gateway 是一种完全托管的服务，可以帮助开发者轻松创建、发布、维护、监控和保护任意规模的 API。只需在 AWS 管理控制台中点击几下，您便可以创建可充当应用程序“前门”的 API，从后端服务访问数据、业务逻辑或功能，例如基于 Amazon Elastic Compute Cloud (Amazon EC2) 运行的工作负载、基于 AWS Lambda 运行的代码或任意 Web 应用。 就是用来发布API服务的，开发不需要管权限，监控，负载等等问题了。TODO

## Reference
[為什麼熟悉Amazon AWS總是高薪工程師的加分條件? 深入淺出的AWS服務教學介紹。](https://progressbar.tw/posts/224)  