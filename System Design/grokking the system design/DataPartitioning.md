## Data Partitioning 笔记

分区就是把大数据库砍成小块的。 

## Partitioning Methods

1.Horizontal Partitioning 水平分区：  
就是将不同的row行数据放到不同的tables中。比如一个存放地址的table，我们把ZIP小于10000的放到tableA，其他的放到tableB。这个也叫range based partitioning，而水平分区也叫数据分片。  
主要问题是分片不但会导致load不均衡，比如ZIP小于10000的流量明显多于tableB的就不均衡了。

2.Vertical Partitioning 垂直分区：  
就是将数据按照类型或者feature来分配服务器。比如，Instagram有用户数据，图片数据，和follow数据，我们可以将用户数据放在DB1，朋友数据放在DB2，图片数据放在DB3。  
垂直分区很直接，但是主要问题在于当应用的规模增加，还需要做相应的二次分区，单个服务器很难处理百亿级别的图片数据。

3.Directory Based Partitioning 目录分区：  
一种loosely coupled松散耦合的方法就是创建lookup服务，并将之进行抽象，脱离物理数据库访问。要找某个特定数据，就查询目录服务器，让目录服务器来mapping每个key和对应的DB服务器。这种松耦合方法意味着我们可以随时添加服务器，和随时修改我们的分区而不影响应用。

## Partitioning Criteria 分区标准

1.Key or Hash-based partitioning 键值分区：  
顾名思义，用hash来分区，比如ID%10。相关理论就是Consistent Hashing

2.List partitioning 列表分区
每个分区分配一列值，当插入新值就看哪个分区包含那个key，就插哪里。比如北美国家包含美国，加拿大。

3.Round-robin partitioning  
就是循环啊，同样可以用ID%10为例，1和11都进入分区1

4.Composite Partitioning  
就是结合以上的几种分区策略，比如先List Partitioning然后再Hash Partitioning

## Common Problmens of Data Partitioning
分区肯定会造成问题，因为某些操作要多表或者多行，但是这些表或者行被分配到了不同的分区。

1.Joins and Denormalization：相交和非规范化  
比如在数据库跑一个join很容易，但是如果表在不同的机器上就不可行了。通常的做法是denormalize数据库，让join的操作现在单表完成，不过，这样数据库也要处理数据一致性问题了。

2.Referential integrity：参照完整性  
在分区数据库完成一个foreign key的操作也将变得很困难。很多RDBMS数据库不支持分区外键，所以referential的操作需要再应用code层面完整。

3.Rebalancing：重新平衡  
根据实际需要分区会需要调整，比如非规范化ZiP code无法进行常规分区，比如某些分区的流量特别大。 这种情况，要么增加更多的DB分区，要么重新组织当前分区，这么多通常都会照成downtime。  
使用directory based partitioning让rebalance更简单，但是也会增加系统复杂性。





