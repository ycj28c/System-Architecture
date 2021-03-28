https://www.jiuzhang.com/qa/627

### 要求
设计一个只读的lookup service. 后台的数据是10 billion个key-value pair, 服务形式是接受用户输入的key，返回对应的value。已知每个key的size是0.1kB，每个value的size是1kB。要求系统qps >= 5000，latency < 200ms.

给的机器
```
commodity server
8X CPU cores on each server
32G memory
6T disk
```

### 解决方案
本质上是一个分布式的文件管理系统，和GFS的处理一样，是Master-slave的结构，Master存放文件key，然后有多个chunk服务器只存放chunk。

本题简化了GFS：  
1.因为是lookup，只涉及到读取，所以所有数据都可以存满disk。  
2.value很小，所以不涉及到拆分value，直接完整的放到各服务器即可。  

核心是计算题，要满足：  
1.qps >= 5000  
2.latency < 200ms  

##### 计算存储

存储所有key需要 10 billion * 0.1kb = 1TB  
存储所有value需要 10 billion * 0.1kb = 10TB  

一个disk可以存放6T，理论上两个disk（服务器）就能够存放下所有的内容  

至此，我们需要至少2台机器。如果考虑1个数据3份拷贝，那么也就是6台机器。

##### 计算latency（单次获取key的时间计算）

首先的常识是普通磁盘的寻轨需要10ms，表示每个地址需要10ms让磁头移动过去（如果使用ssd固态硬盘这个时间是0.2ms）。  

然后搜索这个key在磁盘的位置也需要时间，假设文件的排列是有序的，有两种搜索方式：  
1.从disk进行搜索  
在磁盘通过二分法搜索，需要log(10billion) ~= 33，也就是搜索33次才能找到这个key，每次搜索需要10ms，那么就是330ms。   
查看一个文件的总时间就是330ms + 10ms（开始读文件需要一次磁头读取时间） + 0.5ms（同一个数据中round trip时间） = 340.5ms  
注意340.5ms的单key读取时间超过了200ms的latency的要求了，是不满足要求的。  

2.将key的地址信息存放到memory  
这样的寻找时间可以忽略不计，搜索文件的时间就降到了10.5ms左右了。  

不过memory是有上限的，要看是否能够存下。一台机器的内存是32GB，那么40台机器 = 1280GB就能保证存放下所有的key索引了（假设地址8个Byte就够了）  
8Byte * 10 billion = 74GB   
1TB + 74GB < 1280GB   

至此，我们需要至少40台机器。

##### 计算QPS
一个服务器获取一个文件的时间是10.5ms，磁盘读取内容的速度大概30MB/s，因为文件很小1KB所以这个读取时间可以忽略。

那么单机QPS就是：  
1s / 10.5ms ~= 100 qps  
8个CPU似乎没啥用哈，瓶颈在磁盘。所以单机可以上多块硬盘，这里用2块（感觉具体多少块硬盘还可以深入讨论）  
2个disk的QPS可以提升到 200 qps。

再看我们的要求是5000QPS，目前40 * 200 = 8000是可以满足要求的。

至此，我们需要至少40台机器。就可以满足设计的需求了。

### 一个GFS代码示例

```
Description
Implement a simple client for GFS (Google File System, a distributed file system), it provides the following methods:

read(filename). Read the file with given filename from GFS.
write(filename, content). Write a file with given filename & content to GFS.
There are two private methods that already implemented in the base class:

readChunk(filename, chunkIndex). Read a chunk from GFS.
writeChunk(filename, chunkIndex, chunkData). Write a chunk to GFS.
To simplify this question, we can assume that the chunk size is chunkSize bytes. (In a real world system, it is 64M). The GFS Client's job is splitting a file into multiple chunks (if need) and save to the remote GFS server. chunkSize will be given in the constructor. You need to call these two private methods to implement read & write methods.

Have you met this question in a real interview?  
Example
GFSClient(5)
read("a.txt")
>> null
write("a.txt", "World")
>> You don't need to return anything, but you need to call writeChunk("a.txt", 0, "World") to write a 5 bytes chunk to GFS.
read("a.txt")
>> "World"
write("b.txt", "111112222233")
>> You need to save "11111" at chunk 0, "22222" at chunk 1, "33" at chunk 2.
write("b.txt", "aaaaabbbbb")
read("b.txt")
>> "aaaaabbbbb"

设计GFS系统的分布式存储系统，挺有意思的。
要把文件拆分成为多个chunk，然后插入到各个服务器中去，read的时候根据index信息读取完整内容。
```
/* Definition of BaseGFSClient
 * class BaseGFSClient {
 *     private Map<String, String> chunk_list;
 *     public BaseGFSClient() {}
 *     public String readChunk(String filename, int chunkIndex) {
 *         // Read a chunk from GFS
 *     }
 *     public void writeChunk(String filename, int chunkIndex,
 *                            String content) {
 *         // Write a chunk to GFS
 *     }
 * }
 */
public class GFSClient extends BaseGFSClient {
    class Chunk {
        int chunkId;
        String content;
        public Chunk(int chunkId, String content){
            this.chunkId = chunkId;
            this.content = content;
        }
    }
    class ChunkIndex {
        int chunkId;
        int serverId;
        public ChunkIndex(int chunkId, int serverId){
            this.chunkId = chunkId;
            this.serverId = serverId;
        }
    }
    //需要Index
    Map<String, List<ChunkIndex/*chunkId*/>> index;
    //需要server
    Map<Integer/*serverId*/, Map<Integer/*chunkId*/, Chunk>> servers;
    
    private int globalchunkId;
    private int chunkSize;
    private int serverNum = 5; //假设服务器是0-4
    /*
     * @param chunkSize: An integer
     */
    public GFSClient(int chunkSize) {
        // do intialization if necessary
        this.chunkSize = chunkSize;
        this.globalchunkId = 0;
        index = new HashMap<>();
        servers = new HashMap<>();
    }

    /*
     * @param filename: a file name
     * @return: conetent of the file given from GFS
     */
    public String read(String filename) {
        // write your code here
        if(!index.containsKey(filename)){
            return null;
        }
        StringBuilder sb = new StringBuilder();
        for(ChunkIndex chunkIdx : index.get(filename)){
            String content = servers.get(chunkIdx.serverId).get(chunkIdx.chunkId).content;
            sb.append(content);
        }
        return sb.toString();
    }

    /*
     * @param filename: a file name
     * @param content: a string
     * @return: nothing
     * 注意这里如果已经有存在的文件了，是override，不是append
     */
    public void write(String filename, String content) {
        if(index.containsKey(filename)){
            delete(filename);
        }
        appendWrite(filename, content);
    }

    private void delete(String filename){
        for(ChunkIndex chunkIdx : index.get(filename)){
            servers.get(chunkIdx.serverId).remove(chunkIdx.chunkId);
        }
        index.remove(filename);
    }

    private void appendWrite(String filename, String content){
        // write your code here
        int offset = 0;
        int serverIdx = 0;
        while(offset < content.length()){
            int curChunkId = globalchunkId++;
            Chunk chunk;
            if(offset + chunkSize <= content.length()){
                chunk = new Chunk(curChunkId, content.substring(offset, offset + chunkSize));
                offset += chunkSize;
            } else {
                chunk = new Chunk(curChunkId, content.substring(offset));
                offset += content.length();
            }
            serverIdx %= serverNum;
            ChunkIndex chunkIdx = new ChunkIndex(curChunkId, serverIdx);
            
            servers.computeIfAbsent(serverIdx, x-> new HashMap<>()).put(curChunkId, chunk);
            index.computeIfAbsent(filename, x-> new ArrayList<>()).add(chunkIdx);
            serverIdx++;
        }
    }
}
```
```
