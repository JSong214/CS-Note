## Lecture19 树的遍历 (Tree Traversals) 和 图 (Graphs) 



### **第一部分：树 (Tree) 的核心概念回顾**

在我们深入遍历之前，我们先快速回顾一下树的基本构成。树是一种**非线性**的数据结构，它模拟了层级关系，非常适合表示具有父子关系的数据。

**核心术语**

想象一下一棵家族树，这有助于你理解这些概念：

- **节点 (Node)**: 树的每个基本单元，代表一个数据项。
- **根节点 (Root)**: 树最顶端的节点，它是唯一没有**父节点 (Parent)** 的节点。
- **父节点 (Parent)**: 一个节点的直接上级节点。
- **子节点 (Child)**: 一个节点的直接下级节点。
- **边 (Edge)**: 连接父节点和子节点的线。
- **叶节点 (Leaf)**: 没有任何子节点的节点，位于树的末端。
- **子树 (Subtree)**: 树中的任意一个节点及其所有后代（子孙）构成的部分，本身也是一棵完整的树。
- **深度 (Depth)**: 从根节点到某个节点所经过的边的数量。根节点的深度为 0。
- **高度 (Height)**: 从某个节点到其最远叶节点所经过的边的数量。树的高度等于根节点的高度。

**图例解析**

下面这棵二叉树（每个节点最多有两个子节点）可以帮助你形象地理解这些术语：

```
      (A)  <-- 根节点 (Root), 深度 0
      / \
     /   \
   (B)   (C) <-- A的子节点, B和C互为兄弟节点
   / \     \
  /   \     \
(D)   (E)   (F) <-- D, E, F 都是叶节点 (Leaf)
```

- **节点 A** 是根节点。
- **节点 B** 是 A 的左子节点，A 是 B 的父节点。
- **节点 D, E, F** 是叶节点，因为它们没有子节点。
- **节点 B 和它的子节点 D、E** 构成了一个子树。
- **节点 C** 的深度是 1，高度是 1。
- **整棵树的高度** 是 2（从 A 到 D 或 E 的路径最长）。

------



### **第二部分：树的遍历 (Tree Traversals)**

“遍历”是指按照某种特定的顺序访问树中的每一个节点，并且每个节点只访问一次。这是树结构最基本也是最重要的操作。主要有四种遍历方式。

我们将使用下面这棵二叉搜索树作为例子进行讲解：

```
      (F)
      / \
     /   \
   (B)   (G)
   / \     \
  /   \     \
(A)   (D)   (I)
      / \
     /   \
   (C)   (E)
```



#### 1. 前序遍历 (Pre-order Traversal)

**规则**: **根 (Root) -> 左 (Left) -> 右 (Right)**

这是一种深度优先的策略。先访问当前节点，然后递归地访问其左子树，最后递归地访问其右子树。

- **访问顺序**: `F -> B -> A -> D -> C -> E -> G -> I`
- **应用**: 非常适合用来复制一棵树，或者得到表达式树的前缀表达式。

**代码解析 (Java, 递归实现)**

```java
class TreeNode {
    char val;
    TreeNode left;
    TreeNode right;
    TreeNode(char x) { val = x; }
}

public void preorderTraversal(TreeNode node) {
    if (node == null) {
        return;
    }
    // 1. 访问根节点
    System.out.print(node.val + " ");
    // 2. 遍历左子树
    preorderTraversal(node.left);
    // 3. 遍历右子树
    preorderTraversal(node.right);
}
```



#### 2. 中序遍历 (In-order Traversal)

**规则**: **左 (Left) -> 根 (Root) -> 右 (Right)**

先递归地访问左子树，然后访问当前节点，最后递归地访问右子树。

- **访问顺序**: `A -> B -> C -> D -> E -> F -> G -> I`
- **应用**: 在**二叉搜索树 (Binary Search Tree)** 上进行中序遍历，可以得到一个有序的节点序列。这非常有用！

**代码解析 (Java, 递归实现)**

```java
public void inorderTraversal(TreeNode node) {
    if (node == null) {
        return;
    }
    // 1. 遍历左子树
    inorderTraversal(node.left);
    // 2. 访问根节点
    System.out.print(node.val + " ");
    // 3. 遍历右子树
    inorderTraversal(node.right);
}
```



#### 3. 后序遍历 (Post-order Traversal)

**规则**: **左 (Left) -> 右 (Right) -> 根 (Root)**

先递归地访问左子树，然后递归地访问右子树，最后才访问当前节点。

- **访问顺序**: `A -> C -> E -> D -> B -> I -> G -> F`
- **应用**: 适合用于计算目录（文件夹）大小或安全地删除树的节点（必须先删除子节点才能删除父节点）。

**代码解析 (Java, 递归实现)**

```java
public void postorderTraversal(TreeNode node) {
    if (node == null) {
        return;
    }
    // 1. 遍历左子树
    postorderTraversal(node.left);
    // 2. 遍历右子树
    postorderTraversal(node.right);
    // 3. 访问根节点
    System.out.print(node.val + " ");
}
```



#### 4. 层序遍历 (Level-order Traversal)

**规则**: **从上到下，从左到右，逐层访问**

这是一种广度优先的策略。它从根节点开始，访问完一层的所有节点后，再进入下一层。

- **访问顺序**: `F -> B -> G -> A -> D -> I -> C -> E`
- **实现**: 通常使用一个**队列 (Queue)** 来实现。

**代码解析 (Java, 迭代实现)**

```java
import java.util.Queue;
import java.util.LinkedList;

public void levelOrderTraversal(TreeNode root) {
    if (root == null) {
        return;
    }
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root); // 首先将根节点入队

    while (!queue.isEmpty()) {
        TreeNode currentNode = queue.poll(); // 从队列中取出一个节点
        System.out.print(currentNode.val + " "); // 访问它

        // 如果它有左子节点，将其入队
        if (currentNode.left != null) {
            queue.add(currentNode.left);
        }
        // 如果它有右子节点，将其入队
        if (currentNode.right != null) {
            queue.add(currentNode.right);
        }
    }
}
```

------



### **第三部分：图 (Graphs) 的入门**

图是比树更泛化的数据结构。如果说树是表示“层级”或“父子”关系，那么图则可以表示**任意节点之间的关系**。

**核心术语**

- **顶点 (Vertex/Node)**: 与树的节点相同，代表图中的一个点。
- **边 (Edge)**: 连接两个顶点的线，代表它们之间的关系。
- **无向图 (Undirected Graph)**: 边没有方向，如果 A 连接到 B，那么 B 也连接到 A。
- **有向图 (Directed Graph)**: 边有方向，A 指向 B 不代表 B 指向 A。

**图的表示方法**

如何用代码来存储一个图？主要有两种方法：



#### 1. 邻接矩阵 (Adjacency Matrix)

用一个二维数组 `matrix` 来表示图，`matrix[i][j] = 1` 表示顶点 i 和 j 之间有一条边，为 0 则表示没有。

**图例**:

```
  (0) -- (1)
   |    /
   |   /
  (2) -- (3)
```

**邻接矩阵**:

```
  0 1 2 3
0[0,1,1,0]
1[1,0,1,0]
2[1,1,0,1]
3[0,0,1,0]
```

- **优点**: 查询两个顶点间是否存在边非常快（O(1) 时间复杂度）。
- **缺点**: 如果图很稀疏（边很少），会浪费大量存储空间（O(V²) 空间复杂度，V是顶点数）。



#### 2. 邻接表 (Adjacency List)

为每个顶点维护一个列表，该列表存储了所有与该顶点直接相连的其他顶点。

**图例 (同上)**

**邻接表**:

```
0: [1, 2]
1: [0, 2]
2: [0, 1, 3]
3: [2]
```

- **优点**: 节省空间，特别是对于稀疏图（空间复杂度为 O(V+E)，E是边数）。
- **缺点**: 查询两个顶点间是否存在边需要遍历列表（最坏 O(V) 时间）。

在 CS61B 和大多数实际应用中，**邻接表是更常用和高效的选择**。

------



### **第四部分：图的遍历与常见问题**

与树类似，图的遍历也是核心操作，主要分为深度优先和广度优先两种。

我们将使用下面这个无向图作为例子：

```
      (A) --- (B)
      / \     /
     /   \   /
    (C)---(D)
      \   /
       \ /
        (E)
```



#### 1. 深度优先搜索 (Depth-First Search - DFS)**

**策略**: "一条路走到黑，再回头"。从一个起始顶点出发，尽可能深地探索图的分支。当一个顶点的所有邻居都被访问过，就回溯到上一个顶点，继续探索。

- **实现**: 通常使用**栈 (Stack)** (或递归，递归本身就利用了调用栈)。
- **从 A 开始的 DFS 访问顺序 (一种可能)**: `A -> B -> D -> C -> E`

**代码解析 (Java, 迭代实现)**

```java
import java.util.*;

// 假设图用邻接表表示 Map<Character, List<Character>> graph;
public void dfs(char startNode, Map<Character, List<Character>> graph) {
    Stack<Character> stack = new Stack<>();
    Set<Character> visited = new HashSet<>(); // 记录已访问的节点

    stack.push(startNode);
    visited.add(startNode);

    while (!stack.isEmpty()) {
        char currentNode = stack.pop();
        System.out.print(currentNode + " ");

        // 将当前节点的未访问邻居入栈
        for (char neighbor : graph.get(currentNode)) {
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);
                stack.push(neighbor);
            }
        }
    }
}
```



#### 2. 广度优先搜索 (Breadth-First Search - BFS)**

**策略**: "一层一层地向外探索"。从一个起始顶点出发，首先访问它的所有直接邻居，然后是邻居的邻居，以此类推。

- **实现**: 通常使用**队列 (Queue)**。
- **从 A 开始的 BFS 访问顺序**: `A -> B -> C -> D -> E`
- **应用**: **是寻找无权图中两点之间最短路径的利器**。

**代码解析 (Java, 迭代实现)**

```java
// 假设图用邻接表表示 Map<Character, List<Character>> graph;
public void bfs(char startNode, Map<Character, List<Character>> graph) {
    Queue<Character> queue = new LinkedList<>();
    Set<Character> visited = new HashSet<>();

    queue.add(startNode);
    visited.add(startNode);

    while (!queue.isEmpty()) {
        char currentNode = queue.poll();
        System.out.print(currentNode + " ");

        // 将当前节点的未访问邻居入队
        for (char neighbor : graph.get(currentNode)) {
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);
                queue.add(neighbor);
            }
        }
    }
}
```



#### **其他常见的图问题**

- **环检测 (Cycle Detection)**: 判断图中是否存在环路。DFS 是解决这个问题的常用方法。
- **拓扑排序 (Topological Sort)**: 针对**有向无环图 (DAG)**，给出一个线性的顶点排序，使得对于每条有向边 (u, v)，u 都在 v 的前面。常用于任务调度等场景。
- **最短路径 (Shortest Path)**: 寻找图中两点之间的最短路径。对于无权图，使用 BFS；对于有权图，则需要使用 Dijkstra 算法或 Bellman-Ford 算法。

------



### **总结**

- **树**是特殊的图（无环、连通）。
- **树的遍历**有四种经典方式，各有其用途和实现特点。
- **图**是更通用的结构，用邻接表或邻接矩阵表示。
- **图的遍历**分为 DFS（深入探索）和 BFS（逐层探索），它们是解决许多图问题的基础。