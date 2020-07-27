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
