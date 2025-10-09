## Lecture4 单向链表，嵌套类，哨兵节点



### 1. SLList (Singly Linked List) 单向链表

首先，课程从一个基础的 `IntList` 类开始，这个类直接暴露了它的内部结构（`first` 和 `rest`）。这样做虽然简单，但有两个主要问题：

1. **容易出错**：用户可以直接修改链表的任何部分，甚至是指向链表中间的某个节点，这使得数据结构很容易被破坏。
2. **使用不便**：用户需要了解其递归的内部结构才能正确使用。

为了解决这些问题，我们引入了 `SLList` (Singly Linked List) 类。

**核心思想：封装 (Encapsulation)**

`SLList` 类扮演了一个“中间人”或者叫“包装器” (wrapper) 的角色。它将底层的节点（现在我们称之为 `IntNode`）隐藏起来，不让用户直接接触。

- **访问控制 (Access Control)**：通过使用 `private` 关键字，我们可以将 `SLList` 的内部变量（比如指向第一个节点的引用）设为私有。这样一来，用户就不能直接修改链表的内部状态，而必须通过我们提供的公共方法（如 `addFirst()`, `getFirst()` 等）来操作链表。这大大增强了代码的健壮性和安全性。

  ```java
  public class SLList {
      private IntNode first; // 'private' 关键字是关键
      private int size;
  
      // ... 公共方法 ...
  }
  ```



### 2. Nested Classes (嵌套类)

在 `SLList` 的实现中，`IntNode` 类是专门为 `SLList` 服务的，外部世界完全不需要知道 `IntNode` 的存在。为了更好地组织代码并体现这种“从属”关系，我们把 `IntNode` 类定义在 `SLList` 类的内部，这就是 **嵌套类**。

```java
public class SLList {
    // 这是一个嵌套类
    private static class IntNode {
        public int item;
        public IntNode next;

        public IntNode(int i, IntNode n) {
            item = i;
            next = n;
        }
    }

    private IntNode first;
    private int size;
    // ...
}
```

**为什么使用嵌套类？**

- **组织性**：它将逻辑上相关的类组织在一起。`IntNode` 的唯一目的就是服务于 `SLList`，把它们放在一起，代码结构更清晰。
- **封装性**：将 `IntNode` 设为 `private`，可以彻底向外界隐藏这个实现细节。这样，`SLList` 的用户就完全不需要关心链表内部是如何通过节点连接起来的。

**重要知识点：`static` 嵌套类**

你可能注意到了 `private static class IntNode` 中的 `static` 关键字。

- **含义**：当一个嵌套类被声明为 `static` 时，它意味着这个嵌套类的实例（对象）与外部类的实例（对象）之间没有关联。换句话说，`IntNode` 对象不“属于”任何一个特定的 `SLList` 对象。
- **规则**：`static` 嵌套类不能直接访问外部类的 **实例变量** 或 **实例方法**。在我们的例子中，`IntNode` 只需要存储数据 (`item`) 和指向下一个节点的引用 (`next`)，它完全不需要访问 `SLList` 的 `first` 或 `size` 变量。因此，将它声明为 `static` 是最合适的。
- **优点**：这样做可以节省内存，因为 `IntNode` 对象不需要持有一个指向其外部 `SLList` 对象的隐藏引用。



### 3. Sentinel Nodes (哨兵节点)

在实现链表操作（如插入、删除）时，我们经常需要处理一些 **特殊情况 (edge cases)**，最典型的就是 **空链表**。

例如，在 `addLast(int x)` (在链表末尾添加元素) 方法中，如果链表是空的，我们需要修改 `first` 指针；如果链表非空，我们需要遍历到最后一个节点再进行修改。这使得代码逻辑变得复杂。

为了简化这种逻辑，我们引入了 **Sentinel Node (哨兵节点)**。

**核心思想：统一所有情况**

- **定义**：哨兵节点是一个 **虚拟的** 或 **哑(dummy)** 的头节点，它永远存在于链表中，即使链表本身是“空的”。这个节点不存储任何实际的数据。
- **结构**：在一个带有哨兵节点的 `SLList` 中，`first` (或者我们通常叫它 `sentinel`) 指针永远指向这个哨兵节点。链表的第一个 **实际** 元素存储在 `sentinel.next` 中。

```java
public class SLList {
    private static class IntNode { /* ... */ }

    // sentinel 永远指向一个不存储实际数据的 IntNode
    private IntNode sentinel;
    private int size;

    // 构造函数中就创建哨兵节点
    public SLList() {
        sentinel = new IntNode(63, null); // 63 是一个随意选择的魔法数字
        size = 0;
    }

    // ...
}
```

**使用哨兵节点的好处 (Invariants)**

哨兵节点帮助我们维护了几个重要的 **不变量 (Invariants)**，这些不变量是在任何时候都为真的事实，极大地简化了代码逻辑。

1. `sentinel` 引用永远指向哨兵节点 (永远不为 `null`)。
2. 链表的第一个真实元素（如果存在）总是在 `sentinel.next`。
3. `size` 变量永远准确地记录了链表中真实元素的数量。

**示例：`addLast` 方法的简化**

- **没有哨兵节点**：

  ```java
  public void addLast_no_sentinel(int x) {
      if (first == null) {
          first = new IntNode(x, null);
          return;
      }
      IntNode p = first;
      while (p.next != null) {
          p = p.next;
      }
      p.next = new IntNode(x, null);
  }
  ```

- **有哨兵节点**：

  ```java
  public void addLast_with_sentinel(int x) {
      IntNode p = sentinel; // 直接从 sentinel 开始
      // 无需检查 p 是否为 null，因为 sentinel 永远存在
      while (p.next != null) {
          p = p.next;
      }
      p.next = new IntNode(x, null);
  }
  ```

可以看到，有了哨兵节点后，我们不再需要 `if (first == null)` 这个特殊情况判断了。空链表和非空链表的处理逻辑被统一了。



### 总结

- **SLList**：通过封装和访问控制，提供了一个更安全、更易于使用的链表接口。
- **Nested Classes**：一种代码组织技巧，将仅为外部类服务的辅助类进行封装，`static` 嵌套类是其中的一种内存优化和逻辑解耦方式。
- **Sentinel Nodes**：通过引入一个虚拟的头节点，消除了对“空链表”等特殊情况的处理，统一了操作逻辑，使代码更简洁、更不易出错。