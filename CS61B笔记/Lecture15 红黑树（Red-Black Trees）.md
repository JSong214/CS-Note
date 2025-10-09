## Lecture15 红黑树（Red-Black Trees）



### **为什么需要红黑树？**

在学习红黑树之前，我们先要明白它的“使命”。我们知道，一个普通的二叉搜索树（BST）在最坏的情况下，可能会退化成一个链表（比如，你按顺序插入1, 2, 3, 4, 5），这会导致其搜索、插入等操作的时间复杂度从期望的 O(log N) 下降到 O(N)。

为了解决这个问题，我们需要一种能够**自我平衡**的二叉搜索树，无论我们如何插入或删除节点，它都能通过一系列调整，始终保持一个“矮胖”的、平衡的形态，从而确保操作的时间复杂度稳定在 O(log N)。红黑树就是实现这一目标的杰出代表之一。而CS61B中教授的左倾红黑树，是红黑树的一种简化实现，更易于编码。

------



### **知识点一：树的旋转 (Tree Rotations)**

树的旋转是实现自平衡的关键“动作”。它可以在不破坏二叉搜索树性质（即左子节点 < 父节点 < 右子节点）的前提下，改变树的结构，降低树的高度。旋转分为两种：**左旋（Left Rotation）和右旋（Right Rotation）**。



#### **1. 左旋 (rotateLeft)**

当某个节点 `G` 和它的右子节点 `P` 出现了不平衡的情况时（在LLRB中，特指出现了一个“向右倾斜”的红色链接），我们需要进行左旋。

- **操作对象**：节点 `G`。
- **目标**：将 `G` 的右子节点 `P` “提”上来，取代 `G` 的位置，而 `G` 则成为 `P` 的左子节点。
- **过程详解**：
  1. `P` 的左子树（原本在`G`和`P`之间）现在比 `G` 大，但比 `P` 小。因此，它需要成为 `G` 的新的右子树。
  2. `G` 成为 `P` 的左子节点。
  3. `P` 取代 `G` 原来的位置。

**图解左旋 `rotateLeft(G)`:**

```
     G                  P
    / \                / \
   A   P     --->     G   C
      / \            / \
     B   C          A   B
```



#### **2. 右旋 (rotateRight)**

右旋与左旋完全相反。当某个节点 `P` 和它的左子节点 `G` 出现不平衡时（在LLRB中，特指出现了连续两个“向左倾斜”的红色链接），我们需要进行右旋。

- **操作对象**：节点 `P`。
- **目标**：将 `P` 的左子节点 `G` “提”上来，取代 `P` 的位置，而 `P` 则成为 `G` 的右子节点。

**图解右旋 `rotateRight(P)`:**

```
       P                  G
      / \                / \
     G   C     --->     A   P
    / \                    / \
   A   B                  B   C
```

旋转是后面所有平衡操作的基础，请务必理解这两个操作是如何在不改变节点顺序的情况下调整树的结构的。

------



### **知识点二：创建左倾红黑树 (Creating LLRB Trees)**

CS61B的左倾红黑树与经典的2-3树有着非常紧密的联系，甚至可以说，**LLRB是2-3树的一种二进制表示**。理解这一点是掌握LLRB的关键。

- **2-3树**：一种B树，它的每个节点可以有2个或3个子节点，并且所有叶子节点到根节点的距离都相同，因此是完美平衡的。
  - **2-节点**：包含1个键和2个子节点（一个小于键，一个大于键）。
  - **3-节点**：包含2个键和3个子节点。

LLRB通过引入“红色链接”的概念，巧妙地将2-3树映射到了一个标准的二叉搜索树上。

- **映射规则**：
  1. 2-3树中的**3-节点**，在LLRB中用一个**左倾的红色链接**来表示。较小的键是父节点，较大的键是其右子节点，它们通过一个红色链接连接，并“粘合”在一起，概念上代表同一个3-节点。
  2. 连接2-节点和3-节点的链接，在LLRB中都是**黑色链接**。

**图解2-3树到LLRB的转换：**

```
   (3, 7)         ------->         7 (黑)
  /  |  \                         /    \
 1   5   8                       3 (红) 8
                                / \
                               1   5
```

在这个例子中，2-3树的(3, 7)节点被转换成了一个LLRB结构，其中7是黑节点，3是它的红色左子节点。这个红色的链接告诉我们，3和7在概念上属于同一个2-3树节点。



#### **LLRB的性质 (Invariants):**

基于这种映射关系，我们得出了左倾红黑树必须严格遵守的几条核心性质：

1. **红色链接必须是左倾的** (No right-leaning red links)。绝对不允许出现一个节点的右链接是红色的情况。
2. **根节点必须是黑色的** (Root is black)。
3. **不许有连续的红色链接** (No two consecutive red links)。一个红色节点的父节点必须是黑色的。
4. **完美黑色平衡** (Perfect black balance)：从根节点到任何一个null链接（叶子节点）的路径上，黑色链接的数量都是完全相同的。这直接对应了2-3树中所有叶子节点深度相同的性质。

------



### **知识点三：向左倾红黑树中插入 (Inserting into LLRB Trees)**

LLRB的插入操作非常优雅。它首先像普通BST一样插入一个新节点，然后通过一系列的旋转和颜色翻转操作，自底向上地修复可能被破坏的LLRB性质。

**核心思想：** 总是以**红色链接**将新节点连接到树上。这可以看作是向一个2-3树的节点中添加一个新的键。

插入后，可能会违反LLRB的性质。我们需要在递归返回的路径上，通过以下三种修复操作来恢复平衡：

1. **修复右倾的红色链接**：

   - **触发条件**：`node.left` 是黑色，而 `node.right` 是红色。

   - **操作**：对 `node` 进行**左旋 (`rotateLeft`)**。

   - **代码逻辑**：

     ```java
     if (isRed(h.right) && !isRed(h.left)) {
         h = rotateLeft(h);
     }
     ```

2. **修复连续的左倾红色链接**：

   - **触发条件**：`node.left` 和 `node.left.left` 都是红色。

   - **操作**：对 `node` 进行**右旋 (`rotateRight`)**。

   - **代码逻辑**：

     ```java
     if (isRed(h.left) && isRed(h.left.left)) {
         h = rotateRight(h);
     }
     ```

3. **拆分临时的4-节点**：

   - **触发条件**：`node.left` 和 `node.right` 都是红色。

   - **操作**：进行**颜色翻转 (`flipColors`)**，将两个子节点的颜色由红变黑，父节点的颜色由黑变红。这相当于在2-3树中，一个节点满了（有3个键），于是中间的键被“推”上去，节点分裂。

   - **代码逻辑**：

     ```java
     if (isRed(h.left) && isRed(h.right)) {
         flipColors(h);
     }
     ```



#### **插入操作的完整代码解析**

下面是CS61B风格的LLRB插入方法的伪代码/Java实现，包含了详细的注释。

```java
private static final boolean RED = true;
private static final boolean BLACK = false;

private Node root;

private class Node {
    Key key;
    Value val;
    Node left, right;
    boolean color; // 节点的颜色（指向其父节点的链接的颜色）

    Node(Key key, Value val, boolean color) {
        this.key = key;
        this.val = val;
        this.color = color;
    }
}

private boolean isRed(Node x) {
    if (x == null) return false; // null链接是黑色的
    return x.color == RED;
}

public void put(Key key, Value val) {
    root = put(root, key, val);
    root.color = BLACK; // 确保根节点总是黑色的
}

private Node put(Node h, Key key, Value val) {
    // 1. 和普通BST一样，找到插入位置并插入新节点
    if (h == null) {
        return new Node(key, val, RED); // 新节点总是用红色链接连接
    }

    int cmp = key.compareTo(h.key);
    if (cmp < 0) {
        h.left = put(h.left, key, val);
    } else if (cmp > 0) {
        h.right = put(h.right, key, val);
    } else {
        h.val = val;
    }

    // 2. 在递归返回的路上，修复可能出现的LLRB性质违反
    // 注意这三个if语句的顺序是固定的，不能随意调换

    // Case 1: 右子节点是红色，左子节点是黑色 -> 左旋
    if (isRed(h.right) && !isRed(h.left)) {
        h = rotateLeft(h);
    }

    // Case 2: 左子节点和左子节点的左子节点都是红色 -> 右旋
    if (isRed(h.left) && isRed(h.left.left)) {
        h = rotateRight(h);
    }

    // Case 3: 左右子节点都是红色 -> 颜色翻转
    if (isRed(h.left) && isRed(h.right)) {
        flipColors(h);
    }

    return h;
}

// 颜色翻转
private void flipColors(Node h) {
    h.color = RED;
    h.left.color = BLACK;
    h.right.color = BLACK;
}

// 左旋 (返回新的子树根节点)
private Node rotateLeft(Node h) {
    Node x = h.right;
    h.right = x.left;
    x.left = h;
    x.color = h.color;
    h.color = RED;
    return x;
}

// 右旋 (返回新的子树根节点)
private Node rotateRight(Node h) {
    Node x = h.left;
    h.left = x.right;
    x.right = h;
    x.color = h.color;
    h.color = RED;
    return x;
}
```

------



### **知识点四：运行时分析 (Runtime Analysis)**

LLRB树最强大的地方在于它能保证所有核心操作（`get`, `put`, `delete`）在最坏情况下的时间复杂度也是对数级别的。

- **树的高度**：LLRB的“完美黑色平衡”性质保证了树的高度 `h` 大约是 `log N`（其中N是节点数）。具体来说，从根到最远叶子节点的路径长度不会超过到最近叶子节点路径长度的两倍。因此，树的高度始终保持在 **O(log N)**。
- **搜索 (`get`)**：搜索操作与普通BST完全相同，沿着一条路径从根节点向下查找。由于树高是 O(log N)，所以搜索时间复杂度是 **O(log N)**。
- **插入 (`put`)**：插入操作包括一次向下的遍历（查找插入位置）和一次向上的回溯（进行旋转和颜色翻转）。每次回溯时，旋转和颜色翻转操作都是常数时间 O(1) 的。因此，总的时间复杂度也由树的高度决定，为 **O(log N)**。

------



### **知识点五：总结 (Summary)**

1. **平衡是关键**：普通BST可能退化，导致性能下降。红黑树通过自平衡机制，保证了操作效率。
2. **LLRB是2-3树的二进制实现**：通过左倾的红色链接来模拟2-3树中的3-节点，这是理解LLRB所有规则和操作的基石。
3. **三大核心操作**：
   - `rotateLeft()`: 修复右倾的红色链接。
   - `rotateRight()`: 修复连续的左倾红色链接。
   - `flipColors()`: 拆分临时的4-节点。
4. **插入流程**：先按BST规则插入一个红色节点，然后在递归返回时，依次使用上述三个操作，自底向上地恢复树的平衡。
5. **性能保证**：LLRB树的高度被严格限制在 O(log N)，因此查找、插入和删除（虽然我们没详细讲，但原理类似）操作的最坏时间复杂度都是 **O(log N)**。