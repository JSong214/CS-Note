## Lecture22 (新)最小生成树（Minimum Spanning Trees, MSTs）

### Part 1: 什么是最小生成树 (MST)？



#### 1.1 生成树 (Spanning Tree)

想象一下，你有一些城市（图的**顶点**），以及连接这些城市的潜在道路（图的**边**），每条道路都有一个建设成本（边的**权重**）。现在，你的任务是修建一个道路网络，要求：

1. **连通所有城市**：从任何一个城市出发，都能到达其他任何城市。
2. **成本最低**：这里指的是使用的道路数量最少，即不能有环路。如果形成环路，就意味着有不止一条路径可以连接某些城市，这会造成浪费。

满足这两个条件的道路网络，就是一个**生成树 (Spanning Tree)**。

根据讲义的定义，对于一个无向图 G，它的一个生成树 T 是 G 的一个子图，并且 T 满足以下三个条件 ：

- **连通 (Connected)** ：树 T 内部的任意两个顶点都是相连的。
- **无环 (Acyclic)** ：树 T 中不包含任何环路。
- **包含所有顶点 (Spanning)** ：树 T 包含了原图 G 中所有的顶点。

下面这个图例可以帮助你理解，黑色的边和所有顶点共同构成了一个生成树 。

*图 1: 一个图和它的其中一个生成树（黑色边部分）。* 

![](https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/%E7%94%9F%E6%88%90%E6%A0%91.png)



#### 1.2 最小生成树 (Minimum Spanning Tree)

现在，我们给每条边赋予一个权重（比如建设成本、距离等）。在所有可能的生成树中，边的**总权重之和最小**的那一棵，就叫做**最小生成树 (Minimum Spanning Tree, MST)** 。

MST 在现实世界中有很多应用，例如：

- 设计成本最低的电信、电力或交通网络 。

- 医学图像分析，比如研究癌细胞核的排列方式 。

- 手写识别 。

  

#### 1.3 MST vs. 最短路径树 (Shortest Paths Tree, SPT)

这是一个常见的易混淆点。我们来区分一下：

- **目标不同**：

  - **MST** 的目标是连接**所有**顶点，并使总权重最小。它不关心任意两点间的路径有多长 。
  - **SPT** 的目标是找到从一个**特定源点**到**所有其他顶点**的最短路径 。它的结构和总权重会根据源点的不同而改变。

- **源点依赖**：

  - 一个图的 MST 通常是唯一的（如果所有边权重不同），它没有“源点”的概念 。

  - SPT 必须依赖一个指定的源点 。

    

注：一个图的 MST 并不总是等于它的任何一个 SPT 。

------



### Part 2: 切割属性 (The Cut Property)

切割属性是我们寻找 MST 的理论基础，也是后续两种算法（Prim 和 Kruskal）正确性的核心。



#### 2.1 什么是“切割” (Cut)？

一个**切割 (Cut)** 是指将一个图的所有顶点划分成两个**非空**的集合 。你可以想象用一把“剪刀”把图的顶点分成两部分。

一条**跨越切割的边 (Crossing Edge)** 是指连接这两个集合中顶点的一条边 。

*图 2: 灰色和白色顶点构成一个切割。连接灰色和白色顶点的边是跨越切割的边。* 

![](https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/%E5%88%87%E5%89%B2.png)





#### 2.2 切割属性 (Cut Property)

**切割属性**表明：对于图中的任意一个切割，权重最小的那条跨越切割的边，**必定**属于该图的某个最小生成树 。



**为什么这个属性是正确的？（证明思路）** 讲义中给出了一个简洁的反证法证明 ：

1. **假设**：权重最小的跨越边 `e` **不**在 MST 中。
2. **操作**：将边 `e` 添加到这个所谓的 "MST" 中。由于 MST 本身是连通的，添加 `e` 之后必然会形成一个环路 。
3. **发现**：这个环路中，除了 `e` 之外，必然还存在另一条跨越了同一个切割的边，我们称之为 `f` 。
4. **推理**：因为 `e` 是权重**最小**的跨越边，所以 `weight(e) < weight(f)`。
5. **结论**：我们可以从图中移除 `f` 并加入 `e`，得到一个新的、总权重更小的生成树 。这与我们最初假设的 MST 是“最小”的相矛盾。因此，原假设不成立，边 `e` 必须在 MST 中。

有了这个强大的理论工具，我们就可以设计出寻找 MST 的算法了。本质上，Prim 算法和 Kruskal 算法都是巧妙地利用切割属性来逐步构建 MST 的。

------



### Part 3: Prim 算法

Prim 算法是一种“贪心”算法，它从一个顶点开始，逐步扩大一棵树，直到它覆盖所有顶点。



#### 3.1 算法思想

Prim 算法的思路非常直观 ：

1. 选择任意一个顶点作为初始的“树”（这个树目前只包含一个顶点）。
2. 重复以下步骤直到树包含了所有 V 个顶点： a. 找到一条权重最小的边，这条边的一个顶点在当前的树中，另一个顶点不在树中。 b. 将这条边和它连接的新顶点加入到树中。

这个过程就像是在一个孤岛上，每次都选择修建一座最短的桥，去连接一个尚未到达的新岛屿。

**Prim 算法与切割属性的关系**： Prim 算法每一步都完美地应用了切割属性。在任意一步，我们都可以将图的顶点分为两部分：**已经在树中的顶点**和**还未在树中的顶点**。算法选择的那条权重最小的连接这两部分顶点的边，正是这条切割中权重最小的跨越边 。



#### 3.2 实现与优化 (类似 Dijkstra)

Prim 算法的实现与 Dijkstra 算法惊人地相似 。我们可以用一个**优先队列 (Priority Queue, PQ)** 来高效地找到那条“最短的连接边”。

**与 Dijkstra 的对比** ：

- **共同点**：两者都使用优先队列来选择下一个要访问的顶点，并且都有一个“松弛”(relaxation) 操作来更新顶点的某些属性。
- **不同点**：
  - **优先队列中存储的优先级不同**：
    - Dijkstra: 顶点到**源点**的距离 `distTo[v]`。
    - Prim: 顶点到**当前生成树**的最短距离 `distTo[v]`。
  - **松弛操作的更新逻辑不同**：
    - Dijkstra: 如果 `distTo[source] + weight(edge) < distTo[neighbor]`，则更新。
    - Prim: 如果 `weight(edge) < distTo[neighbor]`，则更新。因为它只关心新边本身的权重是否比之前记录的连接该邻居的边更短 。



#### 3.3 伪代码解析

下面是讲义中提供的伪代码，它和 Dijkstra 的结构几乎一样 。

```java
public class PrimMST {
    // edgeTo[v] = a v-t MST-ba vezető él
    private Edge[] edgeTo;
    // distTo[v] = a v csúcshoz legközelebbi él súlya
    private double[] distTo;
    // marked[v] = true, ha v az MST-ben van
    private boolean[] marked;
    // fringe: a még nem meglátogatott csúcsok prioritási sora
    private IndexMinPQ<Double> fringe;

    public PrimMST(EdgeWeightedGraph G) {
        edgeTo = new Edge[G.V()];
        distTo = new double[G.V()];
        marked = new boolean[G.V()];
        // Initialize all distances to infinity
        for (int v = 0; v < G.V(); v++) {
            distTo[v] = Double.POSITIVE_INFINITY;
        }
        fringe = new IndexMinPQ<Double>(G.V());

        // Start from vertex 0
        distTo[0] = 0.0;
        fringe.insert(0, 0.0); // Start with vertex 0, distance 0 to the tree

        while (!fringe.isEmpty()) {
            int v = fringe.delMin(); // Get vertex closest to the tree [cite: 515]
            scan(G, v);              // Add v to the tree and relax its edges [cite: 515]
        }
    }

    private void scan(EdgeWeightedGraph G, int v) {
        marked[v] = true; // Add v to the MST [cite: 526, 534]
        for (Edge e : G.adj(v)) {
            int w = e.other(v);
            if (marked[w]) continue; // Skip if w is already in MST [cite: 529, 535]

            // Edge e is a better connection for w to the tree
            if (e.weight() < distTo[w]) { [cite: 530]
                distTo[w] = e.weight();  // Update distance to tree [cite: 531]
                edgeTo[w] = e;           // Update best edge to connect w [cite: 532]
                
                // Update priority in the fringe
                if (fringe.contains(w)) {
                    fringe.decreaseKey(w, distTo[w]); // Use decreaseKey for efficiency [cite: 533]
                } else {
                    fringe.insert(w, distTo[w]);
                }
            }
        }
    }
}
```



#### 3.4 运行时间

Prim 算法的运行时间主要取决于优先队列的操作效率 。

- 每个顶点入队和出队一次 (`V` 次 `insert` 和 `V` 次 `delMin`)。
- 每条边最多被考虑一次，可能触发一次 `decreasePriority` 操作 (`E` 次)。

如果使用基于二叉堆的优先队列，这些操作的成本都是 O(logV)。因此，总时间复杂度为 O(VlogV+ElogV)。在连通图中，E≥V−1，所以通常简化为 **O(ElogV)** 。



------



### Part 4: Kruskal 算法

Kruskal 算法是另一种寻找 MST 的贪心算法。它的策略与 Prim 不同，它关注的是边，而不是顶点。



#### 4.1 算法思想

Kruskal 的方法是，将所有边看作是独立的“森林”，然后逐步将这些森林合并成一棵大树 。

1. 创建一个包含所有 V 个顶点的列表，但初始时没有任何边。此时，每个顶点都是一个独立的树/连通分量。
2. 将图中所有的边按权重**从小到大**排序。
3. 遍历排序后的边列表： a. 取出当前权重最小的边。 b. **检查**这条边连接的两个顶点是否已经属于同一个连通分量（即是否已经连通）。 c. 如果不连通，就将这条边加入 MST。这会合并两个不同的连通分量。 d. 如果已经连通，则跳过这条边，因为它会形成环路。
4. 重复步骤3，直到 MST 中包含了 V-1 条边。



#### 4.2 实现与优化 (使用 Union-Find)

Kruskal 算法的关键在于如何高效地“**检查两个顶点是否连通**”以及“**合并两个连通分量**”。这正是 **Union-Find** (也叫 Disjoint Sets) 数据结构的专长。

- `find(v)`: 确定顶点 `v` 属于哪个集合。
- `union(v, w)`: 合并包含 `v` 和 `w` 的两个集合。
- `connected(v, w)`: 通过检查 `find(v) == find(w)` 来判断它们是否在同一个集合中。

使用带路径压缩和按权重合并优化的 Union-Find 结构，`union` 和 `connected` 操作的平均时间复杂度接近常数，记为 O(α(V))，其中 α(V) 是反阿克曼函数，增长极其缓慢 。



#### 4.3 伪代码解析

讲义中给出的伪代码清晰地展示了这一过程 。

```java
public class KruskalMST {
    private List<Edge> mst = new ArrayList<Edge>();

    public KruskalMST(EdgeWeightedGraph G) {
        // 1. Create a min-priority queue and add all edges [cite: 689]
        MinPQ<Edge> pq = new MinPQ<Edge>();
        for (Edge e : G.edges()) {
            pq.insert(e); [cite: 691]
        }

        // 2. Create a Union-Find data structure [cite: 692]
        WeightedQuickUnionPC uf = new WeightedQuickUnionPC(G.V());

        // 3. Iterate while MST has fewer than V-1 edges [cite: 694]
        while (!pq.isEmpty() && mst.size() < G.V() - 1) {
            // Get the edge with the smallest weight
            Edge e = pq.delMin(); [cite: 695]
            int v = e.either();
            int w = e.other(v);

            // 4. If v and w are not already connected, add the edge
            if (!uf.connected(v, w)) { [cite: 698]
                uf.union(v, w);      // Union the two components [cite: 699]
                mst.add(e);          // Add edge to MST [cite: 700]
            }
        }
    }
}
```



#### 4.4 运行时间

Kruskal 算法的瓶颈在于：

1. **对所有边进行排序**：如果使用优先队列来逐个取出最小边，相当于排序。将 E 条边全部插入优先队列的成本是 O(ElogE) 。
2. **Union-Find 操作**：对 E 条边进行 `connected` 检查，以及最多 V-1 次 `union` 操作。总成本约为 O(Elog∗V) 或 O(Eα(V)) 。

由于 E 通常小于 V2，所以 logE 与 logV 是同阶的 (O(logE)=O(logV)) 。因此，总的时间复杂度主要由排序决定，为 **O(ElogE)** 或 **O(ElogV)** 。

**一个重要的优化**：如果边已经预先排序，或者我们可以使用线性时间排序（如计数排序，当权重为小整数时），那么 Union-Find 操作将成为瓶颈，此时时间复杂度可以降低到 O(Eα(V)) 。

------



### Part 5: Prim vs. Kruskal 总结

| 特性           | Prim 算法                                                    | Kruskal 算法                                                 |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **基本思想**   | 从一个点开始，逐步生长一棵树，每次都选择离树最近的顶点加入。 | 将边排序，从小到大依次尝试加入，只要不形成环路就行。         |
| **数据结构**   | 优先队列 (Priority Queue)                                    | Union-Find + 优先队列 (或预排序数组)                         |
| **图的形态**   | MST 始终是**一棵连通的树**。                                 | MST 在构建过程中是**一个森林**，最后才连接成一棵树。         |
| **时间复杂度** | O(ElogV)                                                     | O(ElogE) 或 O(ElogV)                                         |
| **适用场景**   | 对于**稠密图**（E 接近 V2），Prim 算法可能更有优势。         | 对于**稀疏图**（E 接近 V），Kruskal 算法通常表现更好，特别是当边已排序时。 |

讲义中的这张可视化对比图非常形象地展示了两种算法的区别 ：

- **Prim**: 像一个不断扩张的墨迹。
- **Kruskal**: 像许多分散的小水滴，通过最短的桥梁逐渐汇合。