## Lecture24 前缀树 (Tries)



### 1. Trie 的概念介绍

#### 什么是 Trie？

Trie，发音同 "try" ，其全称是 "Retrieval Tree" (检索树) 。它是一种专门为存储和检索字符串键而设计的树形数据结构。与平衡二叉搜索树 (BST) 或哈希表 (Hash Table) 不同，Trie 并不在节点中存储整个键。相反，它将键的各个字符分散存储在从根到叶节点的路径上。



**核心思想：**

- **每个节点只存储一个字符**：从根节点到某个节点的路径构成了一个字符串（前缀）。
- **共享前缀**：多个键可以共享相同的节点路径。例如，"sad", "sam", "sap" 这三个单词会共享代表 "s" 和 "a" 的节点。



#### Trie vs. BST vs. 哈希表

让我们来看一个例子。假设我们要存储一个集合，包含 {"sam", "sad", "sap", "same", "a", "awls"}。

- 在 **BST** 中，节点存储完整的字符串，并根据字符串的字典序进行排列。
- 在 **哈希表** 中，字符串通过哈希函数映射到数组的不同索引上，可能会有冲突。
- 在 **Trie** 中，结构如下所示：



图 1: 包含 "a", "awls", "sad", "sam", "same", "sap" 的 Trie 结构

<img src="https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/Trie%20%E7%BB%93%E6%9E%84.png" style="zoom:50%;" />





在这棵 Trie 中：

- 从根节点出发，路径 `s -> a -> m` 代表了 "sam"。
- 路径 `s -> a` 是 "sad", "sam", "sap", "same" 的共同前缀，因此它们共享 `(s)` 和 `(a)` 这两个节点。



#### 如何区分键和前缀？

你可能会问，既然 "sa" 是一个路径，我们如何知道它本身不是一个键，而 "sad" 是一个键呢？

Trie 通过在节点上做一个标记来解决这个问题。我们可以把代表一个完整键的最后一个字符所在的节点涂成“蓝色”（或设置一个 `isKey` 布尔值为 `true`）。



图 2: 蓝色的节点表示其对应的前缀是一个完整的键  

<img src="https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/Trie%20%E7%BB%93%E6%9E%84.png" style="zoom:50%;" />

基于这个标记，我们可以轻松判断一个字符串是否存在于集合中：

- **`contains("sam")`**: 我们沿着路径 `s -> a -> m` 走，最终到达的 `(m)` 节点是蓝色的。返回 `true`。
- **`contains("sa")`**: 我们沿着路径 `s -> a` 走，最终到达的 `(a)` 节点是白色的。返回 `false`。
- **`contains("sax")`**: 我们走到 `s -> a` 后，发现没有指向 `x` 的分支。我们称之为“从树上掉下来了”(fell off the tree)。返回 `false`。

Trie 同样可以作为 **Map** 使用，只需在标记为键的节点上额外存储对应的值 (value) 即可。

------



### 2. Trie 的节点实现与性能



#### 节点内部结构

一个 Trie 节点需要包含两个核心信息：

1. 一个标记，用于判断当前节点是否代表一个键的结尾 (`isKey`)。
2. 一个指向其所有子节点的引用集合。

一个常见的实现方式如下：

```java
private static class Node {
    private boolean isKey; // 标记是否为键的结尾
    // 用于存储子节点的映射，键是字符，值是子节点
    private Map<Character, Node> next; 
    
    public Node(boolean isKey) {
        this.isKey = isKey;
        this.next = new HashMap<>(); // 或者其他 Map 实现
    }
}
```

> **注意**：讲义中提到，节点本身存储的字符 `char ch` 其实是多余的，因为这个字符信息已经隐含在从父节点指向它的链接中了（例如，父节点的 `next` Map 中的键就是这个字符）。



#### 子节点跟踪策略 (Child Tracking Strategies)

如何实现 `next` 这个 Map 是 Trie 实现的关键，直接影响其时空效率。主要有三种策略：



**1. DataIndexedCharMap (基于数组的实现)**

- **思想**: 使用一个固定大小的数组来存储所有可能的子节点链接。例如，如果字符集是 ASCII，就创建一个长度为 128 的数组。
- **优点**: **速度极快**。查找特定字符的子节点是 O(1) 的数组访问。
- **缺点**: **非常消耗内存**。每个节点都需要一个长度为 R (字符集大小) 的数组，即使它只有很少的子节点，也会导致大量的空间浪费（大部分链接为 null）。



图 3: 基于数组的节点实现，其中大部分指针都为 null  

![](https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/Trie%E5%9F%BA%E4%BA%8E%E6%95%B0%E7%BB%84%E5%AE%9E%E7%8E%B0.png)



**2. Hash Table (基于哈希表的实现)**

- **思想**: 使用哈希表 (如 Java 的 `HashMap`) 来存储从字符到子节点的映射。
- **优点**: **内存效率高**，因为它只存储实际存在的子节点链接。访问速度也很快，平均为 O(1)。
- **缺点**: 比数组稍慢，因为需要计算哈希值，且在最坏情况下可能退化。



图 4: 基于哈希表的节点实现  

<img src="https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/Trie%E5%9F%BA%E4%BA%8E%E5%93%88%E5%B8%8C%E8%A1%A8%E5%AE%9E%E7%8E%B0.png" style="zoom:50%;" />



**3. Balanced BST (基于平衡二叉搜索树的实现)**

- **思想**: 使用平衡 BST (如红黑树) 来存储子节点链接。
- **优点**: 内存使用与哈希表类似，只存储存在的链接。
- **缺点**: 查找子节点的时间复杂度为 O(logR)，其中 R 是字符集大小，比前两者慢。



图 5: 基于 BST 的节点实现  

<img src="https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/Trie%E5%9F%BA%E4%BA%8EBST%E5%AE%9E%E7%8E%B0.png" style="zoom:50%;" />



#### 性能分析

Trie 的 `add` 和 `contains` 操作性能取决于键的长度，我们用 **L** 表示。

- **`add(key)` / `contains(key)`**: 操作需要从根节点开始，沿着键的字符路径遍历 L 个节点。在每个节点，查找下一个字符的链接所需的时间取决于我们选择的子节点跟踪策略。
  - **DataIndexedCharMap**: O(L)，因为每次查找都是 O(1)。
  - **Hash Table**: 平均 O(L)。
  - **BST**: O(LlogR)。

由于字符集大小 R 通常被视为一个常数（例如 ASCII 的 128 或 256），因此 O(logR) 也可以看作是常数时间。所以，在多数情况下，我们可以认为 Trie 的 `add` 和 `contains` 操作的**时间复杂度为 Θ(L)**。



如果我们将键的长度也看作一个常数（在很多应用中是合理的），那么 Trie 的操作可以达到 **Θ(1)**，这比 BST 的 Θ(logN)（N 是键的数量）更快。

------



### 3. Trie 的字符串操作

Trie 真正的优势在于它能高效地支持标准数据结构难以实现的一些特殊的字符串操作，特别是**前缀匹配 (prefix matching)**。



#### `keysWithPrefix(String prefix)`

这个操作返回 Trie 中所有以 `prefix` 开头的键。

**算法步骤:** 

1. **导航到前缀节点**: 从根节点开始，沿着 `prefix` 的字符路径向下遍历，找到代表该前缀的最后一个节点（下图中的粉色节点 `a`）。如果中途“掉下树”，说明没有任何键以此为前缀，返回空列表。
2. **收集所有后续键**: 从该前缀节点开始，执行一个类似于 `collect()` (收集所有键) 的递归遍历，找出其下的所有完整键，并将它们添加到结果列表中。



图 6: 执行 keysWithPrefix("sa") 的过程  

<img src="https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/Trie%E5%89%8D%E7%BC%80%E5%8C%B9%E9%85%8D.png" style="zoom:50%;" />

`collect()` 的递归辅助函数 `colHelp` 逻辑如下：

```
colHelp(String currentString, List<String> results, Node n):
    // 如果当前节点是一个键的结尾，将其对应的字符串加入结果列表
    if n.isKey:
        results.add(currentString)
    
    // 遍历当前节点的所有子节点
    for each character c in n.next.keys():
        // 递归调用，传入更新后的字符串和子节点
        colHelp(currentString + c, results, n.next.get(c))
```

`keysWithPrefix` 只需在前缀节点上调用这个辅助函数即可。



#### `longestPrefixOf(String query)`

这个操作是在 Trie 中查找一个最长的键，该键是 `query` 字符串的前缀。

- **例如**: Trie 中有 {"a", "awls", "sam"}，执行 `longestPrefixOf("sample")`。
- `query` "sample" 的前缀有 "s", "sa", "sam", "samp", ...
- Trie 中存在的键且是 "sample" 前缀的有 "a" 和 "sam"。
- 最长的一个是 "sam"，所以结果为 "sam"。



#### 应用：自动补全 (Autocomplete)

自动补全是 Trie 的一个典型应用。 当用户在搜索框输入时，系统需要快速推荐匹配的查询。

**基本思路**：

1. 将海量搜索查询及其权重（例如搜索频率）存入一个 Trie Map。

2. 当用户输入字符串 `prefix` 时，调用 `keysWithPrefix(prefix)`。

3. 从返回的结果中，选出权重最高的 K 个作为推荐。

   

**效率优化**： 当 `prefix` 很短时（如 "b"），匹配的键可能有数十亿个，全部找出再排序非常低效。

一个更高效的方法是**在 Trie 节点中预先存储额外信息**。 每个节点除了保存自己的值，还保存其所有子孙节点中的**最高权重 (best)**。



图 7: 节点中存储了其子树中的最高权重  

<img src="https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/Trie%20%E6%9C%80%E9%AB%98%E6%9D%83%E9%87%8D.png" style="zoom:67%;" />



这样，在搜索时，我们可以优先遍历那些 `best` 值更高的分支（例如使用优先队列），并且当发现当前找到的 Top-K 结果的权重已经全部高于某个分支的 `best` 值时，就可以安全地剪掉那个分支，极大地提高了搜索效率。

------



### 4. 知识点总结

1. **Trie 是什么**：一种专用于字符串键的检索树，通过路径共享来节省空间，并能高效执行前缀相关的操作。
2. **核心优势**：在 `add` 和 `contains` 上性能优于或持平于哈希表（Θ(L)），但真正强大之处在于高效支持 `keysWithPrefix` 和 `longestPrefixOf` 等特殊字符串操作。
3. **实现选择**：节点中如何存储子节点是关键。
   - **数组 (DataIndexedCharMap)**：速度最快，但内存开销巨大。
   - **哈希表 (Hash Table)**：速度快且内存高效，通常是比较均衡和自然的选择。
   - **平衡树 (Balanced BST)**：速度稍慢，内存使用与哈希表相当。
4. **重要应用**：自动补全系统是 Trie 的经典应用场景，通过 `keysWithPrefix` 实现，并可通过在节点中缓存额外信息（如最高权重）进行深度优化。