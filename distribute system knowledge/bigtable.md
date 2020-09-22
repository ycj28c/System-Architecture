BigTable是Google的一个论文，Google BigTable是一个分布式，结构化数据的存储系统，它用来存储海量数据。该系统用来满足“大数据量、高吞吐量、快速响应”等不同应用场景下的存储需求。

具体的实现比如HBase，是列式数据库，本质上是一个超级大的map（key value存储）。具体的就是使用sortedMap<String, sortedMap<String, String>>来存储每行数据，这样数据可以方便的添加新属性，也可以方便的按照列hash，实现分布式。

这类nosql的用法都一样，就是高吞吐，可以并行做一项操作（也就是map），然后合并结果（也就是reduce）。和多线程是一个道理，不过将垂直的机器power变成了水平的机器power，这样通过加机器就行实现吞吐量的提升。

一个场景就是存放历史网页，一个url一行数据，但是一个属性比如title可以存放多个历史版本。

Google官方的bigtable，[Cloud Bigtable](https://cloud.google.com/bigtable），其实也和Hbase一样，都是按照白皮书写的。另外，Hbase是一个产品，而Google Bigtable是直接提供服务了，就不用考虑服务器啥的了，直接用就行。

具体参考指南https://cloud.google.com/bigtable/docs/quickstarts

Bigtable将数据统统看成无意义的字节串，客户端需要将结构化和非结构化数据串行化再存入Bigtable。所以在youtube的设计提到用Bigtable存储缩略图，因为是列式的，所以缩略图存储位置接近。

[HBase实操 | 如何使用HBase存储图片](https://yq.aliyun.com/articles/670092)
[理解BigTable](https://lianhaimiao.github.io/2018/03/19/%E7%90%86%E8%A7%A3BigTable/)