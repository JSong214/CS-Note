## Lecture17 哈希（Hashing）



### **第一部分：哈希思想的诞生与演进**

在学习哈希之前，我们先思考一个问题：如何设计一个数据结构，能够实现**平均O(1)时间复杂度**的插入、删除和查找操作？

我们学过的平衡二叉搜索树（如2-3树、红黑树）虽然高效，但其操作复杂度为 O(logN)，并且要求存入的元素必须是**可比较的**。哈希表则打破了这些限制。



#### **1. 最初的尝试：数据索引数组 (Data-Indexed Arrays)**

最直观的想法是：**直接用数据本身作为数组的索引**。

**案例 1: `DataIndexedIntegerSet` (用布尔数组存储整数)**

如果我们想存储一系列不重复的整数，可以创建一个巨大的布尔数组。如果数字 `x` 存在，就将 `present[x]` 设为 `true`。

```java
public class DataIndexedIntegerSet {
    private boolean[] present;

    // 假设我们只处理非负整数，并创建一个足够大的数组
    public DataIndexedIntegerSet(int max_value) {
        present = new boolean[max_value + 1];
    }

    public void add(int i) {
        present[i] = true; // O(1) 操作
    }

    public boolean contains(int i) {
        return present[i]; // O(1) 操作
    }
}
```

**优点:**

- `add` 和 `contains` 操作都是 O(1) 复杂度，速度极快。

**问题:**

1. **空间浪费**：如果只存储几个数字，比如 `{10, 1000, 100000}`，却需要创建一个长度为100001的数组，大部分空间都被浪费了。
2. **数据类型限制**：这种方法只能处理能直接作为数组索引的整数。如果是字符串或其他对象，怎么办？



#### **2. 演进：将非数字数据转换为索引**

为了处理像字符串这样的数据，我们需要一个“转换函数”，将它们变成独一无二的整数索引。

**案例 2: `DataIndexedWordSet` (处理英文单词)**

- **策略一（失败）**：用单词的首字母作为索引。比如，'a' 对应索引0，'b' 对应1，以此类推。这很快就会遇到问题：`"apple"` 和 `"ant"` 的首字母相同，它们会映射到同一个索引。这就是**哈希碰撞 (Collision)**。
- **策略二（改进）**：将每个单词看作一个26进制的数。例如，`a=0, b=1, ..., z=25`。 `"cat"` → c*26² + a*26¹ + t*26⁰ = 2*676 + 0*26 + 19*1 = 1371。 这样每个单词都有了唯一的整数值。

**案例 3: `DataIndexedStringSet` (扩展到任意字符串)**

我们可以进一步扩展，使用ASCII码（最大值为127）作为基数，将任意字符串转换为一个唯一的整数。

```java
public static int asciiToInt(String s) {
    int intRep = 0;
    for (int i = 0; i < s.length(); i++) {
        // 将字符串看作一个128进制的数
        intRep = intRep * 128;
        intRep = intRep + s.charAt(i);
    }
    return intRep;
}
```

**演进中遇到的根本问题：**

尽管策略二和`asciiToInt`保证了索引的唯一性，但它们生成的整数值会**极其巨大**。一个稍长的字符串转换后的数字可能远超Java `int` 类型的最大值，我们根本不可能创建这么大的数组。如果再考虑中文字符（Unicode编码有数万个字符），这个问题会更加突出。

------



### **第二部分：核心概念 - 哈希码 (Hash Code) 与哈希函数 (Hash Function)**

既然“唯一且巨大的索引”这条路走不通，我们必须换个思路。哈希的核心思想就此诞生：

> **我们不再追求为每个元素生成一个“唯一”的数组索引，而是允许不同的元素映射到同一个索引（即允许碰撞），然后我们再想办法解决这个碰撞。**



#### **1. `hashCode()` 方法：对象的整数表示**

为了将任意对象（字符串、自定义类等）转换成一个整数，Java 在 `Object` 类中定义了一个 `hashCode()` 方法。这意味着**任何Java对象都拥有一个哈希码**。

这个转换过程就是**哈希函数**。例如，Java中 `String` 类的 `hashCode()` 实现大致如下： `s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]` （选择31是因为它是一个奇素数，有助于让哈希码更均匀地分布，减少碰撞）。



#### **2. 压缩哈希码：取模运算**

`hashCode()` 返回的整数范围依然很大（`int` 类型的范围）。我们不可能创建一个能覆盖所有哈希码的数组。解决方法是**取模运算**，将巨大的哈希码“压缩”到我们数组的合法索引范围内。

```
index = Math.floorMod(key.hashCode(), array.length);
```

- `key.hashCode()`：获取对象的哈希码。
- `array.length`：我们实际创建的数组大小。
- `Math.floorMod()`：取模运算，确保结果为非负数。

**至此，我们将一个任意对象映射到数组索引的过程就完整了：** `Object key` → `key.hashCode()` → `(hash code) % array.length` → `int index`

------



### **第三部分：“好”哈希码与“坏”哈希码**

哈希表的性能严重依赖于哈希函数的质量。一个好的哈希函数应该具备哪些特点？



#### **“有效 (Valid)” 哈希码的两个铁律**

1. **确定性 (Deterministic)**：如果两个对象通过 `.equals()` 方法判断是相等的 (`a.equals(b) == true`)，那么它们的 `hashCode()` **必须**返回相同的值。这是哈希表能正确工作的最基本前提。
2. **一致性 (Consistent)**：在一次程序执行期间，对同一个对象多次调用 `hashCode()` 方法，必须始终返回同一个整数值。

**注意：**

- 对象相等，哈希码一定相等。
- 哈希码相等，对象未必相等。



#### **“好 (Good)” 哈希码的特征**

1. **有效性**：必须满足上述两个铁律。
2. **均匀分布**：哈希码应尽可能均匀地分布在所有整数范围内。一个好的哈希函数应该让不同的对象产生尽可能不同的哈希码，这样取模后才能均匀地分布在数组中，从而**减少碰撞**。
3. **高效性**：计算哈希码的过程必须快，否则会拖慢整个数据结构的速度。

------



### **第四部分：关键挑战 - 处理哈希碰撞 (Handling Collisions)**

由于我们将无限的对象映射到有限的数组索引上，**碰撞是不可避免的**。当两个不同的对象（比如 "apple" 和 "banana"）经过哈希计算后得到同一个索引时，我们必须有策略来处理。

最主流的解决方案是**外部拉链法 (External Chaining)**。

**原理：** 哈希表的数组不再直接存储元素本身，而是存储一个“桶”(Bucket)，这个桶通常是一个**链表 (`LinkedList`)**。

- 当一个元素需要插入到索引 `i` 时，我们不直接放入 `array[i]`，而是将它加入 `array[i]` 位置上那个链表的末尾。
- 当查找一个元素时，我们首先计算它的索引 `i`，然后遍历 `array[i]` 处的链表，用 `.equals()` 方法找到目标元素。

**代码逻辑示意：**

```java
public class ChainingHashSet<T> {
    private LinkedList<T>[] buckets;

    // 1. 计算索引
    private int getIndex(T item) {
        return Math.floorMod(item.hashCode(), buckets.length);
    }

    // 2. 添加元素
    public void add(T item) {
        int index = getIndex(item);
        if (buckets[index] == null) {
            buckets[index] = new LinkedList<>();
        }
        // 防止重复添加
        if (!buckets[index].contains(item)) {
            buckets[index].add(item);
        }
    }

    // 3. 查找元素
    public boolean contains(T item) {
        int index = getIndex(item);
        if (buckets[index] == null) {
            return false;
        }
        return buckets[index].contains(item);
    }
}
```

- **另一种方案（了解即可）：** **线性探测法 (Linear Probing)**。当索引 `i` 已被占用时，就尝试 `i+1`, `i+2`, ... 直到找到一个空位。这种方法实现复杂，且容易产生“聚集”现象，导致性能下降。

------



### **第五部分：性能保障 - 动态扩容 (Resizing)**

如果哈希表中元素越来越多，会发生什么？ 每个桶（链表）里的元素会越来越多，查找时遍历链表的时间会变长。在最坏的情况下（所有元素都碰撞到同一个索引），哈希表的性能会从 O(1) **退化到 O(N)**。

为了维持 O(1) 的平均性能，我们需要确保桶的平均长度是一个较小的常数。这通过**动态扩容**实现。



#### **负载因子 (Load Factor)**

这是个关键指标，用来衡量哈希表的“拥挤程度”。 **`Load Factor = N / M`**

- `N`：哈希表中元素的数量。
- `M`：数组的长度（桶的数量）。



#### **扩容机制**

1. 我们设定一个**最大负载因子**阈值（Java `HashMap` 默认为 **0.75**）。
2. 每次添加新元素后，检查当前的负载因子。如果超过了这个阈值，就触发**扩容 (Resizing)**。
3. 扩容过程： a. 创建一个更大的新数组，通常是**原数组大小的两倍**。 b. **遍历旧数组中的每一个元素**（不是遍历桶，而是桶里的每一个元素）。 c. 对于每个元素，**重新计算**它在新数组中的索引（因为数组长度 `M` 变了，取模结果也会变！），然后放入新数组的对应桶中。 d. 旧数组被垃圾回收。

扩容操作本身是 O(N) 的，比较耗时。但由于它发生的频率不高，平摊到每次插入操作上，其**均摊时间复杂度**仍然是 O(1)。

------



### **第六部分：Java 中的 `hashCode()` 与 `equals()` 契约**

我们已经知道，每个 Java 对象都有一个 `hashCode()` 方法。但它的默认行为是什么？我们又为什么需要重写（override）它呢？



#### **1. Java 的默认 `hashCode()` 实现**

如果你创建了一个自定义类，但没有重写 `hashCode()` 方法，那么它将继承来自 `Object` 类的默认实现。

**`Object` 类的默认 `hashCode()` 方法通常是基于对象的内存地址来生成的。**

- **优点**：由于每个新创建的对象都有不同的内存地址，默认的 `hashCode()` 能够很好地将不同对象散列到不同的桶中，分布性通常不错。
- **引出问题**：既然默认实现这么好，为什么我们还要费心去重写它？

答案就在于 `equals()` 方法。



#### **2. 哈希表的黄金法则：`equals()` 与 `hashCode()` 契约**

在 Java 中，有一条必须遵守的铁律：

> **如果你重写了 `equals(Object obj)` 方法，那么你必须也重写 `hashCode()` 方法。**

**为什么？**

让我们通过一个反例来说明。假设我们有一个 `Point` 类，我们认为只要 x 和 y 坐标相同，两个 `Point` 对象就是相等的。

```java
public class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // 我们重写了 equals 方法
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Point point = (Point) o;
        return x == point.x && y == point.y;
    }

    // 但是！！我们忘记重写 hashCode() 方法
    // public int hashCode() { ... }
}
```

现在，让我们看看会发生什么灾难：

```java
Point p1 = new Point(1, 2);
Point p2 = new Point(1, 2);

System.out.println("p1.equals(p2)? " + p1.equals(p2)); // 输出: true

HashSet<Point> set = new HashSet<>();
set.add(p1);

System.out.println("set.contains(p2)? " + set.contains(p2)); // 输出: false  <-- 问题在这里！
```

**问题分析：**

1. 根据我们的 `equals` 逻辑，`p1` 和 `p2` 是相等的。
2. 但是，我们没有重写 `hashCode()`，所以 `p1` 和 `p2` 使用的是基于内存地址的默认哈希码。因为它们是两个不同的对象，所以它们的内存地址不同，**哈希码也不同**。
3. 当 `set.add(p1)` 时，`p1` 的哈希码决定了它被放入了某个桶（比如3号桶）。
4. 当 `set.contains(p2)` 时，`HashSet` 首先计算 `p2` 的哈希码。由于 `p2` 的哈希码与 `p1` 不同，`HashSet` 会去一个完全不同的桶（比如7号桶）里寻找。它永远不会去3号桶检查，因此它永远找不到一个与 `p2` 相等的对象。

**结论：** 违反 `equals()` / `hashCode()` 契约会导致哈希表完全无法正常工作。



#### **3. `contains()` 方法的内部逻辑**

为了加深理解，我们必须清楚 `contains(item)` 的两步工作流程：

1. **第一步：定位桶**。计算 `item.hashCode()`，然后通过取模运算找到对应的桶索引。这是为了**快速缩小查找范围**。
2. **第二步：精确查找**。遍历定位到的那个桶（链表），对桶中的每一个元素 `e`，调用 `item.equals(e)` 进行比较。如果找到一个返回 `true` 的，则查找成功。

只有当哈希码相同时（第一步），才有机会通过 `equals` 方法进行比较（第二步）。

------



### **第七部分：哈希函数的设计对性能的影响**

一个设计糟糕的哈希函数，即使是“有效”的，也可能给哈希表带来毁灭性的性能打击。

**糟糕的哈希函数示例：**

- **常数哈希码**: `public int hashCode() { return 42; }`
  - **影响**: 所有对象都会被映射到同一个桶里。哈希表退化成一个巨大的链表，所有操作的复杂度都从 O(1) 变成 O(N)。
- **只使用部分字段**: 假设 `Point` 类只使用 `x` 坐标计算哈希码：`public int hashCode() { return x; }`
  - **影响**: `new Point(3, 1)`, `new Point(3, 99)`, `new Point(3, -50)` 这些点都会碰撞到一起，造成数据分布不均，形成一些特别长的链表。

一个好的哈希函数应该**让所有参与 `equals` 判断的字段都参与到哈希码的计算中**，并使用一些技巧（如乘以一个素数31）来最大化散列效果，确保最终结果分布均匀。

**一个合格的 `Point` 类实现：**

```java
public class Point {
    private final int x; // 设为 final，使其不可变
    private final int y;

    // ... 构造函数 ...

    @Override
    public boolean equals(Object o) {
        // ... (同上)
    }

    // 一个好的 hashCode 实现
    @Override
    public int hashCode() {
        int result = x;
        result = 31 * result + y; // 乘以素数31并加上其他字段
        return result;
        // 或者直接使用内置工具类：
        // return java.util.Objects.hash(x, y);
    }
}
```

------



### **第八部分：关键实践 - 为何应使用不可变对象作键 (Key)**

这是使用哈希表（尤其是 `HashMap` 和 `HashSet`）时最重要的一个实践原则：

> **绝对不要使用可变对象 (Mutable Objects) 作为哈希表的键 (Key)。**

**什么是可变对象？** 一个对象在被创建后，其内部状态（成员变量的值）仍然可以被改变。我们上面定义的 `Point` 类（如果 `x` 和 `y` 不是 `final` 的）就是可变的。

**使用可变键的致命问题：**

想象一下这个场景：

1. 你创建了一个可变的 `Point` 对象 `p = new Point(5, 10)`。
2. 你计算了它的哈希码（假设是 123），并将它存入了 `HashSet`。它被放进了基于哈希码123计算出的桶里。
3. **随后，你改变了这个对象的状态**：`p.setX(99);`。
4. 现在，`p` 对象的内容变成了 `(99, 10)`。如果你此时再调用 `p.hashCode()`，它会返回一个**全新的哈希码**（假设是 456）。
5. 当你调用 `set.contains(p)` 时，`HashSet` 会根据新的哈希码456去寻找对应的桶。但 `p` 对象**仍然躺在旧的、基于哈希码123计算出的那个桶里**！
6. 结果是，`contains(p)` 返回 `false`。这个对象就像在 `HashSet` 中“丢失”了一样，你再也找不到它了，也无法删除它。

**结论：** 为了保证哈希表的正确性，用作键的对象在存入哈希表后，其影响 `hashCode()` 计算的内部状态绝不能再发生改变。

因此，**不可变对象 (Immutable Objects)**，如 `String`、`Integer`、`Double` 以及我们上面用 `final` 关键字定义的 `Point` 类，是作为哈希表键值的最理想选择。



### **总结**

至此，我们完整地学习了哈希的全过程：

1. **目标**：实现 O(1) 的查找、插入、删除。
2. **核心思想**：通过**哈希函数 (`hashCode()` + 取模)** 将任意数据映射到数组的有限索引上。
3. **关键挑战**：处理不可避免的**哈希碰撞**。
4. **主流解决方案**：**外部拉链法**，在每个数组索引位上挂一个链表（桶）。
5. **性能保障**：通过**负载因子**监控拥挤程度，在过于拥挤时进行**动态扩容**，确保桶的平均长度始终很短。
6. **`equals()`/`hashCode()` 契约**：重写 `equals` 必须重写 `hashCode`，以保证相等的对象拥有相同的哈希码。
7. **`contains()` 原理**：先用 `hashCode` 定位桶，再用 `equals` 精确查找。
8. **哈希函数质量**：好的哈希函数应保证哈希值的均匀分布，避免性能退化。
9. **键的不可变性**：哈希表的键必须是不可变对象，以防止对象在集合中“丢失”。