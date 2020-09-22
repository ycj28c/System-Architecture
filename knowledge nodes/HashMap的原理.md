Hash的理论支持
----------
Hash表设计:  
1.开放寻址法  
就是使用数组的方式存储。hash就是直接mod得到在数组的index。

解决冲突的方式有：  
1)线性探查（Linear Probing）：最简单的场景，如果当前被占了，就顺位找下一个有空的。  
2)二次探查（Quadratic Probing）：线性探查的太慢，让每次检查位置空间的步长为平方倍数，不过冲突仍然存在   
3)二度哈希（Rehashing）（或称为双重哈希（Double Hashing））：一组哈希函数 H1...Hn 的集合。当需要从哈希表中添加或获取元素时，首先使用哈希函数 H1。如果导致冲突，则尝试使用 H2，以此类推，直到 Hn。所有的哈希函数都与 H1 十分相似，不同的是它们选用的乘法因子（multiplicative factor）。哈希表中的所有元素值将依赖于哈希表的位置空间值，所以表中所有值也需要重新二度哈希。具体函数没有深入理解，可以看[哈希表和完美哈希](https://www.cnblogs.com/gaochundong/p/hashtable_and_perfect_hashing.html)

2.链接技术：  
链表的做法，把哈希到同一个槽中的所有元素都放到一个链表中，当冲突发生时，冲突的元素将被添加到桶（bucket）列表中，而每个桶都包含了一个链表以存储相同哈希的元素。也就是Java的做法。

3.BST技术：  
链表的演化版本，在冲突过多的时候，可以在log(N)复杂度进行元素的增减。

Hash函数的设计
----------
一个好的哈希函数应满足假设：每个关键字都等可能地被哈希到 m 个槽位的任何一个之中，并且与其他的关键字已被哈希到哪一个槽位中无关。不幸的是，通常情况下不太可能检查这一条件是否成立，因为人们很少能知道关键字所符合的概率分布，而各关键字可能并不是完全互相独立的。在实践中，常常运用启发式技术来构造好的哈希函数。比如在设计中，可以利用有关关键字分布的限制性信息等。

1.除法哈希法（The Division Method）  
就是mod大法了
```
hash(key) = key mod m
```

2.乘法哈希法（The Multiplication Method）  
```
hash(key) = floor( m * ( A * key mod 1) )
```
其中 floor 表示对表达式进行下取整，常数 A 取值范围为（0<A<1），m 表示哈希表的大小，mod 为取余操作。[A * key mod 1] 表示将 key 乘上某个在 0~1 之间的数并取乘积的小数部分，该表达式等价于 [A*key - floor(A * key)]。  
乘法哈希法的一个优点是对 m 的选择没有什么特别的要求，一般选择它为 2 的某个幂次，这是因为我们可以在大多数计算机上更方便的实现该哈希函数。

3.全域哈希法（Universal Hashing）  
任何一个特定的哈希函数都有可能出现这种最坏情况，唯一有效的改进方法就是随机地选择哈希函数，使之独立于要存储的元素。这种方法称作全域哈希（Universal Hashing）。全域哈希的基本思想是在执行开始时，从一组哈希函数中，随机地抽取一个作为要使用的哈希函数。就像在快速排序中一样，随机化保证了没有哪一种输入会始终导致最坏情况的发生。

## Java HashMap设计
Hash的原理比较复杂，以下用问题的形式解释这个结构（针对Java的HashMap设计）：

1.HashMap的意思，为什么不用Array？  
我们知道，通过对数组进行直接寻址（Direct Addressing），可以在 O(1) 时间内访问数组中的任意元素。所以，如果存储空间允许，可以提供一个数组，为每个可能的关键字保留一个位置，就可以应用直接寻址技术。

哈希表（Hash Table）是普通数组概念的推广。当实际存储的的关键字数比可能的关键字总数较小时，这时采用哈希表就会比使用直接数组寻址更为有效。因为哈希表通常采用的数组尺寸与所要存储的关键字数是成比例的。在哈希表中查找一个元素的期望时间是 O(1) 。

2.HashMap的构造函数怎么理解  
public HashMap(int initialCapacity, float loadFactor) ;  
第一个参数：初始容量，指明初始的桶的个数；相当于桶数组的大小。  
第二个参数：装载因子，是一个0-1之间的系数，根据它来确定需要扩容的阈值，默认值是0.75。  

3.HashMap是O(1)吗？  
其实不是，根据hashcode的冲突程度，会有O(K)的。

4.HashMap是O(N)的空间吗？  
其实不是，Java中的HashMap是根据2^X逐渐扩容的，所以通常都会有空间上的浪费。数组的大小永远是2的N次方，你随便给一个初始值比如17会转为32。默认第一次放入元素时的初始值是16。

5.什么是Hash冲突？  
用Hash函数hash Object的时候，可能有多个结果落在一个Bucket上，这就是hash冲突

6.HashMap的hash函数是怎么样的？  
在写Object的时候我们也会写hashcode()，结果是一个int的hashcode，int上下限有约40亿的范围，完全够用了，但是为了避免太多的冲突，Java底层还会加上一层hash使结果更加random。 

7.HashMap是怎么存储的？  
先对Entry进行Hash，每个Hash结构存放于Bucket中。Java8之前使用的单向链表，在Java8后新增了默认为8的阕值。Bucket根据长短可能会有LinkedList和BSTree两种implement，在冲突大的时候使用BSTree让复杂度变为log(N)。

在leetcode中也有相应的训练题，可以帮助理解Hash原理: Design HashSet [Solution](https://leetcode.com/problems/design-hashset/solution/)    

8.HashMap的(桶)大小为什么是2的幂？  
答案当然是为了性能。在HashMap通过键的哈希值进行定位桶位置的时候，调用了一个indexFor(hash, table.length);方法。
```
static int indexFor(int h, int length) {
	return h & (length-1);
}
```
可以看到这里是将哈希值h与桶数组的length-1（实际上也是map的容量-1）进行了一个与操作得出了对应的桶的位置，h & (length-1)。而&的性能相对%，/要好很多。  

9.什么是rehash？   
注意不同的键的的hashcode仅仅只能通过低位来区分，高位的信息没有被充分利用，这样问题就来了，就算我的散列值分布再松散，钥匙只取最后几位的话，碰撞也很严重。比如key1的hashcode为11111 10101，另一个key2的hashcode为00000 10101，很明显这两个hashcode不是一样的，甚至连相似性（例如海明距离）也是很远的。但是直接进行&操作得出的桶位置是同一个桶，这直接就产生了哈希冲突。为了防止这种情况的出现，HashMap它使用一个supplemental hash function对键的hashCode再进行了一个supplemental hash ，将最终的hash值作为键的hash值来进行桶的位置映射（也就是说JDK团队在为我们这群程序员加性能保险Orz）。这个过程叫做再哈希(rehash)。

具体代码(Java 8中的散列优化函数)：
```
//高16bit不变，低16bit和高16bit做了一个异或
static final int hash(Object key){
	int h;
	return (key == null) ? 0 : (h = key.hashcode()) ^ (h >>> 16);
}
```
这段代码也叫“扰动函数”，在Java8经过了简化，只做一次16位右位移异或混合，而不是四次，但是原来不变。 
用最低4位做hash混合了扰动函数的例子：  
```
h:               1111 1111 1111 1111 1111 0000 1110 1010
h >>> 16:        0000 0000 0000 0000 1111 1111 1111 1111
hash=h^(h>>>16): 1111 1111 1111 1111 0000 1111 0001 0101
(n-1) & hash:    0000 0000 0000 0000 0000 0000 0000 1111
                 1111 1111 1111 1111 0000 1111 0001 0101
                                0101 = 5				 
```
增加了扰动函数，让数更随机，实际大概减少了30%的碰撞。

10.HashMap是怎么扩容的？  
当map中包含的Entry的数量大于等于threshold = loadFactor * capacity的时候，且新建的Entry刚好落在一个非空的桶上，此刻触发扩容机制，将其容量扩大为2倍。

11.HashMap扩容的性能如何？  
因为扩容需要transfer现有的buckets，所以是有性能消耗的，所以初始化的时候设置好正确的大小对性能很有帮助。

12.HashMap是否线程安全？  
不，扩容的时候，hashmap会transfer现有的buckets。某个线程t所持有的引用next，可能已经被转移到了新桶数组中，那么最后该线程t实际上在对新的桶数组进行transfer操作。如果有更多的线程出现这种情况，那很可能出现大量线程都在对新桶数组进行transfer，那么就会出现多个线程对同一链表无限进行链表反转的操作，极易造成死循环，数据丢失等等，因此HashMap不是线程安全的，考虑在多线程环境下使用并发工具包下的ConcurrentHashMap。

13.HashMap的遍历是乱序吗？  
iterator()时顺着哈希桶数组来遍历，看起来是个乱序。

代码演示
----------
1.使用Array的HashSet
```
class MyHashSet {

    class Bucket {
	private int[] arr;
	public Bucket(){
		this.arr = new int[1000]; //0-1000
		Arrays.fill(arr, -1);
	}
	public void add(int y){
		arr[y] = y;
	}
	public void delete(int y){
		arr[y] = -1;
	}
	public boolean contains(int y){
		return arr[y] != -1;
	}
    }
    /** Initialize your data structure here. */
	private Bucket[] buckets;
    public MyHashSet() {
		this.buckets = new Bucket[1001]; //1000000 = 1000 * 1000;
        for(int i=0;i<1001;i++) buckets[i] = new Bucket();
    }
    
    public void add(int key) {
		int x = key / 1000; //0-1000
		int y = key % 1000; //1-999
        //System.out.println(x +","+y+","+ (buckets[x] == null));
		buckets[x].add(y);
    }
    
    public void remove(int key) {
		int x = key / 1000;
		int y = key % 1000;
		buckets[x].delete(y);
    }
    
    /** Returns true if this set contains the specified element */
    public boolean contains(int key) {
		int x = key / 1000;
		int y = key % 1000;
		return buckets[x].contains(y);
    }
}
```

2.使用LinkedList的HashSet
```
class MyHashSet {
    /** Initialize your data structure here. */
    class Bucket {
        private LinkedList<Integer> list;
        
        public Bucket(){
            this.list = new LinkedList<>();
        }
        public void insert(int key){
            if(list.indexOf(key)==-1){
                list.addFirst(key);
            }
        }
        public void remove(int key){
            list.remove(Integer.valueOf(key));
        }
        public boolean contains(int key){
            return list.indexOf(key) !=-1;
        }
    }
    private Bucket[] buckets;
    private int range = 769; //质数因子
    public MyHashSet() {
        buckets = new Bucket[range];
        for(int i=0;i<range;i++) buckets[i] = new Bucket();
    }
    
    public void add(int key) {
        int id = hash(key);
        buckets[id].insert(key);
    }
    
    public void remove(int key) {
        int id = hash(key);
        buckets[id].remove(key);
    }
    
    /** Returns true if this set contains the specified element */
    public boolean contains(int key) {
        int id = hash(key);
        return buckets[id].contains(key);
    }
    
    private int hash(int key){
        return key % range;
    }
}
```

3.使用BST的HashSet  
源自[Leetcode Design HashSet](https://leetcode.com/problems/design-hashset/solution/)
```
class MyHashSet {
  private Bucket[] bucketArray;
  private int keyRange;

  /** Initialize your data structure here. */
  public MyHashSet() {
    this.keyRange = 769;
    this.bucketArray = new Bucket[this.keyRange];
    for (int i = 0; i < this.keyRange; ++i)
      this.bucketArray[i] = new Bucket();
  }

  protected int _hash(int key) {
    return (key % this.keyRange);
  }

  public void add(int key) {
    int bucketIndex = this._hash(key);
    this.bucketArray[bucketIndex].insert(key);
  }

  public void remove(int key) {
    int bucketIndex = this._hash(key);
    this.bucketArray[bucketIndex].delete(key);
  }

  /** Returns true if this set contains the specified element */
  public boolean contains(int key) {
    int bucketIndex = this._hash(key);
    return this.bucketArray[bucketIndex].exists(key);
  }
}


class Bucket {
  private BSTree tree;

  public Bucket() {
    tree = new BSTree();
  }

  public void insert(Integer key) {
    this.tree.root = this.tree.insertIntoBST(this.tree.root, key);
  }

  public void delete(Integer key) {
    this.tree.root = this.tree.deleteNode(this.tree.root, key);
  }

  public boolean exists(Integer key) {
    TreeNode node = this.tree.searchBST(this.tree.root, key);
    return (node != null);
  }
}

public class TreeNode {
  int val;
  TreeNode left;
  TreeNode right;

  TreeNode(int x) {
    val = x;
  }
}

class BSTree {
  TreeNode root = null;

  public TreeNode searchBST(TreeNode root, int val) {
    if (root == null || val == root.val)
      return root;

    return val < root.val ? searchBST(root.left, val) : searchBST(root.right, val);
  }

  public TreeNode insertIntoBST(TreeNode root, int val) {
    if (root == null)
      return new TreeNode(val);

    if (val > root.val)
      // insert into the right subtree
      root.right = insertIntoBST(root.right, val);
    else if (val == root.val)
      // skip the insertion
      return root;
    else
      // insert into the left subtree
      root.left = insertIntoBST(root.left, val);
    return root;
  }

  /*
   * One step right and then always left
   */
  public int successor(TreeNode root) {
    root = root.right;
    while (root.left != null)
      root = root.left;
    return root.val;
  }

  /*
   * One step left and then always right
   */
  public int predecessor(TreeNode root) {
    root = root.left;
    while (root.right != null)
      root = root.right;
    return root.val;
  }

  public TreeNode deleteNode(TreeNode root, int key) {
    if (root == null)
      return null;

    // delete from the right subtree
    if (key > root.val)
      root.right = deleteNode(root.right, key);
    // delete from the left subtree
    else if (key < root.val)
      root.left = deleteNode(root.left, key);
    // delete the current node
    else {
      // the node is a leaf
      if (root.left == null && root.right == null)
        root = null;
      // the node is not a leaf and has a right child
      else if (root.right != null) {
        root.val = successor(root);
        root.right = deleteNode(root.right, root.val);
      }
      // the node is not a leaf, has no right child, and has a left child
      else {
        root.val = predecessor(root);
        root.left = deleteNode(root.left, root.val);
      }
    }
    return root;
  }
}
```

Reference
----------
1.[JDK 源码中 HashMap 的 hash 方法原理是什么？](https://www.zhihu.com/question/20733617)  
2.[HashMap工作原理和扩容机制](https://blog.csdn.net/u014532901/article/details/78936283)  
3.[Java HashMap工作原理及实现](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)  
4.[哈希表和完美哈希](https://www.cnblogs.com/gaochundong/p/hashtable_and_perfect_hashing.html)  