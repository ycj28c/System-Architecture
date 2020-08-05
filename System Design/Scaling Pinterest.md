## 观看笔记
https://www.infoq.com/presentations/pinterest/

## pinterest的演化
在从小traffic到大traffic的过程中，尝试了很多不同的技术，但是太复杂的结构反而容易错误。

后来改成简单的结构：
1.Amazon EC2/S3 reliablility，reporting，很好的DB，load balancing，一个Engineer就能搞定大部分工作。缺点是比较少，每个box都是一样的，想自定义某个数据库多点内存都不行。不过因为都一样，也不容易出问题。

2.Why MySQL？很成熟，不丢数据，大家都会用。响应时间是线程增长的，可以预期今后几周的表现。社区支持好，免费。

3.Why Memcache？结构简单，就是key，value结构，不容易出错，也免费

4.Why Redis？结构简单，key value store，可以persistence和replication，可以快速回复，性能好，免费。

### Clustering vs Sharding的问题  
clustering通常是指nosql，任何数据的分数都是自动分配，数据可以移动到其他节点，我们不知道数据在哪个节点。每个node不知道其他node有啥东西，靠上级的manager进行管理。而Sharding的刚好相反。

### Why Clustering？  
比如Cassandra，Membase，HBase，设置很简单，AWS上点一个按键就自动设置好了。增加节点直接加上就完了，High availability，也load balance，no single point of failure。全部自动化了，都不用engineer了。

### Why not clustering？  
比较困难升级，比如shutdown this，turn on that part，很令人担心，因为不知道是怎么工作的。的确没有single point of failure，但是有Big one，因为cluster alogrithm存在于每个node，所以一旦出bug了，所有的node都宕机了。

### Cluster Manager  
pinterest的观点是scary。  
1）Data rebalance breaks，当增加一个新节点box，全部分布自动完成了，但是突然宕机了，每人知道怎么解决，因为都是自动的。  
2）Data corruption acorss all nodes，只要有1个bug，全部node就出问题了，然后就丢数据了  
3）Improper balancing that cannot be fixed（easily），cluster太聪明，比如有10个node，但是机制自动把数据balance到了一个node上。  
4）Data authority failure  
比如有primary和secondary，通常工作不错，比如负载是80%，20%，但某一次我们bring up secondary，需要off primary，然后忽然secondary说自己是primary，而primary认为自己是secondary，所以primary开始drop数据到20%，而secondary要上到80%，所以丢失了20%的数据之类，会比较头疼。

### Why Sharding？  
1）可以split database来增加capacity  
2）还是可以distribute到不同的数据中心  
3）High availbility  
4）Load balancing  
5）Alogrithm for placing data is very simple  
6）ID generation is simplistic

## Pinterest的shard以及细节

### When to shard？  
如果有很多数据，比如几个TB，那么应该早点shard，越晚就会让transition更难。我们删除了大多数join，在之间使用cache。  
每次挑选最大的table，分区（shard），如果shard还不够，那就继续shard。

### pinterest的数据布局变化：
这是基于读远大于写的情况  
1DB + Foreign Keys + Joins  -->  
1DB + Denormalized + Cache 就是解耦表的联系 -->   
1DB + Read slaves(也就是读写分离了） + cache -->  
Several functionally shared DBs（开始shard大表） + Read slaves + Cache -->  
ID sharded DBs（通过id的范围来shard了） + Backup slaves（slave是用来提供备份的） + Cache（cache存在于各个表的读取之间，提高性能）

### Watch out for...  
1）cannot perform most joins要解耦了，不能太多join  
2）No transaction capabilities 因为是解耦的，所以写入某个数据库可能是会失败的，所以transaction是丢失了  
3）Extra effort to maintain unique constraints 某些id需要全局unique比如额外管理比如userid啊，pinId啊之类，比如新user，先用id generate生成一个，确保是唯一的。在走shard的流程。    
4）Schema changes requires more planning 这个很明显  
5）Reports require running same query on all shards，比如要找到最近的用户activities，要么需要在每一个shard上执行该query，虽然也比较麻烦，但是也是可能的。  

### How we sharded  
我们project估计了下个5年的可能规模，然后pre shard规划大小。when project is out there, we out there.  
1）shared server topology：最初是8个物理服务器，每个服务器有512个DBs（第一台服务器00001-00512，第2台000513-001024 etc），每个DB上包含了所有schema结构，512个db就是预估的5年规模。  
2）High Available，就是使用multi master replication，每个区域有一个backup，比如第一台机器是0-512的挂的，就上那个replication的机器。会有downtime，视频说1小时。  
3）Increased load on DB？比如之前8个服务器，每个机器512个DB，分配到16个服务器，每个服务器256个DB，移动过程中会有downtime，需要设置好程序的mapping。deploy好了replicate之后切换一下（大概几分钟）。

### ID Structure： 
使用ID range进行shard，这样还方便cache。有很多id generate的技术，比如twitter的snowflake。

new user会随机分配到不同的shards？所有该用户的数据也都存放在那里。这样会比较方便访问和快速返回结果。（碰到热门用户就是拆shard）  
  
pinterest使用了64bits的ID，包括ShardID，Type，LocalID  
1）ShareID（约17bit）表示which shard，ID足够65536个shards，目前只用了头4096个。如果之后实在不够需要扩展，就开辟一个新的range，新注册的就直接使用新的range就行。    
2）Type（约4bit）表示类型，比如1表示user，2表示是pin，boards是3之类。    
3）LocalID表示position in table？估计就是某个table的id吧，是自动增长的。  

这样能用好几年，至于好几年以后？再想呗。。总觉得都是笨办法。

### Lookup Structure：  
每个shard都有这样一个config的文件，让服务知道去哪里找shard。比如： 
``` 
{"sharddb001a:(1,512),
 "sharddb002a:(513, 1024),
...
```

### Objects and Mappings  
1）Object tables，  
使用LocalID进行mapping，LocalId就相当于key了，使用的存储就是primay key：LocalID，value：（Object）MySQL bolb（这个存在的可以是JSON，后来pinterest移动到了Serilized thrif），这个和noSQL的挺像了。

2）Mapping tables（就是关系表，比如user has boards，pin has likes）  
FullID -> FullID (+ timestamp用来区分version？)  
这样就可以当做join来用了，需要新的应用logic就增加新的关系table，比如user_watch_pin，这样立即就能投入使用。不需要了就暂停这个index的pop，并删除表。

Naming schema is noum_verb_noun（比如user_has_boards，pin_has_likes），所有pinterest的table都是noum，这样比较容易理解。

3）Queries are PK or index lookups（no joins），也就是LocalID是PK，这样解耦，也便于使用cache  

4）Data DOES NOT MOVE  
准确的说是不移动shard，但是物理数据库或者服务器是可以移动的。

5）all tables exist on all shards  

6）No schema changes required（index = new table）   
因为使用的是和NoSQL类似的MySQL bolb存放，所以加一个属性也是一样方便，应用程序的code可以自动识别和添加（这里还需要一个version check的功能）。

有些情况需要更新数据库，是需要downtime的，那么现在replicate中把数据做好，然后替换上去。
  
### Loading a page：  
一个人的pinterest可能包含人物信息，pins，like等等信息，我们可能需要以下多个query来找出数据：
```
select body from user where id = <local user_id>
select board_id from user_has_boards where user_id =<user_id>
select body from boards where id in (<board_ids>)
select pin_id from board_has_pins where board_id =<board_id>
select body from pins where id in (pin_ids)
```
query很多，要用很多query才能得到需要的一个页面数据，但是因为几乎所有的call都是有cache的，所以速度不慢。

### Scripting：  
当有新功能的时候，需要同步更新old shard和new shard，当数据量大的时候，实际可能花费很久，但自信数据是正确的时候，就替换掉old shard。但是过程要很小心，可能会有数据丢失，总是会出现一系列的问题运行大量数据的时候。这里pinterest推荐了一个Pyres来进行queue？替换了rabbitQueue。

### In The Works：  
1）因为太多connection，需要需要分配更多的memory用来支持connection，避免swap。  
2）所有的业务是service based architecture，例如user_like_pind的设计，这样可以有效的isolate of functionality，方便team合作，同时也能isolation of access，保证了安全，这个team不会干扰另外的team。

## 提的几个问题
1.怎么开发，每个develop有没有sandbox？  
没有，现在每个人都有权限访问所有，呵呵

2.看到说使用了solr来搜索，有没有scale的问题?  
开始使用elastic search，这是2012的视频，现在已经各种流行es了

3.关于auto increase的security的考虑？  
因为现在所有pinterest的东西都是public的，所以不用安全验证，谁都可以访问谁