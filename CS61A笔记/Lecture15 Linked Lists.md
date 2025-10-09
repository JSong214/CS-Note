## Lecture14 Linked Lists



#### 1. 核心思想：什么是链表？

链表是一种线性的、递归的数据结构。与 Python 内置的 `list` 不同，它不是一整块连续的内存空间，而是由一系列**节点 (Node)** 链接而成。

- **每个节点 (Node) 包含两部分信息：**
  1. **当前元素的值 (the first element)**
  2. **指向下一个节点的引用 (a reference to the rest of the list)**
- **链表的结尾：** 最后一个节点指向一个特殊的、表示“空链表”的对象。

这个定义本身就是递归的：**一个链表要么是空的，要么是一个节点，该节点包含一个值和对另一个链表的引用。**

------



#### 2. **【重要】** 数据抽象：`Link` 类的实现

在 CS61A 中，我们不会直接使用内置库，而是从零开始构建链表，以此来学习数据抽象。这通常通过一个名为 `Link` 的类来实现。



##### **`Link` 类的基本结构**

```python
class Link:
    """A linked list."""
    # 一个特殊的类属性，用于表示所有空链表。这是一个单例对象。
    empty = ()

    def __init__(self, first, rest=empty):
        # 断言确保 rest 是一个 Link 对象或者是 Link.empty
        assert rest is Link.empty or isinstance(rest, Link)
        self.first = first
        self.rest = rest

    def __repr__(self):
        # 这个方法能让我们方便地打印链表，例如 Link(1, Link(2, Link(3)))
        if self.rest is Link.empty:
            return f'Link({self.first})'
        else:
            return f'Link({self.first}, {self.rest})'

    def __str__(self):
        # 这个方法让打印输出更美观，例如 <1 2 3>
        string = '<'
        curr = self
        while curr is not Link.empty:
            string += str(curr.first) + ' '
            curr = curr.rest
        return string.strip() + '>'
```



##### **重点解释：**

- **构造器 (Constructor):** `__init__(self, first, rest=Link.empty)`
  - `first`: 存储当前节点的值。
  - `rest`: 存储对下一个 `Link` 对象的引用。它的默认值是 `Link.empty`，这使得创建只包含一个元素的链表非常方便（例如 `Link(1)`）。
- **选择器 (Selectors):**
  - `link.first`: 获取链表的第一个元素。
  - `link.rest`: 获取除第一个元素外，由余下节点组成的链表。
- **`Link.empty` 的作用 (极其重要！)**
  - 它是一个**哨兵值 (sentinel value)**，代表链表的终点。
  - **为什么不用 `None`？** 在 CS61A 中，我们强调清晰的抽象。`None` 在 Python 中有多种含义（例如，函数没有返回值），而 `Link.empty` 的含义非常专一：它就是一个**空链表**。
  - 所有空链表都指向**同一个对象** `Link.empty`。这在判断链表是否为空时非常高效，只需进行身份检查 `if my_link is Link.empty:`。

------



#### 3. **【重要】** 处理链表：递归 vs. 迭代

链表的递归结构使得用递归函数来处理它变得非常自然和优雅。



##### **递归方法 (The Recursive Way)**

这是 CS61A 中最推崇的方式。处理逻辑分为两步：

1. **基本情况 (Base Case):** 如果链表是空的 (`link is Link.empty`)，我们通常返回一个初始值（如 0, '', True/False 等）。
2. **递归步骤 (Recursive Step):** 对 `link.first` 执行某些操作，然后结合对 `link.rest` 进行递归调用的结果。

**示例：计算链表长度**

```python
def len_link(link):
    """返回链表的长度 (递归实现)"""
    if link is Link.empty:
        return 0
    else:
        # 长度 = 1 (当前节点) + 剩余链表的长度
        return 1 + len_link(link.rest)
```

**示例：获取第 k 个元素**

```python
def getitem_link(link, k):
    """返回链表中索引为 k 的元素 (递归实现)"""
    if link is Link.empty:
        raise IndexError("Index out of bounds")
    elif k == 0:
        return link.first
    else:
        # 在剩余的链表中寻找第 k-1 个元素
        return getitem_link(link.rest, k - 1)
```



##### **迭代方法 (The Iterative Way)**

迭代方法使用 `while` 循环，通常更节省内存（因为它不增加函数调用栈的深度），但有时代码不如递归直观。

1. 初始化一个**“当前指针” (current pointer)** 指向链表的头部。
2. 初始化一个累加器（如 `count`）。
3. 使用 `while` 循环，当“当前指针”不指向 `Link.empty` 时，持续循环。
4. 在循环内部，更新累加器，并将“当前指针”移动到下一个节点 (`current = current.rest`)。

**示例：计算链表长度**

```python
def len_link_iterative(link):
    """返回链表的长度 (迭代实现)"""
    length = 0
    current = link
    while current is not Link.empty:
        length += 1
        current = current.rest
    return length
```

------



#### 4. 常用链表操作函数

这些函数是练习递归思想的绝佳例子。



##### `map_link(f, link)`

创建一个新链表，其元素是对原链表中每个元素应用函数 `f` 的结果。

```python
def map_link(f, link):
    if link is Link.empty:
        return Link.empty
    else:
        # 新链表的第一个元素是 f(link.first)
        # 新链表的剩余部分是对 link.rest 进行 map 的结果
        return Link(f(link.first), map_link(f, link.rest))
```



##### `filter_link(predicate, link)`

创建一个新链表，仅包含原链表中满足 `predicate` 函数（返回 `True`）的元素。

```python
def filter_link(predicate, link):
    if link is Link.empty:
        return Link.empty
    else:
        # 先对剩余部分进行 filter
        filtered_rest = filter_link(predicate, link.rest)
        if predicate(link.first):
            # 如果当前元素满足条件，则将其包含在新链表中
            return Link(link.first, filtered_rest)
        else:
            # 否则，直接返回过滤后的剩余部分
            return filtered_rest
```



##### `extend_link(link1, link2)`

将 `link2` 连接到 `link1` 的末尾，返回一个**新**的链表。

```python
def extend_link(link1, link2):
    if link1 is Link.empty:
        return link2
    else:
        # 递归地将 link2 连接到 link1.rest 的末尾
        return Link(link1.first, extend_link(link1.rest, link2))
```

------



#### 5. **【重要】** 可变性 (Mutability)

前面的 `map`, `filter`, `extend` 函数都创建并返回了**新**的链表，而没有修改原始链表。但链表本身是可变的数据结构。我们可以直接修改节点的 `first` 或 `rest` 属性。

- `link.first = new_value`: 修改当前节点的值。
- `link.rest = new_link`: 将当前节点的“下一个”指向一个全新的链表。

**示例：原地反转链表 (In-place Reverse)** 这是一个比较高级的操作，它通过修改 `rest` 指针来改变链表结构，并且只使用常数级别的额外空间。

```python
def reverse_link_inplace(link):
    """原地反转链表，不创建新节点。"""
    prev = Link.empty
    curr = link
    while curr is not Link.empty:
        # 核心操作：
        # 1. 保存下一个节点
        next_node = curr.rest
        # 2. 将当前节点的 rest 指向前一个节点
        curr.rest = prev
        # 3. 移动 prev 和 curr 指针
        prev = curr
        curr = next_node
    return prev # prev 现在是新的头节点
```

