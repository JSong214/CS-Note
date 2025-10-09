## Lecture12 不相交集（Disjoint Sets）



### 1. Disjoint Sets 简介 (Introduction)

想象一下，你有一堆元素，比如 0, 1, 2, 3, ... , 9。最初，它们各自独立，每个元素都属于自己的一个集合。

**不相交集（Disjoint Sets）** 这个数据结构的核心任务就是维护这些集合，并支持两种基本操作：

1. `connect(p, q)` 或 `union(p, q)`：**连接**。将元素 `p` 所在的集合与元素 `q` 所在的集合合并成一个集合。
2. `isConnected(p, q)` 或 `find(p) == find(q)`：**查询**。判断元素 `p` 和 `q` 是否属于同一个集合。

**应用场景**： 最经典的应用就是**网络连通性 (Network Connectivity)**。 假设你有一堆计算机，编号为 0 到 N-1。`connect(p, q)` 就表示在计算机 `p` 和 `q` 之间拉一根网线。`isConnected(p, q)` 就是查询计算机 `p` 和 `q` 能否通过网络互相通信。

上图中，{0, 1, 2, 3, 4} 是一个连通的集合，{5, 6} 是另一个，而 {7}, {8}, {9} 各自独立。在这种状态下，`isConnected(0, 4)` 的结果是 `true`，而 `isConnected(0, 5)` 的结果是 `false`。

现在，我们来探索如何用代码实现这个想法。

------



### 2. Quick Find: 最直观的实现

这是我们的第一次尝试。我们如何表示“谁和谁在同一个集合”呢？一个非常自然的想法是：给每个集合一个唯一的 ID。

**数据结构**： 我们使用一个整型数组 `id[]`。`id[i]` 的值就代表元素 `i` 所属集合的 ID。

```java
public class QuickFindDS {
    private int[] id;

    public QuickFindDS(int N) {
        id = new int[N];
        for (int i = 0; i < N; i++) {
            id[i] = i; // 初始时，每个元素自成一个集合
        }
    }
}
```

初始状态下，`id[0]=0, id[1]=1, ...`，表示每个元素都在自己的集合里。

**操作实现**：

- **`isConnected(p, q)`** 这个操作非常简单：只要检查 `p` 和 `q` 的集合 ID 是否相同即可。

  ```java
  public boolean isConnected(int p, int q) {
      return id[p] == id[q];
  }
  ```

  这个操作只需要访问数组两次，所以它的时间复杂度是 O(1)，非常快！

- **`connect(p, q)`** 要合并 `p` 和 `q` 所在的集合，我们就需要将其中一个集合的所有成员的 ID，全部改成另一个集合的 ID。 例如，要 `connect(1, 6)`，假设 `id[1]` 是 1，`id[6]` 是 6。我们就需要遍历整个 `id` 数组，把所有值为 1 的元素都改成 6。

  ```java
  public void connect(int p, int q) {
      int pid = id[p];
      int qid = id[q];
  
      // 如果已经在同一个集合，则无需操作
      if (pid == qid) {
          return;
      }
  
      // 遍历整个数组，将所有属于 p 集合的元素，都归到 q 集合中
      for (int i = 0; i < id.length; i++) {
          if (id[i] == pid) {
              id[i] = qid;
          }
      }
  }
  ```

  这个操作的代价很大！每次 `connect` 都需要遍历整个数组，所以它的时间复杂度是 O(N)，其中 N 是元素的总数。

**小结**： Quick Find 的 `isConnected` 操作（查询）非常快，但 `connect` 操作（连接）太慢了。如果我们需要执行大量的连接操作，整个程序的性能会变得非常差。我们需要找到更好的方法。

------



### 3. Quick Union: 一种巧妙的改进

既然 Quick Find 的 `connect` 操作慢，我们能不能换一种数据表示方式来优化它呢？

**核心思想**： 我们不再让 `id[i]` 直接表示集合 ID，而是让它表示 `i` 的**父节点**。这样，所有元素就构成了一片森林（多个树）。同一个集合的元素必然在同一棵树中，这棵树的**根节点**就是这个集合的唯一标识。

**数据结构**： 我们还是用一个数组，但现在我们叫它 `parent[]` 更贴切。`parent[i]` 存储的是元素 `i` 的父节点。如果 `parent[i] == i`，说明 `i` 是它所在树的根节点。

```java
public class QuickUnionDS {
    private int[] parent;

    public QuickUnionDS(int N) {
        parent = new int[N];
        for (int i = 0; i < N; i++) {
            parent[i] = i; // 初始时，每个元素都是自己的父节点，即自成一棵树
        }
    }
}
```

**操作实现**：

- **`find(p)`** 这是一个辅助函数，用于找到元素 `p` 所在树的根节点。我们只需要不断地向上查找，直到找到一个父节点是自己的元素。

  ```java
  private int find(int p) {
      while (p != parent[p]) {
          p = parent[p];
      }
      return p;
  }
  ```

- **`isConnected(p, q)`** 判断 `p` 和 `q` 是否连通，就等价于判断它们是否在同一棵树里，也就是它们是否有相同的根节点。

  ```java
  public boolean isConnected(int p, int q) {
      return find(p) == find(q);
  }
  ```

- **`connect(p, q)`** 连接 `p` 和 `q`，就是将一棵树接到另一棵树上。我们只需要找到它们各自的根节点，然后将一个根节点的父指针指向另一个根节点即可。

  ```java
  public void connect(int p, int q) {
      int rootP = find(p);
      int rootQ = find(q);
  
      if (rootP == rootQ) {
          return;
      }
  
      parent[rootP] = rootQ; // 将 p 所在树的根，挂到 q 所在树的根上
  }
  ```

**性能分析**： `connect` 操作本身很快，只需要两次 `find` 和一次数组赋值。关键在于 `find` 操作的耗时。在最坏的情况下，这棵树可能会退化成一条链（比如依次 `connect(0,1)`, `connect(1,2)`, `connect(2,3)`...），树的高度会达到 N。

- **`connect`** 和 **`isConnected`** 的时间复杂度都取决于 `find` 的深度，最坏情况下是 O(N)。

**小结**： Quick Union 优化了 `connect` 操作的本身，但引入了 `find` 的成本。虽然在平均情况下它比 Quick Find 好，但在最坏情况下，性能依然很差。问题出在树的形态不可控，可能会变得又高又瘦。

------



### 4. Weighted Quick Union (WQU): 避免最坏情况

如何防止树变得又高又瘦呢？答案是：在合并两棵树时，我们总是将**小树**连接到**大树**的根上，而不是随意连接。这样可以有效地控制树的高度。

**数据结构**： 我们需要一个额外的数组 `size[]` 来记录每棵树的大小（即节点数）。`size[i]` 存储的是以 `i` 为根的树中元素的总数。

```java
public class WeightedQuickUnionDS {
    private int[] parent;
    private int[] size;

    public WeightedQuickUnionDS(int N) {
        parent = new int[N];
        size = new int[N];
        for (int i = 0; i < N; i++) {
            parent[i] = i;
            size[i] = 1; // 初始时，每棵树的大小都为 1
        }
    }
    // find 和 isConnected 方法与 QuickUnion 完全相同
    private int find(int p) { /* ... */ }
    public boolean isConnected(int p, int q) { /* ... */ }
}
```

**`connect` 操作的改进**：

```java
public void connect(int p, int q) {
    int rootP = find(p);
    int rootQ = find(q);

    if (rootP == rootQ) {
        return;
    }

    // 核心改进：比较两棵树的大小
    if (size[rootP] < size[rootQ]) {
        // p 树更小，将 p 树连接到 q 树上
        parent[rootP] = rootQ;
        size[rootQ] += size[rootP]; // 更新 q 树的大小
    } else {
        // q 树更小或一样大，将 q 树连接到 p 树上
        parent[rootQ] = rootP;
        size[rootP] += size[rootQ]; // 更新 p 树的大小
    }
}
```

**性能分析**： 通过这个简单的“加权”策略，可以证明，对于一个含有 N 个元素的集合，任何节点的深度都不会超过 log_2N。

- `find`, `connect`, `isConnected` 的时间复杂度都优化到了 O(logN)。

这是一个巨大的进步！对于一百万个元素，logN 大约是 20，而 N 是 1,000,000。性能提升显而易见。

------



### 5. 终极优化：路径压缩 (Path Compression)

我们已经做得很好了，但还能更好吗？答案是可以的！ 当我们调用 `find(p)` 查找根节点时，我们会经过一条从 `p` 到根的路径。这条路径上的所有节点，最终都指向同一个根。那么，我们何不“顺手”把这条路径上的所有节点的父指针，都直接指向根呢？

这就是**路径压缩**。它能极大地扁平化我们的树。

**`find` 操作的终极改进**： 一个简单的实现方式是使用递归。

```java
private int find(int p) {
    if (p == parent[p]) {
        return p; // 找到了根
    }
    // 递归查找根，并将路径上所有节点的 parent 直接指向根
    parent[p] = find(parent[p]);
    return parent[p];
}
```

**性能分析**： 当我们同时使用 **Weighted Quick Union** 和 **Path Compression** 时，`find` 操作的摊还时间复杂度 (Amortized Time Complexity) 接近常数时间，记为 O(α(N))，其中 α(N) 是反阿克曼函数。这个函数的增长速度极其缓慢，对于我们能想象到的任何实际输入的 N 值，α(N) 都不会超过 5。因此，我们可以认为它几乎是 O(1) 的。



### 总结与回顾

我们来整理一下今天学习的四种 Disjoint Sets 实现的性能对比：

| 实现方法                   | `connect(p, q)` | `isConnected(p, q)` | 备注                                   |
| -------------------------- | --------------- | ------------------- | -------------------------------------- |
| Quick Find                 | O(N)            | O(1)                | 连接操作太慢。                         |
| Quick Union                | O(N)*           | O(N)*               | 树可能退化成链状，性能不稳定。         |
| Weighted Quick Union (WQU) | O(logN)         | O(logN)             | 保证树的平衡，性能稳定。               |
| WQU with Path Compression  | O(α(N))         | O(α(N))             | 终极形态，接近常数时间，实际应用首选。 |

Export to Sheets

*注：最坏情况下的复杂度。*