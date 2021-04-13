其实在算法题中对string进行hash经常会用到，就类似一个sliding windows存放。不过bloom Filter有其他的应用场景。

## 应用场景
1)垃圾邮件地址过滤(地址数量很庞大)  
2)爬虫URL地址去重  
3)解决缓存击穿问题  
4)浏览器安全浏览网址提醒  
5)google 分布式数据库Bigtable以及Hbase使用布隆过滤器来查找不存在的行或列，以减少磁盘查找的IO次数  
6)文档存储检索系统也可以采用布隆过滤器来检测先前存储的数据  

## Bloom Filter的特点：  
1>、不存在漏报（False Negative），即某个元素在某个集合中，肯定能报出来。  
2>、可能存在误报（False Positive），即某个元素不在某个集合中，可能也被爆出来。  
3>、确定某个元素是否在某个集合中的代价和总的元素数目无关。  

简单说就是用速度换准确度

## 具体的实现  
1.Standard Bloom Filter  
就是hash后的标志位置是0或者1，比较快

2.Counting Bloom Filter  
就是hash后的标志位置是一个数，这样可以处理数据删除和修改的情况

实际的hash function可能有多个，对于一个data可能会标记多个数据位，来增加准确性（和Bloom Filter的hash长度也有密切关系）。

## 和HashMap的区别
比hashmap的占用空间明显少很多，hashmap需要存放所有数据，而bloom filter只存放检验数据。比如Bigtable就在每一个块内设置了bloom filter，快速检查数据是否存在，只要不存在直接跳到下一块，加快检索速度。  

有一个题目：  
如果A和B都有大量的URL数据，内存放不下，如果要找到A和B的交集URL，怎么处理？  
答案就是使用bloom filter，这样不用存放具体的url（每个url还是比较长的，很占空间），先将所有A的URL进行bloom filter，然后遍历B，只要B的url也存在于bloom filter，就是交集。缺点就是可能存在误报。通常的误报率大概3% - 4%。

## Bloom Filter的扩容
Bloom Filter对于扩容支持不太好，如果一开始的array长度是10，当数据量变大，我们需要array长度100的时候，该怎么做？  

做法是保留原array不变，新建一个长读100的array，新的数据只加入到新的array，每次检索同时要检索所有bloom filter，只要一个说positive那么就算positive。  
缺点很明显，速度明显变慢，误报率明显变高。

## 代码演示
普通版BloomFiler（多个Hash Function）
```
public class StandardBloomFilter {
    class HashFunction {
      public int cap, seed;

      public HashFunction(int cap, int seed) {
        this.cap = cap;
        this.seed = seed;
      }

      public int hash(String value) {
        int ret = 0;
        int n = value.length();
        for (int i = 0; i < n; ++i) {
          ret += seed * ret + value.charAt(i);
          ret %= cap;
        }
        return ret;
      }
    }

    public BitSet bits;
    public int k;
    public List<HashFunction> hashFunc;

    public StandardBloomFilter(int k) {
      this.k = k;
      hashFunc = new ArrayList<HashFunction>();
      for (int i = 0; i < k; ++i) {
        hashFunc.add(new HashFunction(100000 + i, 2 * i + 3));
      }
      bits = new BitSet(100000 + k);
    }

    public void add(String word) {
      for (int i = 0; i < k; ++i) {
        int position = hashFunc.get(i).hash(word);
        bits.set(position);
      }
    }

    public boolean contains(String word) {
      for (int i = 0; i < k; ++i) {
        int position = hashFunc.get(i).hash(word);
        if (!bits.get(position)) {
          return false;
        }
      }
      return true;
    }
}
```

计数版BloomFilter  
和普通版的BloomFilter几乎一样，主要区别在于使用int[]来统计而不是BitSet/boolean[]，这样就可以对Hash的结果进行统计（加减）  
```
public class CountingBloomFilter {

    class HashFunction {
      public int cap, seed;

      public HashFunction(int cap, int seed) {
        this.cap = cap;
        this.seed = seed;
      }

    public int hash(String value) {
      int ret = 0;
      int n = value.length();
      for (int i = 0; i < n; ++i) {
        ret += seed * ret + value.charAt(i);
        ret %= cap;
      }
      return ret;
    }
  }
  
  public int[] bits;
  public int k;
  public List<HashFunction> hashFunc;

  public CountingBloomFilter(int k) {
      this.k = k;
      hashFunc = new ArrayList<HashFunction>();
      for (int i = 0; i < k; ++i) {
        hashFunc.add(new HashFunction(100000 + i, 2 * i + 3));
      }
      bits = new int[100000 + k];
  }

  public void add(String word) {
    for (int i = 0; i < k; ++i) {
      int position = hashFunc.get(i).hash(word);
      bits[position] += 1;
    }
  }

  public void remove(String word) {
    for (int i = 0; i < k; ++i) {
      int position = hashFunc.get(i).hash(word);
      bits[position] -= 1;
    }
  }

  public boolean contains(String word) {
    for (int i = 0; i < k; ++i) {
      int position = hashFunc.get(i).hash(word);
      if (bits[position] <= 0)
      return false;
    }
    return true;
  }
}
```

## Reference
[bloom Filter布隆过滤器简介](https://www.jianshu.com/p/51e40483911f)  
[海量数据处理算法—Bloom Filter](https://blog.csdn.net/hguisu/article/details/7866173)
