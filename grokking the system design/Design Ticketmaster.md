## Design Ticketmaster 笔记

设计一个online票务系统比如Ticketmaster或者BookMyShow

1. What is an online movie ticket booking system?  
提供用户在线购买电影票的服务。可以看当前的放映电影，还有预订座位，任何时间任何地点。

2. Requirements and Goals of the System  
功能需求：  
1)可以显示不同城市的成员电影院  
2)当用户选择了城市，只显示该城市的相关电影  
3)当用户选择了电影，服务应该显示上映该电影的影院和show time  
4)用户可以选择特定电影院和预订ticket  
5)服务可以让用户自己选择座位，用户可以选择任意数量的座位  
6)用户可以看出已经被订走的座位和有效的座位  
7)用户可以在支付前Hold座位5分钟  
8)用户可以实时？等待座位重新有效  
9)所有用户同样应该受到平等的服务，先到先得逻辑

非功能需求： 
1)系统要求高并发，同一时间会有多个预订请求同一个座位在特定时间。  
2)服务核心是票务预订，同时也包含了支付转账。系统必须高安全，所以database需要满足ACID

3. Some Design Considerations  
1)为了简化问题，假设服务无需用户验证  
2)系统不能只完成部分order，要么用户得到所有订票，要么都没有（transaction）  
3)公平是强制的  
4)为了防止抱怨，可以限制每个用户一次最多订10张票  
5)我们可以假设个别热门电影的上映会让这类电影的座位销售特别快，所以系统需要能够scalable并且highly available

4. Capacity Estimation  
Traffic estimate: 假设我们的服务有3billion页面浏览量/每月，并且销售10million的票/每月  

Storage estimate: 假设我们有500个城市，平均每个城市有10个电影院。每个电影院平均有2000座位，每天上映2个电影。假设每个座位预订需要50bytes的数据包(IDs, NumberOfSeats, ShowID, MovieID, SeatNumbers, SeatStatus, Timestamp, etc.)。所以如果要存放所有的电影院的每日上映数据我们需要：  
500 cities * 10 cinemas * 2000 seats * 2 shows * (50+50) bytes = 2GB / day  
如果要存储5年的数据，需要大概3.6TB的数据存储。

5. System APIs  
我们可以使用SOAP或者REST API来提供接口。比如：  

1)搜索电影的API
```
SearchMovies(api_dev_key, keyword, city, lat_long, radius, start_datetime, end_datetime, postal_code, 
includeSpellcheck, results_per_page, sorting_order)
```
api_dev_key(string):这个就是每个注册账号的特有key。  
keyword(string):用户搜索keyword。  
city(string):filter的城市。  
lat_long(string):latitude或者longitude用来filter，也可能包含radius(number)来选定某个想搜索的区域。  
start_datetime(string):filter电影的起始时间。  
end_datetime(string):filter电影的终止时间。  
postal_code(string):通过zipcode过滤。  
includeSpellcheck(Enum: "yes" or "no"):Yes表示提供拼写验证服务。  
results_per_page(number):返回的结果数量，最大30。  
sorting_order(string):排序结果的方式，可以有'name,asc','name,desc'等。

Returns:(JSON)  
一个返回的JSON结构例子：
```
[
  {
    "MovieID": 1,
    "ShowID": 1,
    "Title": "Cars 2",
    "Description": "About cars",
    "Duration": 120,
    "Genre": "Animation",
    "Language": "English",
    "ReleaseDate": "8th Oct. 2014",
    "Country": USA,
    "StartTime": "14:00",
    "EndTime": "16:00",
    "Seats": 
    [
      {  
        "Type": "Regular"
        "Price": 14.99
        "Status: "Almost Full"
      },
      {  
        "Type": "Premium"
        "Price": 24.99
        "Status: "Available"
      }
    ]
  },
  {
    "MovieID": 1,
    "ShowID": 2,
    "Title": "Cars 2",
    "Description": "About cars",
    "Duration": 120,
    "Genre": "Animation",
    "Language": "English",
    "ReleaseDate": "8th Oct. 2014",
    "Country": USA,
    "StartTime": "16:30",
    "EndTime": "18:30",
    "Seats": 
    [
        {  
          "Type": "Regular"
          "Price": 14.99
          "Status: "Full"
      },
        {  
          "Type": "Premium"
        "Price": 24.99
        "Status: "Almost Full"
      }
    ]
  },
 ]
```

2)预订电影的API  
```
ReserveSeats(api_dev_key, session_id, movie_id, show_id, seats_to_reserve[])
```
api_dev_key(string):同理，每个注册用户的特定key。  
session_id(string):用户的sessionID用来跟踪该次预订。一旦reservation超时，该session就将被删除。  
movie_id(string):要预订的movie。  
show_id(string):要预订的场次id。  
seats_to_reserve(number):包含了用预订的座位Ids。  

Returns:(JSON)  
返回预订的状态，比如"Researvation Successful","Reservation Failed - Show Full","Reservation Failed - Retry, as other users are holding reserved seats".

6. Database Design  
设计数据库的表（略），会有很多表！比如Movie，Show，Booking，User，Cinema，City，Cinema_Hall，Cinema_Seat，Show_Seat，Payment表。 

设计数据库时候要写出每个表的主键，外键，字段类型和长度。
  
几个relation：每个City有多个Cinemas，每个Cinema有多个halls，每个Movie有多个Shows，每个Show有多个Bookings，每个用户可以有多个Booking。

7. High Level Design  
同样是需要画图，这里略。整体结构比较简单，就是client->Load Balancers -> Web Servers -> Application Servers to handle Ticket Management -> Cache Server和Database

8. Detailed Component Design  
先假设是用single server提供服务进行设计。下面是一个订票流程：  
1)用户搜索电影  
2)用户选择电影  
3)系统显示有效的电影场次给用户  
4)用户选择了一个场次  
5)用户选择了需要的座位数量   
6)如果剩余的座位数量足够，给用户显示电影院座位图。如果不足，跳过step 8  
7)一旦用户选择座位，系统尝试预留座位  
8)一旦用户不能被预留，进行如下操作：  
*场次已满，显示错误信息  
*如果座位不再有效，但是其他座位有效，让用户选择其他座位  
*如果座位不足够了，但是并不是所有座位被预定而是被其他用户Hold了，该用户被放在等待页面知道有足够的座位。有可能有如下情况： 
  +有足够的座位了，用户重新进入电影座位页面选座  
  +在等待中，剩余的座位数量不足以满足用户的准备购买量，显示错误信息  
  +用户cancel请求，回到电影搜索页面  
  +用户最多能等待1小时，然后就会超时回到电影搜索页面  
9)如果用户订票成功，用户有5分钟去完成支付。只有完成支付才算完成booking。如果5分钟内没有完成支付，所有被预留的座位就全部变为avaiable状态可供其他用户选择。

这个workfall逻辑是比较长的，难点之一

How would the server keep track of all the active reservation that haven't been booked yet? And how would the server keep track of all the waiting customers?  
我们需要两个守护进程服务，1个用来跟踪所有的有效reservation和删除过期reservation，叫做ActiveReservationService。另外一个服务用来跟踪所有等待的用户，当有足够的seats，这个进程就会通知（最长等待）的用户来进行选择，叫做WaitingUserService。

a. ActiveRservationService  
我们可以将每个Show的所有reservations存放在内存中，使用Linked HashMap或者TreeMap之类的数据结构。如果就可以根据时间顺序判断reservation的状态。  
为了记录每场show的预约，可以用HashTable，其中key是ShowID，value是LinkedHashMap包含了BookingID和Timestamp。

在数据库中，我们将预约存储在Booking表，expiry time保存在Timestamp column。其中Status字段有值Reserved(1)，当booking完成后，Status会更新为Booked(2)并且将之从LinkedHashMap中移除。如果预约过期，我们从LinkedHashMap移除记录，同时也更新Status为Expired(3)。

此外ActiveReservationsService也会连接外部的金融接口，处理用户的支付。当Booking完成后或者预约过期，WaitingUsersService会得到信号让等待信号得到服务。

b. WaitingUsersService  
和ActiveReservationsService一样，也可以将一个场次的waiting users存放在memory，同样使用LinkedHashMap或者TreeMap。如果用户cancel了请求就可以直接从map中删除。并且，因为我们是first-come-first-serve的，所以LinkedHashMap是指向最长等待的用户的，当有位置时候，总是先服务等待最久的用户。

大范围我们使用HashTable来存储所有的场次的等待用户，key是ShowID，value就是LinkedHashMap包含了UserIDs和wait-start-time。

用户可以使用Long Polling技术来保持他们的预约更新。 

Rservation Expiration  
在服务端，ActiveRservationService保持对所有reservations的expiry的跟踪。但是因为客户端也会显示一个计时器，这个计时器有可能和服务器的有差异，所以需要增加大概5秒钟的buffer，防止例如用户在服务器超时后才过期，让购买失败。

9. Concurrency  
How to handle concurrency，such that no two users are able to book same seat.这是重点之一，我们可以通过SQL databases的transactions来防止冲突。比如，如果我们使用SQL server我们可以利用Transaction Isolation Levels(??)来锁定rows，下面是sample：  
```
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
 
BEGIN TRANSACTION;
 
    -- Suppose we intend to reserve three seats (IDs: 54, 55, 56) for ShowID=99 
    Select * From Show_Seat where ShowID=99 && ShowSeatID in (54, 55, 56) && Status=0 -- free 
 
    -- if the number of rows returned by the above statement is three, we can update to 
    -- return success otherwise return failure to the user.
    update Show_Seat ...
    update Booking ...
 
COMMIT TRANSACTION;
```  
'Serializable'就是最高的isolation level，防止了Dirty，Nonrepeatable和Phantoms的读取。需要注意的是，在transaction中，如果我们读取了rows，这些rows也会被锁定无法update。

10. Fault Tolerance  
What happens when ActiveRservationService or WaitingUsersService crashes？  
当ActiveReservationsService宕机，我们仍然可以从Booking表中读取预约数据。因为我们在表中保存了Status column。另外一个方案是master-slave设置，当master的AcitiveReservationsService服务挂了，slave可以接上。  
注意我们这里没有存放waiting users到数据库，所以，当WaitingUsersService挂了，我们无法恢复除非使用了master-slave设置。  
同理，我们也可以在数据库上使用master-slave设置，让数据库fault tolerant。

11. Data Partitioning  
主要是讨论在大量场次数据量情况下无法分区。  

Database partitioning: 如果我们用MovieID分区，那么我们该movie的场次都会存放在一个服务器中。这样对于个别热门电影，可能造成大量load。一个更好的方案是通过ShowID分区，这样能保证负载平均分布到不同服务器中。  

ActivieReservationService and WaitingUserService partitioning: 所有的用户sessions和行为都由我们的服务器管理，所以需要一些机制来分配。我们可以使用Consistent Hashing来处理ActiveReservationSercieh和WaitingUserService，通过ShowID来进行Hash。这样，一个场次的所有预约和等待用户将通过特定的服务器来处理。假设复杂均衡让Consistent Hashing为每场Show了分配了3个服务器，每当预约过期，服务器将做如下操作： 
1)更新数据库删除booking数据，在show_seats表中删除seats数据  
2)在内存的LinkedHashMap中删除reservation  
3)通知用户他们的预约已经过期  
4)广播所有的WaitingUserService服务器，找到等待最久的那个用户。Consistent Hashing机制可以告知那个服务器有这些目标用户。（这个具体是？？）  
5)发送信息到WaitingUserService服务器激活等待最久的用户，如果有足够的seats

当reservation成功，做以下操作：  
1)服务器hold预约，并且发送给所有包含该show记录的服务器。这样这些服务器就可以让所有waiting users过期，如果waiting user需要的seats多余avaiable的seats。  
2)在收到上述信息后，所有hold等待用户的服务器会query数据库找出还剩下多少有效座位。一个数据库cache可以保证这个query只跑一次（这个留意下）  
3)如果waiting用户需要的seats多余avaialbe的seats，将被expire，因此WaitingUserService必须iterate LinkedHashMap看需要删除的waiting users(这个可能会较慢)。


自己总结下： 
这个设计题要了解清楚需求，包括搜索，预定，座位，支付等功能。本题的业务流程比较复杂，性能上要求不是非常高，通过标准的分布式比如Consistent Hashing就可以处理，但是对安全性和可靠性要求很高。所以对并发的处理和事务的处理是重点，不过这些都可以由一个关系型数据库来搞定。对数据库的设计和表间关系比较复杂，每个对象尽量独立。业务逻辑复杂，对细节的挖掘处理要把握好，比如等待时候没座位了，等待过期等等。这里的AcitiveReservationsService和WatingUserService是独立出来的重点服务，因为负载较大，所以也需要引入Load Balance和Consistent Hashing。用户信息用Map<ShowID, LinkedHashMap<BookingID, TimeStamp>>来保存的细节很重要。此外最后还要深挖细节，比如用Long polling解决用户状态更新，还有用户服务器时间不一致的处理都是加分项，不过大方向对了细节都是有一定基础了。
