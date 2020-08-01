## Indexes 笔记
Indexes在数据库中广为人知。在一个table创建index的作用是为了让搜索更快。

总之indexes就是一个key指向目标地址，因为索引较小所以找的快。如果数据分布在多个物理设备上，indexes就是最好的选择了。

How do indexes decrease write performance？  
index能极大的加速检索，但是如果index本身太大反而会降低insertion和update的速度。  
因为每当增加新行或者更新现有的行时候，我们不仅仅要将数据写入而且需要更新index。所以要避免无意义的索引。

### 自我总结
感觉这章没讲啥有用的