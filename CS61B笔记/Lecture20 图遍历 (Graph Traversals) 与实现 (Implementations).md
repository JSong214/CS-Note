## Lecture20 图遍历 (Graph Traversals) 与实现 (Implementations)



### 1. 图的核心概念与表示方法 (Graph Representations)

在讨论遍历之前，我们必须先确定图的结构。一个图由 **顶点 (Vertices)** 和 **边 (Edges)** 组成。我们用 `V` 表示顶点的集合，`E` 表示边的集合。

要在程序中表示图，最常见的两种方式是 **邻接表 (Adjacency List)** 和 **邻接矩阵 (Adjacency Matrix)**。



#### 邻接矩阵 (Adjacency Matrix)

邻接矩阵是一个 V×V 的二维数组。如果顶点 `i` 和顶点 `j` 之间有一条边，那么矩阵中 `adj[i][j]` 的值通常为 1（或边的权重），否则为 0。

**图例:**

考虑下面这个无向图：

```
      (0)
     /   \
    /     \
  (1)-----(2)
   |       |
   |       |
  (3)-----(4)
```

它的邻接矩阵表示为：

```
  //   0  1  2  3  4
  // ----------------
  // 0|  0  1  1  0  0
  // 1|  1  0  1  1  0
  // 2|  1  1  0  0  1
  // 3|  0  1  0  0  1
  // 4|  0  0  1  1  0
```

- **优点**：
  - 检查两个顶点之间是否存在边非常快，只需要 O(1) 的时间复杂度。
- **缺点**：
  - 空间复杂度高，为 O(V2)。如果图是 **稀疏的**（边的数量远小于 V2），会浪费大量空间。
  - 获取一个顶点的所有邻居需要遍历一整行，时间复杂度为 O(V)。



#### 邻接表 (Adjacency List)

邻接表使用一个数组（或哈希表），数组的每个位置 `i` 对应一个链表（或动态数组），这个链表存储了所有与顶点 `i` 相邻的顶点。

**图例:**

对于同一个图：

```
  0: -> 1 -> 2
  1: -> 0 -> 2 -> 3
  2: -> 0 -> 1 -> 4
  3: -> 1 -> 4
  4: -> 2 -> 3
```

- **优点**：
  - 空间复杂度低，为 O(V+E)，非常适合表示稀疏图。
  - 获取一个顶点的所有邻居非常高效。
- **缺点**：
  - 检查两个顶点 `u` 和 `v` 之间是否存在边，需要遍历 `u` 的邻接链表，时间复杂度最差为 O(degree(u))，其中 degree(u) 是顶点 `u` 的度（邻居数量）。

**CS61B 重点**: 在大多数情况下，特别是对于稀疏图，**邻接表是更常用和更高效的选择**。我们接下来的讨论和代码实现都将基于邻接表。

------



### 2. 广度优先搜索 (Breadth-First Search, BFS)

BFS 的核心思想是 **逐层探索**。从一个起始顶点 `s` 开始，首先访问所有与 `s` 直接相连的邻居，然后访问与这些邻居直接相连的、尚未被访问过的顶点，以此类推，直到所有可达的顶点都被访问。

想象一下在平静的湖面上扔下一颗石子，BFS 的过程就像是水波纹一圈一圈地向外扩散。



#### 关键数据结构：队列 (Queue)

为了实现“逐层”访问，BFS 需要一个“待办事项”列表，这个列表必须满足 **先进先出 (First-In, First-Out)** 的特性，这正是 **队列** 的功能。我们称这个待访问的顶点集合为 **Fringe** (边缘集合)。



#### BFS 算法流程

1. 创建一个队列 `fringe`，并将起始顶点 `s` 加入队列。
2. 创建一个集合 `visited` 用于记录已访问过的顶点，将 `s` 加入 `visited`。
3. 当 `fringe` 不为空时，循环执行： a. 从 `fringe` 的队头取出一个顶点 `v`。 b. **处理 `v`** (例如，打印它)。 c. 遍历 `v` 的所有邻居 `u`： i. 如果 `u` **没有**在 `visited` 集合中： * 将 `u` 加入 `visited` 集合。 * 将 `u` 加入 `fringe` 队列。



#### 图解 BFS 过程

我们以上面的图为例，从顶点 `0` 开始进行 BFS：

1. **初始状态**:
   - `fringe` (Queue): `[0]`
   - `visited`: `{0}`
   - `访问顺序`: ``
2. **步骤 1**: 队头 `0` 出队。处理 `0`。将 `0` 的未访问邻居 `1`, `2` 入队。
   - `fringe`: `[1, 2]`
   - `visited`: `{0, 1, 2}`
   - `访问顺序`: `0`
3. **步骤 2**: 队头 `1` 出队。处理 `1`。将 `1` 的未访问邻居 `3` 入队 (`0` 和 `2` 已被访问)。
   - `fringe`: `[2, 3]`
   - `visited`: `{0, 1, 2, 3}`
   - `访问顺序`: `0, 1`
4. **步骤 3**: 队头 `2` 出队。处理 `2`。将 `2` 的未访问邻居 `4` 入队 (`0` 和 `1` 已被访问)。
   - `fringe`: `[3, 4]`
   - `visited`: `{0, 1, 2, 3, 4}`
   - `访问顺序`: `0, 1, 2`
5. **步骤 4**: 队头 `3` 出队。处理 `3`。`3` 的邻居 `1` 和 `4` 都已被访问。
   - `fringe`: `[4]`
   - `visited`: `{0, 1, 2, 3, 4}`
   - `访问顺序`: `0, 1, 2, 3`
6. **步骤 5**: 队头 `4` 出队。处理 `4`。`4` 的邻居 `2` 和 `3` 都已被访问。
   - `fringe`: `[]` (空)
   - `visited`: `{0, 1, 2, 3, 4}`
   - `访问顺序`: `0, 1, 2, 3, 4`
7. `fringe` 为空，遍历结束。最终访问顺序为 `0, 1, 2, 3, 4`。

**核心应用**: BFS 最重要的应用之一是在 **无权图** 中寻找 **最短路径**。因为它是按层级遍历的，所以当它第一次到达某个目标顶点时，所经过的路径一定是层数最少的，也就是最短的。

------



### 3. 深度优先搜索 (Depth-First Search, DFS)

与 BFS 不同，DFS 的核心思想是 **“一条路走到黑”**。从起始顶点 `s` 开始，选择一个邻居，然后沿着这个邻居的路径一直深入下去，直到无法再前进（到达一个没有未访问邻居的顶点）时，才 **回溯 (backtrack)** 到上一个顶点，选择另一条路继续探索。

这就像在走迷宫，你总是沿着一条路走到死胡同，然后才退回来，尝试别的岔路。



#### 关键数据结构：栈 (Stack)

为了实现“深入”和“回溯”的特性，DFS 的 `fringe` 需要一个 **后进先出 (Last-In, First-Out)** 的数据结构，也就是 **栈**。当然，DFS 也常用 **递归 (Recursion)** 实现，此时系统调用的函数栈就隐式地充当了这个栈的角色。



#### DFS 算法流程 (迭代版)

1. 创建一个栈 `fringe`，并将起始顶点 `s` 压入栈。
2. 创建一个集合 `visited` 用于记录已访问过的顶点，将 `s` 加入 `visited`。
3. 当 `fringe` 不为空时，循环执行： a. 从 `fringe` 的栈顶弹出一个顶点 `v`。 b. **处理 `v`**。 c. 遍历 `v` 的所有邻居 `u`： i. 如果 `u` **没有**在 `visited` 集合中： * 将 `u` 加入 `visited` 集合。 * 将 `u` 压入 `fringe` 栈。

*注意：在迭代版中，邻居入栈的顺序会影响最终的遍历路径。*



#### 图解 DFS 过程

我们还是用同一个图，从顶点 `0` 开始进行 DFS（假设邻居按数字从小到大顺序入栈）：

1. **初始状态**:
   - `fringe` (Stack): `[0]`
   - `visited`: `{0}`
   - `访问顺序`: ``
2. **步骤 1**: 栈顶 `0` 出栈。处理 `0`。将 `0` 的未访问邻居 `2`, `1` 按顺序压栈（先压 `2` 再压 `1`，这样 `1` 就在栈顶）。
   - `fringe`: `[2, 1]`
   - `visited`: `{0, 1, 2}`
   - `访问顺序`: `0`
3. **步骤 2**: 栈顶 `1` 出栈。处理 `1`。将 `1` 的未访问邻居 `3` 压栈。
   - `fringe`: `[2, 3]`
   - `visited`: `{0, 1, 2, 3}`
   - `访问顺序`: `0, 1`
4. **步骤 3**: 栈顶 `3` 出栈。处理 `3`。将 `3` 的未访问邻居 `4` 压栈。
   - `fringe`: `[2, 4]`
   - `visited`: `{0, 1, 2, 3, 4}`
   - `访问顺序`: `0, 1, 3`
5. **步骤 4**: 栈顶 `4` 出栈。处理 `4`。`4` 的邻居 `2`, `3` 已被访问。
   - `fringe`: `[2]`
   - `visited`: `{0, 1, 2, 3, 4}`
   - `访问顺序`: `0, 1, 3, 4`
6. **步骤 5**: 栈顶 `2` 出栈。处理 `2`。`2` 的邻居 `0`, `1`, `4` 已被访问。
   - `fringe`: `[]` (空)
   - `visited`: `{0, 1, 2, 3, 4}`
   - `访问顺序`: `0, 1, 3, 4, 2`
7. `fringe` 为空，遍历结束。最终访问顺序为 `0, 1, 3, 4, 2`。

**核心应用**: DFS 非常适合用于检测 **环路 (Cycle Detection)**、**路径查找 (Pathfinding)**、**拓扑排序 (Topological Sort)** 以及寻找图的 **连通分量 (Connected Components)**。

------



### 4. 总结与对比 (Summary)

| 特性                  | 广度优先搜索 (BFS)                | 深度优先搜索 (DFS)               |
| --------------------- | --------------------------------- | -------------------------------- |
| **核心思想**          | 逐层扩展，稳扎稳打                | 一条路走到底，再回溯             |
| **数据结构 (Fringe)** | **队列 (Queue)**                  | **栈 (Stack)** 或 **递归**       |
| **遍历路径**          | 离起点近的顶点先被访问            | 表现为“深入”的路径               |
| **找到的路径**        | 在无权图中，找到的是 **最短路径** | 找到的 **不一定** 是最短路径     |
| **空间复杂度**        | 最差情况 O(V) (当图很宽时)        | 最差情况 O(V) (当图很深时)       |
| **应用场景**          | 无权图最短路径、社交网络分析      | 拓扑排序、环路检测、寻找连通分量 |

**最重要的 takeaway**：BFS 和 DFS 的算法结构几乎完全一样。唯一的区别就在于管理 `fringe` (待访问列表) 的数据结构。**用队列就是 BFS，用栈就是 DFS**。这个统一的视角是 CS61B 课程强调的重点。

------



### 5. 代码解析 (Java Implementation)

下面我们用邻接表来实现一个简单的图，并提供 BFS 和 DFS 的代码。

```java
import java.util.*;

public class Graph {
    private final int V; // 顶点数量
    private final List<Integer>[] adj; // 邻接表

    public Graph(int V) {
        this.V = V;
        adj = new ArrayList[V];
        for (int i = 0; i < V; i++) {
            adj[i] = new ArrayList<>();
        }
    }

    // 添加一条边 (无向图)
    public void addEdge(int v, int w) {
        adj[v].add(w);
        adj[w].add(v);
    }

    // BFS 遍历
    public void bfs(int startNode) {
        System.out.println("BFS starting from node " + startNode);
        Queue<Integer> fringe = new LinkedList<>(); // 使用 LinkedList 实现队列
        Set<Integer> visited = new HashSet<>();

        fringe.add(startNode);
        visited.add(startNode);

        while (!fringe.isEmpty()) {
            int currentNode = fringe.poll(); // poll() 是出队操作
            System.out.print(currentNode + " ");

            for (int neighbor : adj[currentNode]) {
                if (!visited.contains(neighbor)) {
                    visited.add(neighbor);
                    fringe.add(neighbor); // add() 是入队操作
                }
            }
        }
        System.out.println("\nBFS finished.");
    }

    // DFS 遍历 (递归版)
    public void dfsRecursive(int startNode) {
        System.out.println("DFS (Recursive) starting from node " + startNode);
        Set<Integer> visited = new HashSet<>();
        dfsHelper(startNode, visited);
        System.out.println("\nDFS finished.");
    }

    private void dfsHelper(int currentNode, Set<Integer> visited) {
        visited.add(currentNode);
        System.out.print(currentNode + " ");

        for (int neighbor : adj[currentNode]) {
            if (!visited.contains(neighbor)) {
                dfsHelper(neighbor, visited);
            }
        }
    }

    // DFS 遍历 (迭代版)
    public void dfsIterative(int startNode) {
        System.out.println("DFS (Iterative) starting from node " + startNode);
        Deque<Integer> fringe = new ArrayDeque<>(); // Deque 可以作为栈使用
        Set<Integer> visited = new HashSet<>();

        fringe.push(startNode); // push() 是入栈操作

        while (!fringe.isEmpty()) {
            int currentNode = fringe.pop(); // pop() 是出栈操作

            // 注意：迭代版DFS，如果一个节点被访问过但未处理，需要跳过
            // 我们在入栈前检查 visited 状态可以避免这个问题
            if (!visited.contains(currentNode)) {
                visited.add(currentNode);
                System.out.print(currentNode + " ");

                for (int neighbor : adj[currentNode]) {
                    if (!visited.contains(neighbor)) {
                        fringe.push(neighbor);
                    }
                }
            }
        }
        System.out.println("\nDFS finished.");
    }

    public static void main(String[] args) {
        // 创建我们例子中的图
        Graph g = new Graph(5);
        g.addEdge(0, 1);
        g.addEdge(0, 2);
        g.addEdge(1, 2);
        g.addEdge(1, 3);
        g.addEdge(2, 4);
        g.addEdge(3, 4);

        g.bfs(0);
        System.out.println("--------------------");
        g.dfsRecursive(0);
        System.out.println("--------------------");
        g.dfsIterative(0);
    }
}
```



#### 代码运行结果分析

```
BFS starting from node 0
0 1 2 3 4 
BFS finished.
--------------------
DFS (Recursive) starting from node 0
0 1 2 4 3 
DFS finished.
--------------------
DFS (Iterative) starting from node 0
0 2 4 3 1 
DFS finished.
```

- **BFS 结果**: `0 1 2 3 4` - 完美地体现了逐层遍历：第0层(0)，第1层(1, 2)，第2层(3, 4)。
- **DFS (递归) 结果**: `0 1 2 4 3` - 体现了深度探索：`0 -> 1 -> 2 -> 4`，走到头了，回溯到 `2` 发现没路，回溯到 `1`，发现新路 `3`。
- **DFS (迭代) 结果**: `0 2 4 3 1` - 同样是深度探索，但路径不同。这是因为邻居入栈的顺序导致了探索顺序的改变。这两种DFS路径都是完全正确的。



### 总结

- **记住这个口诀**：**BFS 用队列，找最短；DFS 用栈，探到底**。
  - **广度优先搜索 (BFS)**：一种像水波纹一样逐层向外探索的遍历方法。
  - **深度优先搜索 (DFS)**：一种像走迷宫一样“一条路走到黑”的探索方法。
- **核心思想**: 算法的唯一区别在于 `fringe` 的数据结构。