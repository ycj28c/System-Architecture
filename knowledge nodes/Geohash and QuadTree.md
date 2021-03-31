### 应用场景
核心是降维，对3维地球的坐标转换为1维编码，可以快速定位和搜索附近的点，下面就用找咖啡店为例

### Geohash：
使用的是peano空间填充曲线。编码过程类似2分划大小，经度-180到180，维度-90到90，具体可以看[Geohash原理](https://www.jianshu.com/p/1ecf03293b9a)。具体的使用就是先用定位点的GeoHash编码进行匹配查询，编码近的块中搜索咖啡店。
缺点是个别位置距离相差过大，这样在搜索边界处的点的编码却差很远。所以使用时候还需要周围8个区域的GeoHash编码一起找。边界问题需要在2维坐标解决，这里有讲解：[Geohash](https://github.com/GongDexing/Geohash)。
Geohash主要就是用来快速筛选附近地点的，具体求距离还是需要计算每个的坐标。

这段介绍Geohash也不错：[GeoHash核心原理解析](https://www.cnblogs.com/LBSer/p/3310455.html)

Geohash很好的一点就可以每一位可以代表一个缩放地图的层级，所以Geohash可以直接定位，而且可以和latitude，longtitude直接转换，速度很快。

Geohash和location的转换代码
```
public class GeoHash {
    /*
     * @param latitude: one of a location coordinate pair 
     * @param longitude: one of a location coordinate pair 
     * @param precision: an integer between 1 to 12
     * @return: a base32 string
     * 
     * 输入: 
     * lat = 39.92816697 
     * lng = 116.38954991
     * precision = 12 
     * 输出: "wx4g0s8q3jf9"
     * 
     * 纬度latitude：-90~90
     * 经度longitude：-180～180
     * 经纬各生成一个30位的二进制数：2分法，>mid取1，<=mid取0
     * 先经度再纬度，把2个二进制数合并成一个60位的二进制数
     * 32 = 2^5
     * 对2进制数每5位作为一个数转换成integer，转换成的integer在0~32之间，在base32数组中取integer对应位的char
     * precision: 取前x 位的char
     * 前p位char: 5* p位数，对每个维度取 5* p／2 + 1个digit即可
     */
    final String BASE32 = "0123456789bcdefghjkmnpqrstuvwxyz";
    public String encode(double latitude, double longitude, int precision) {
        // write your code here
        int len = (precision + 1) / 2 * 5;
        System.out.println(len);
        String latBitStr = getBit(latitude, -90.0, 90.0, len);
        String lngBitStr = getBit(longitude, -180.0, 180.0, len);
        
        String geoBitStr = combine(latBitStr, lngBitStr);
        //这里注意只取精度位，有可能超过精度的
        return getGeoHash(geoBitStr).substring(0, precision);
    }
    private String getGeoHash(String geoBitStr){
        int start = 0;
        StringBuilder sb = new StringBuilder();
        while(start < geoBitStr.length()){
            String piece = geoBitStr.substring(start, start + 5);
            int bit = Integer.parseInt(piece, 2);
            sb.append(BASE32.charAt(bit));
            start += 5;
        }
        return sb.toString();
    }
    private String combine(String latBitStr, String lngBitStr){
        StringBuilder sb = new StringBuilder();
        for(int i=0;i<latBitStr.length();i++){
            sb.append(lngBitStr.charAt(i));
            sb.append(latBitStr.charAt(i));
        }
        return sb.toString();
    }
    private String getBit(double lat, double left, double right, int len){
        StringBuilder sb = new StringBuilder();
        
        for(int i=0;i<len;i++){
            double mid = left + (right - left) / 2;
            if(lat <= mid){
                sb.append("0");
                right = mid;
            } else {
                sb.append("1");
                left = mid;
            }
        }
        return sb.toString();
    }
}
```
还有设计miniYelp的例子，挺有趣
```
**
 * https://www.lintcode.com/problem/detail/509/note/225267
 * Definition of Location:
 * class Location {
 *     public double latitude, longitude;
 *     public static Location create(double lati, double longi) {
 *         // This will create a new location object
 *     }
 * };
 * Definition of Restaurant
 * class Restaurant {
 *     public int id;
 *     public String name;
 *     public Location location;
 *     public static Restaurant create(String name, Location location) {
 *         // This will create a new restaurant object,
 *         // and auto fill id
 *     }
 * };
 * Definition of Helper
 * class Helper {
 *     public static get_distance(Location location1, Location location2) {
 *         // return distance between location1 and location2.
 *     }
 * };
 * class GeoHash {
 *     public static String encode(Location location) {
 *         // return convert location to a GeoHash string
 *     }
 *     public static Location decode(String hashcode) {
 *          // return convert a GeoHash string to location
 *     }
 * };
 */
public class MiniYelp {
    //餐厅表 记录所有餐厅 (类似Uber的Driver Table)
    //Map<Integer, Restaurant> restaurants
    
    //类似Uber的Location Table 记录Geo_Hash到其涵盖的餐厅的映射
    //Map<String, Set<Restaurant>> geo2restaurants

    public MiniYelp() {
        // initialize your data structure here.
    }

    // @param name a string
    // @param location a Location
    // @return an integer, restaurant's id
    public int addRestaurant(String name, Location location) {
        // 更新restaurants和geo2restaurants
    }

    // @param restaurant_id an integer
    public void removeRestaurant(int restaurant_id) {
        // 更新restaurants和geo2restaurants
    }

    // @param location a Location
    // @param k an integer, distance smaller than k miles
    // @return a list of restaurant's name and sort by 
    // distance from near to far.
    public List<String> neighbors(Location location, double k) {
        // 计算用户的Geo_Hash
        // 如果要查询的范围k 超过了geo2restaurants的精度(这里是630km)
        // 扫描整个餐厅表找满足的餐厅
        // 否则 利用geo2restaurants找满足的餐厅
        // 利用TreeMap对餐厅的距离排序
    }
};
```

### QuadTree：
或者Google s2算法，使用Hilbert填充曲线，将地球体投影到了6个平面正方形，然后4分块递归，计算每一片区域的cellID。块的放大缩小是4的次方，比较平滑，优点是没有边界的问题，一点在小坐标系和大坐标系位置变化也不大。
具体使用就是直接使用cellID的坐标匹配就行了。64位的坐标，基本通过坐标就可以得知周围的坐标的。具体的看[Google S2 是如何解决空间覆盖最优解问题的](https://halfrost.com/go_s2_regioncoverer/)，细节不懂。

在这里有详细讲解：
[高效的多维空间点索引算法 — Geohash 和 Google S2](https://halfrost.com/go_spatial_search/)

一个QuadTree结构的例子
```
// Definition for a QuadTree node.
class Node {
    //没有找到相关资料，不过我估计这里还需要4个顶角位置来确定范围
    public Location topLeft;
    public Location topRight;
    public Location bottomLeft;
    public Location bottomRight;
    
    public boolean isLeaf; //只有叶子节点才存放地点数据
    public List<Location> locations; //一个QuadTree最多存放500个地点
    
    public Node topLeft;
    public Node topRight;
    public Node bottomLeft;
    public Node bottomRight;
};
class Location {
    public long latitude;
    public long longtitude;
}
```
