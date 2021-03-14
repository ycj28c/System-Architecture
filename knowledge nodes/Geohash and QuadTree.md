### 应用场景
核心是降维，对3维地球的坐标转换为1维编码，可以快速定位和搜索附近的点，下面就用找咖啡店为例

### Geohash：
使用的是peano空间填充曲线。编码过程类似2分划大小，经度-180到180，维度-90到90，具体可以看[Geohash原理](https://www.jianshu.com/p/1ecf03293b9a)。具体的使用就是先用定位点的GeoHash编码进行匹配查询，编码近的块中搜索咖啡店。
缺点是个别位置距离相差过大，这样在搜索边界处的点的编码却差很远。所以使用时候还需要周围8个区域的GeoHash编码一起找。边界问题需要在2维坐标解决，这里有讲解：[Geohash](https://github.com/GongDexing/Geohash)。
Geohash主要就是用来快速筛选附近地点的，具体求距离还是需要计算每个的坐标。

这段介绍Geohash也不错：[GeoHash核心原理解析](https://www.cnblogs.com/LBSer/p/3310455.html)

### QuadTree：
或者Google s2算法，使用Hilbert填充曲线，将地球体投影到了6个平面正方形，然后4分块递归，计算每一片区域的cellID。块的放大缩小是4的次方，比较平滑，优点是没有边界的问题，一点在小坐标系和大坐标系位置变化也不大。
具体使用就是直接使用cellID的坐标匹配就行了。64位的坐标，基本通过坐标就可以得知周围的坐标的。具体的看[Google S2 是如何解决空间覆盖最优解问题的](https://halfrost.com/go_s2_regioncoverer/)，细节不懂。

在这里有详细讲解：
[高效的多维空间点索引算法 — Geohash 和 Google S2](https://halfrost.com/go_spatial_search/)

一个QuadTree结构的例子
```
// Definition for a QuadTree node.
class Node {
    public List<Location> locations; //一个QuadTree最多存放500个地点
    public boolean isLeaf; //只有叶子节点才存放地点数据
    public Node topLeft;
    public Node topRight;
    public Node bottomLeft;
    public Node bottomRight;

    public Node() {
        this.locations = new ArrayList();
        this.isLeaf = false;
        this.topLeft = null;
        this.topRight = null;
        this.bottomLeft = null;
        this.bottomRight = null;
    }
};
class Location {
    public long latitude;
    public long longtitude;
}
```
