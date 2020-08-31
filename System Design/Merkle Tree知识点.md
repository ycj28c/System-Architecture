Merkle Tree的相关知识点

## 什么是Merkle Tree
Merkle Tree也就是胜超前文说的Merkle树，中文译名还有梅克尔树或默克尔树，因为这是一棵用哈希值搭建起来的树，树的所有节点都存储了哈希值，所以也叫哈希树，英文名为Ｈash Tree。Merkle树是一种典型的二叉树结构，看起来像一棵倒着的树，由一个根节点，一组中间节点和一组叶节点组成，最早由Merkle Ralf在1980年提出，广泛用于文件系统和 P2P 系统中。

## 如何理解Merkle树
结构很简单，就是普通的BST二叉树，不过节点代表的意义不同
```
       H(ABCD)
	     /\
  H(AB)      H(CD)
    /\         /\  
 H(A) H(B)  H(C) H(D)
  |     |    |    |
Data1 Data2 Data3 Data4
 
其中A,B,C,D就是表示，
H(*)代表了是一个Hash值，
而Data1-4就是实际的数据块了。
```	 
叶节点：在二叉树中，没有子节点的节点称为叶节点，这是初始节点，对于一个区块而言，每一笔交易数据，进行哈希运算后，得到的哈希值就是叶节点。  
中间节点：子节点两两匹配，子节点哈希值合并成新的字符串，对合并结果再次进行哈希运算，得到的哈希值，就是对应的中间节点，这是过程节点。  
根节点：有且只有一个，也就是胜超之前分享的Merkle 根，也叫Ｍerkle Root，这是终止节点。  

## Merkle如何运作
以上面的图为例：  
为了验证数据[Data4] 的根哈希，我们可以使用单向函数哈希[Data4] 来取得H(D).  
当H(D) 和未知的数据组C哈希时，会产生H(CD).  
H(CD)与H(AB) 哈希得到H(ABCD).它正好是公共的merkle根.  

因此，在不显示D或任何数据的情况下，我们通过使用H(C), H(AB), H(ABCD)就证明了数据组Data4确实在merkle树里。

## Merkle Tree用途
1，Merkle树比较典型的应用场景的就是P2P下载，在点对点网络中作数据传输的时候，为了校验数据的完整性，把大的文件分割成小的数据块，如果小块数据在传输过程中损坏了，那么只要重新下载这一小块数据就行了。  
2，Merkle树还可以被用来快速比较大量的数据，因为当两个Merkle树根相同时，则意味着所代表的数据必然相同。  
3，数字签名，Merkle树被广泛地用来验证大数据组和大多数区块链应用的包含性，它使得说谎或作假根本不可能发生，保证了数据的真实和有效性。Merkle Tree 拓展了单向哈希的应用。Merkle proof 是通过把子哈希连在一起然后再计算哈希的方式一直递归向上，直到得到根哈希值，作为公钥。单向哈希算法不会产生碰撞，并且是确定性算法，不会有两个明文的哈希值相同的情况。

另外在系统设计很重要的应用：  
1.处理replica  
Merkle Tree，每个节点，对于自己存储的每一段key range，建立merkle tree，通过与另外一个replica比较一段range的hash值来决定是否需要sync，而不需要传播和比较所有值。  
2.处理partition  
Merkle Tree的特点使得我们需要Partition 3的操作：先把key range分bucket。否则一旦有新的node加入进来，在转移data的同时，我们需要扫描data，重新进行hash的计算，因为data partitioning和merkle tree的key range partitioning并不一致。而如果我们通过分bucket让他们保持一致，则只需要把merkle tree的一部分子树转移到另一个节点上，并重新计算一下向上的根结点的hash就可以了。  

## Reference
[叶胜超：一分钟搞懂Merkle Tree以及它的特点和作用！](https://zhuanlan.zhihu.com/p/84019208)  
[Merkle Tree- 哈希树证明的简单概念/原理/用途分析](https://steemit.com/blockchain/@susanli3769/merkle-tree)  
[学习资料 读论文 Amazon Dynamo](https://www.1point3acres.com/bbs/forum.php?mod=viewthread&tid=505467&extra=page%3D1)