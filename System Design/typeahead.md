就是twitter的一个JavaScript的库，相当于1个搜索栏，具体请看[typeahead.js](https://twitter.github.io/typeahead.js/)，大概是2013年左右的项目。因为简单直观，用于系统设计讨论很好，这种叫做设计auto complete系统。

1.有默认的搜索项目
2.可以进行局部匹配，类似模糊匹配
3.有建议项

基础设计：

算法上：  
如何实现，使用Trie，使用Hash，BigO，优势，劣势

设计上：  
0. 需求，SLA(qps, latency, availability time)….
1. 流量计算，大数据，机器数量(cpu/内存/disk)
2. 如何获取出现频率，数据库设计（SQL/NOSQL，表和表关系，query）
3. 大规模如果分布式sharding/replica，consistent hashing，可用性Availability
4. 接口，前后端接口，API的接口，比如rest的接口设计
5. OOP设计，类和数据结构的设计
4. 优先资源优化，trade off，怎么减少log，cache

高级设计：
各种关联问题，比如监控，KPI，热门事件之类

哇，细想的话涉及的东西真是不少

## Reference
[Google系统设计面试题--设计一个auto complete 系统(type ahead系统)](http://www.noteanddata.com/interview-problems-system-design-learning-notes-autocomplete.html)
[Type ahead/Auto Complete 设计](https://www.jianshu.com/p/c7fc9092d9fe)
