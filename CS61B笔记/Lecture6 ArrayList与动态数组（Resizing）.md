## Lecture5 ArrayList与动态数组（Resizing）

### 核心思想：为什么选择数组来实现列表？

我们已经学习过链表（如 `SLList` 或 `DLList`），它在添加（`add`）和删除（`remove`）操作上非常高效（通常是 O(1)），但获取（`get`）特定位置的元素却很慢。比如，要获取第 `i` 个元素（`get(i)`），链表需要从头节点开始，一步一步地向后移动 `i` 次，时间复杂度是 O(N)。

而数组（Array）这种数据结构，最大的优势就是**随机访问**。通过索引直接访问数组中的任何元素（`array[i]`）是一个非常快速的操作，时间复杂度是 O(1)。

`ArrayList`（在 CS61B 教材中被称为 `AList`）的目标就是结合这两者的优点：既能像列表一样动态地添加和删除元素，又能像数组一样快速地获取元素。它的内部实现就是基于一个数组。

------



### 知识点一：`AList` 的基本结构

一个 `AList` 类通常包含三个核心部分：

1. 一个用来存储元素的数组（`items`）。
2. 一个整数，记录当前存储的元素数量（`size`）。
3. 一些基本的操作方法，如 `addLast`, `getLast`, `get`, `size` 等。

下面是一个 `AList` 的基本框架：

```java
public class AList<Item> {
    private Item[] items;
    private int size;

    /** 创建一个空的列表 */
    public AList() {
        items = (Item[]) new Object[100]; // 初始容量设为100
        size = 0;
    }

    /** 在列表末尾添加一个新元素 x */
    public void addLast(Item x) {
        items[size] = x;
        size = size + 1;
    }

    /** 获取列表末尾的元素 */
    public Item getLast() {
        return items[size - 1];
    }

    /** 获取第 i 个位置的元素 */
    public Item get(int i) {
        return items[i];
    }

    /** 返回列表中的元素数量 */
    public int size() {
        return size;
    }

    /** 删除并返回列表末尾的元素 */
    public Item removeLast() {
        Item x = getLast();
        items[size - 1] = null; // 防止内存泄漏 (loitering)
        size = size - 1;
        return x;
    }
}
```

**重点解释：**

- `items = (Item[]) new Object[100];` 这行代码是 Java 中创建泛型数组的一种特殊语法。我们不能直接 `new Item[100]`，所以先创建一个 `Object` 数组，然后强制类型转换为 `Item[]`。
- `items[size - 1] = null;` 在 `removeLast` 中，当我们删除一个元素后，需要将数组中对应的位置设为 `null`。这非常重要，因为如果不这样做，即使这个元素已经被“删除”，数组仍然持有对它的引用，垃圾回收器（Garbage Collector）就无法回收这部分内存，导致**内存泄漏（Memory Leak）**，这种情况在教材中被称为 **loitering**。

------



### 知识点二：数组容量问题与动态扩容 (Resizing)

上面的 `AList` 实现有一个致命的问题：它的容量是固定的（100）。当 `size` 达到 100 时，如果我们再调用 `addLast`，就会发生 `ArrayIndexOutOfBoundsException` 错误。

为了解决这个问题，我们需要在数组满了的时候进行**扩容（Resizing）**。



#### 1. 朴素的扩容方法（Naive Resizing）

最直观的想法是：当数组满了，就创建一个比原来大一点点的新数组（比如，大 1 个单位），然后把旧数组的元素全部复制过去。

```java
public void addLast(Item x) {
    if (size == items.length) {
        // 创建一个大1的新数组
        Item[] newItems = (Item[]) new Object[size + 1];
        // 把旧数组的元素复制过来
        System.arraycopy(items, 0, newItems, 0, size);
        items = newItems;
    }
    items[size] = x;
    size = size + 1;
}
```

**为什么这种方法不好？** 这种每次只增加固定容量（比如 1）的方法，效率极低。假设我们要连续添加 N 个元素，那么每次添加都需要进行一次数组复制。第 1 次添加复制 0 个，第 2 次复制 1 个，第 3 次复制 2 个... 第 N 次复制 N-1 个。总的复制次数大约是 `1 + 2 + ... + (N-1)`，这是一个等差数列，总和约为 N²/2。因此，添加 N 个元素的时间复杂度是 O(N²)，非常慢。



#### 2. 几何级数扩容（Geometric Resizing） - **【核心重点】**

更高效的策略是**几何级数扩容**或**比例扩容**。当数组满了的时候，我们创建一个容量是原来 `R` 倍的新数组（`R` 通常取 2，或者 1.5）。

```java
public class AList<Item> {
    private Item[] items;
    private int size;
    private static final int RFACTOR = 2; // 扩容因子

    public AList() {
        items = (Item[]) new Object[8]; // 初始容量可以小一点
        size = 0;
    }
    
    /** 调整数组大小的核心方法 */
    private void resize(int capacity) {
        Item[] newItems = (Item[]) new Object[capacity];
        System.arraycopy(items, 0, newItems, 0, size);
        items = newItems;
    }

    public void addLast(Item x) {
        if (size == items.length) {
            // 当数组满了，按比例扩容
            resize(size * RFACTOR);
        }
        items[size] = x;
        size = size + 1;
    }
    // ... 其他方法 ...
}
```

**为什么这种方法高效？** 这种策略的奇妙之处在于，虽然单次 `resize` 操作很耗时（需要复制所有元素），但它并**不经常发生**。

我们来分析一下连续添加 N 个元素的情况。假设初始容量为 8，扩容因子为 2。

- 添加到第 8 个元素时，数组满了。
- 添加第 9 个元素时，触发 `resize`，容量变为 16。这次复制了 8 个元素。
- 接下来添加 8 个元素（从第 9 到第 16）都不需要 `resize`。
- 添加第 17 个元素时，再次触发 `resize`，容量变为 32。这次复制了 16 个元素。

可以看到，`resize` 发生的频率越来越低。经过复杂的数学分析（摊销分析, Amortized Analysis），可以证明，使用几何级数扩容策略，添加 N 个元素的总时间复杂度是 O(N)。这样，**平均下来，每次 `addLast` 操作的时间复杂度就是 O(1)**。这是一种“牺牲”单次操作的最坏情况性能，来换取整体、长期的高效性能的典型例子。

------



### 知识点三：缩容 (Downsizing)

当我们从 `AList` 中不断删除元素时，可能会出现一个问题：`size` 变得很小，但 `items` 数组的容量（`length`）却依然很大，造成了内存空间的浪费。

为了解决这个问题，我们需要在元素数量过少时进行**缩容**。

一个好的策略是：设定一个**使用率（Usage Ratio）**的阈值。 `Usage Ratio = size / items.length`

当使用率低于某个值（例如 25% 或 0.25）时，我们就将数组的容量减半。

```java
public Item removeLast() {
    Item x = getLast();
    items[size - 1] = null;
    size = size - 1;

    // 检查使用率是否过低
    double usageRatio = (double) size / items.length;
    if (items.length >= 16 && usageRatio < 0.25) {
        resize(items.length / 2);
    }
    
    return x;
}
```

**重点解释：**

- `items.length >= 16`：设置一个最小容量，防止数组缩得太小。
- `usageRatio < 0.25`：为什么是 0.25 而不是 0.5？这是一个重要的设计选择。如果我们在使用率低于 0.5 时就缩容到一半，可能会导致在某个临界点（比如数组半满时）频繁地进行 `addLast` 和 `removeLast` 操作，从而不断触发扩容和缩容，造成性能抖动（Thrashing）。将缩容阈值设为 0.25，可以为数组提供一个“缓冲地带”，避免这种情况。



### 总结

`AList` (ArrayList) 通过以下几个关键设计，实现了高效的动态列表功能：

1. **内部数组存储**：保证了 `get(i)` 操作的 O(1) 高效率。
2. **几何级数扩容**：在数组满时，按比例（如 `*2`）创建新数组，确保了 `addLast` 操作的摊销时间复杂度为 O(1)。
3. **使用率监控与缩容**：在删除元素导致数组使用率过低时（如 `< 25%`），将数组容量减半，回收了多余的内存空间，同时避免了性能抖动。
4. **处理内存泄漏**：在删除元素时，将数组对应位置设为 `null`，以避免 "loitering"。