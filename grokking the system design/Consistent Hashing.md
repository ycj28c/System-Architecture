## Consistent Hashing 笔记

### 什么是Consistent Hashing
分布式哈希表DHT（Distributed Hash Table），如果给定n个服务器，最简单的hash分布就是Key % n，但是存在问题drawbacks：  
1.这样做无法水平扩展，每当增加新服务器，所有的mapping都会失效，因为现在是key % (n+1)了  
2.不能很好的load balanced，特别是对于非统一数据，可能部分服务器满负荷，而其他服务器却idle  

而如果引入了Consistent Hashing，能让云的扩展和收缩更加平滑。  
比如使用一个集群的Memcache，cache数据能均匀到每一个Memcache node，即使宕机或者撤机器了也能自动平衡负载，然后扩容也能平滑进行，多么美丽的设计。

### 原理
通常一个环范围是[0, 2^32-1]，假设有3个服务器，和4个object，将服务器和object离散后，分布到环上。每个object对应于顺序的下一个服务器。如果服务器down了或者增加服务器，只需要重定向变化节点之前的object位置即可。

对于具体的code来说，就是使用TreeSet。为了离散，key可以使用ip+port+name拼成的串计算。  
如果点A被删除了，那么将A的object都移动到下一个，就是使用TreeSet.ceiling(A)，
如果没有下一个，就直接从第一个开始，也就是TreeSet.first()。
```
TreeSet<Integer> set = new TreeSet<>();
int hashKey = hash(ip+port+name+i);
set.add(hashKey);
...

set.ceiling(1);
set.first();
```


### 虚拟节点 virtual replicas
为了负载均衡，假设各台服务器性能差不多，此时流量突增，一台server由于流量过载而挂掉，那么它的下一台因为承载了2倍的流量，很有可能也会挂掉，依此类推，最后所有的节点都会挂掉，造成"雪崩"。  
所以要将1个物理server编程3个虚拟的hash节点（离散hash分布），那么一台机器down了，就会离散的引流到不同的服务器中去。具体看[一致性Hash(Consistent Hashing)原理剖析](https://blog.csdn.net/lihao21/article/details/54193868)

这种情况下的也差不多使用treeMap，不过key表示虚拟节点，value要包含物理节点信息。比如
```
for(i=1  -->  N) // N为每个server对应的分片数量
{
   Map.put(hash(ip+port+name+i)，node) // 所有虚拟节点放进去
}
```
在实际应用中，通常将虚拟节点数设置为32甚至更大，因此即使很少的服务节点也能做到相对均匀的数据分布。

### 缺陷
一致性哈希算法Consistent Hashing并不是强一致性，也不是高可用方案，如果server挂了数据丢了就是丢了，除非有恢复手段，它只是一种减少由扩缩容引起的命中率下降的手段。

## Reference
[一致性Hash(Consistent Hashing)原理剖析](https://blog.csdn.net/lihao21/article/details/54193868)  
[Java一致性Hash算法的实现](https://blog.csdn.net/flyfeifei66/article/details/82458618)