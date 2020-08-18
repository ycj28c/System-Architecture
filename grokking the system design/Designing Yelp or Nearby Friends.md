## Designing Yelp or Nearby Friends 笔记

设计一个类似Yelp的服务，可以找附近的餐厅，电影院，mall等。

### 1.Why Yelp or Proximity Server?(接近服务)
Proximity servers就是用来搜索最近的场地和活动的。

### 2.Requirements and Goals of the System
What do we wish to achieve from a Yelp like service? 我们的服务存储了不同地点信息，通过query，可以返回用户附近的地点。

Functional Requirements：  
1.用户可以增删改地点  
2.给定用户的地点（longitude/latitude），用户可以搜索半径内地点  
3.用户可以写feedback和review，可以是图片，text或者评分

Non-functional Requirements：  
1.用户可以实时搜索，很小的延迟  
2.我们服务是read heavy，相对增加新地点有更多的搜索请求

### 3.Scale Estimation
假设有500M的地点和100K的query每秒（QPS），假设每年增长20%的新地点。

### 4.Database Schema
每个地点包含以下属性：  
1.LocationID（8 bytes）  
4个bytes = 32bits = 2^16 = 4,294,967,296 可以覆盖500M了，但是考虑到未来增长，使用8byte作为LocationID。  
2.Name（256 bytes）  
3.Latitude（8 bytes）  
4.Longitude（8 bytes）  
5.Description（512 bytes）  
6.Category（1 byte）比如是咖啡厅，餐厅，电影院等

一个地点总数据大小：8+256+8+8+512+1>=793 bytes  

我们还需要存放reviews，photos，ratings等信息，这些用另外的一个table存放：  
1.LocationID（8 bytes）  
2.ReviewID（4 bytes）一个地点少于2^32reviews，肯定够了  
3.ReviewText（512 bytes）  
4.Rating（1 byte）

同理，可以设计photos的table

### 5.System APIs
搜索API
```
search(api_dev_key, search_terms, user_location, radius_filter, maximum_results_to_return, category_filter, sort, page_token)
```
api_dev_key (string): 就是用户账号，可以用来做rate limiter  
search_terms (string): 就是搜索条件  
user_location (string): 用户的坐标  
radius_filter (number): 搜索的半径，单位是米  
maximum_results_to_return (number): 最多的返回结果  
category_filter (string): Optional 分类信息，比如餐厅，影院等  
sort (number): Optional 排序模式比如: Best matched (0 - default), Minimum distance (1), Highest rated (2).  
page_token (string): 返回指定的页数

Return（JSON）  
将会返回一系列的符合条件的结果，包含名称，地址，分类，评分等等。

### 6.Basic System Design and Algorithm
注意一次query会返回大量的数据，而且用户希望能实时返回数据。地址信息并不会更新非常频繁（不同于打车，更新很快），所以做了以下设计：  

a.SQL solution  
最简单的就是存MySQL之类关系数据库，找最近地点就用query：
```
Select * from Places where Latitude between X-D and X+D and Longitude between Y-D and Y+D
```
这样找地点不是特别准确，找的是一个方块而不是严格的距离公式。不过简单。

How efficient would this query be？假设有500M的地点，因为我们有latitude和longitude两个大index，所以肯定不会快。而且使用X+D之类的between可能返回大量数据。

b.Grids  
这个应该属于GeoHash的方法，就是把地球铺平，grid化，这样每个小grid都是包含了一定范围的longtitude和latitude。这样通过给定的用户位置和半径，就可以找到neighoring grid，然后再从这些grid中找出附近的的地点。  
假设GridID（4 bytes）就是这个grid系统的每个grid的编号。

What could be a reasonable grid size？  
如果grid大小刚好是用户要query的，那么只需要再这个grid里面找到满足条件的地点就行了。因为grid都是固定大小，所以找neighboring grids也是比较简单的。  
在数据库层面，也用table来记录GridID并且进行index，这样搜索的query类似：
```
Select * from Places where Latitude between X-D and X+D and Longitude between Y-D and Y+D and GridID in (GridID, GridID1, GridID2, ..., GridID8)
```
这样就大大降低了runtime。

Should we keep our index in memory？  
如果index都在内存中显然更快，如果内存够大，可以将index都存放在Hashtable中，key就是GridID，value就是list of LocationID

How much memory will we need to store the index？  
假设我们的搜索半径是10miles，整个地球的面积大概200million square miles（这个真不知道--!），我们就会有20million个grids。所以需要4 bytes来存放这些gridID，因为LocationID是8 bytes，所以大概需要4GB的空间来存放：
```
(4 * 20M) + (8 * 500M) ~= 4 GB
```
这种方式在某个Grid有大量数据的时候还是可能表现很慢，解决方法就是如果某一个gridID含有过多的LocationID就继续分成更细小的grids。所以主要挑战有：1）怎么map这些grid到Location 2）怎么找到这些grid的neighboring grid

c.Dynamic size grids  
假设如果一个grid少于500个地点就可以快速检索，那么就需要将密集的Grid继续细分。比如San Francisco就会有很多grid，而1个太平洋只有个别大grid。

What data-structure can hold this information？  
就是quad-tree了，一个tree node下面有4个children。一旦某个node超过了500个地点，就拆分为4个一样大小的子树。

How will we build a QuadTree？  
建树过程就是从整个世界开始，然后持续拆分直到所有node都少于500个locations。

How will we find the grid for a given location？  
用Quadtree的遍历，进行dfs，设计好gridID就知道dfs哪个了，直到叶子节点。

How will we find neighboring grids of a given grid？  
因为只有叶子节点包含locationID信息，所以可以用个doubly linked list将所有叶子节点连接起来，这样就可以方便的前进和后退。另外一种方法就是通过node的parent，在设计QuadTree时候也假如parent指针，这样也能方便的定位兄弟节点siblings了。  
一旦获取了附近的LocationID信息，就可以通过这些locationID到database去query了。

What will be theh search workflow？  
首先找到用户location所在的node，如果该node包含了足够的地点，就直接返回，如果没有，就持续扩展neighboring（通过parent pointer或者doubly linked list）直到有足够数量的地点，或者超过了maximum radius。

How much memory will be needed to store the QuadTree？  
显然整个QuadTree都要存放在内存中，如果我们只缓存LocationID和latitude/Longitude，那么需要12GB内存
```
(8+8+8) * 500M => 12 GB
```
因为每个grid最多500个地点，所以会有超过1M的grids
```
500M / 500 >= 1M grids
```
一个QuadTree除了叶子节点，还有中间的一系列internal node，大概1/3左右，存放这些需要大概10MB内存
```
1M * 1/3 * 4 * 8 = 10 MB
```
所以，总共QuadTree需要大概12.01GB内存，很容易满足。

How would we insert a new Place into our system？  
当有新地点，我们需要同时加入到数据库和QuadTree中去。加入数据库很简单。加入树的话，如果树在一个服务器很简单，但是如果QuadTree是分布在不同的服务器，那么还需要先定位服务器然后添加（后续讨论）。

### 7.Data Partitioning
假如地点增长太多，一个服务器内存不够怎么办？一台服务器无法服务大量请求怎么办？这种情况，就需要分区QuadTree了。  
这里提到了两种分区方式：  

a.Sharding based on regions: 
将地点分成多个regions，所有同一个region的存放在一个fixed的node里面。存放新地点和query附近的地点都只从某1个region里面寻找，这种方式的问题在于：  
1.如果一个region太热门，那么针对该region的query就会有很多，显然性能跟不上。  
2.某些region相对其他region会包含更多的地点，这样当regions不规则增大，维护扩展会很困难。  
解决方法是要么重新分区repartition或者使用consistent hashing

b.Sharding based on LocationID：  


