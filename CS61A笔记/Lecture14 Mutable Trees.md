## Lecture14 Mutable Trees 



#### 1. 核心概念：什么是树 (Tree)？

在深入“可变性”之前，首先要理解“树”这种数据结构。

- **定义**：树是一种**分层**的数据结构，由**节点 (Node)** 和连接节点的**边 (Edge)** 组成。
- **关键术语**：
  - **节点 (Node)**：树的基本组成部分，包含一个**标签 (Label)**（或称为值/value/entry）和指向其他节点的引用。
  - **根节点 (Root)**：树最顶层的节点，是唯一没有父节点的节点。
  - **分支 (Branches)**：从一个节点延伸出的连接，指向其**子树 (Sub-trees)**。一个节点的所有子树构成一个列表。
  - **子节点 (Child)** 和 **父节点 (Parent)**：如果节点A通过一条边连接到下方的节点B，那么A是B的父节点，B是A的子节点。
  - **叶节点 (Leaf)**：没有任何子节点的节点（即，其分支列表为空）。

**重要思想**：树的定义本身就是**递归**的。一棵树由一个根节点和它的一组分支组成，而每个分支本身又是一棵完整的树。这个递归定义是理解树相关算法的关键。



#### 2. Python 中的树表示法 (The `Tree` Class)

CS61A 中使用面向对象编程 (OOP) 来定义树。这使得数据（标签和分支）和操作（方法）能够被优雅地封装在一起。

```python
class Tree:
    """
    一个包含标签(label)和分支(branches)列表的树。
    每个分支本身也是一个 Tree 对象。
    """
    def __init__(self, label, branches=[]):
        self.label = label
        # 防御性拷贝，防止多个树共享同一个分支列表
        for b in branches:
            assert isinstance(b, Tree)
        self.branches = list(branches)

    def is_leaf(self):
        """如果树没有分支，则返回 True。"""
        return not self.branches # 或者 len(self.branches) == 0

    def __repr__(self):
        """生成树的字符串表示形式，方便调试。"""
        if self.branches:
            branch_str = ', ' + repr(self.branches)
        else:
            branch_str = ''
        return f'Tree({self.label}{branch_str})'

    def __str__(self):
        """更美观的打印形式（通常通过一个递归的 print_tree 函数实现）"""
        return '\n'.join(self.indented())

    def indented(self, k=0):
        """带缩进的打印辅助函数"""
        indent = '  ' * k
        yield indent + str(self.label)
        for b in self.branches:
            yield from b.indented(k + 1)
```

**详细解释**：

- `__init__(self, label, branches=[])`: 这是构造函数。
  - `self.label`: 存储节点的值。
  - `self.branches`: 存储一个列表，列表中的**每一个元素都必须是另一个 `Tree` 对象**。这是实现递归结构的核心。
  - `self.branches = list(branches)`: 这是一个很重要的细节，叫做**防御性拷贝 (defensive copy)**。如果直接写 `self.branches = branches`，那么当外部的列表变化时，树的结构也会意外地改变。通过 `list()` 创建一个新的列表副本，可以避免这种副作用。
- `is_leaf(self)`: 这是一个非常有用的辅助方法，用于判断一个节点是否是叶节点。它是许多递归算法的**基线条件 (Base Case)**。



#### 3. **核心重点：可变性 (Mutability)**

这是本讲的灵魂。所谓“可变”，意味着我们可以在**不创建新树**的情况下，直接修改现有树的内部状态。

- **什么可以被改变？**
  1. **节点的标签 (Label)**: 我们可以直接给 `t.label` 赋一个新值。
  2. **节点的分支 (Branches)**: 我们可以修改 `t.branches` 这个列表，比如：
     - 添加一个新的子树：`t.branches.append(another_tree)`
     - 删除一个子树：`t.branches.pop()` 或 `del t.branches[i]`
     - 甚至清空所有子树：`t.branches.clear()`

**与不可变数据结构（如元组）的对比**: 如果你用元组来表示树（例如 `(label, (branch1, branch2))`），那么你无法修改它。要“改变”一个元组树，你必须创建一个全新的元组。而对于可变树，修改是**就地的 (in-place)**。

**示例**:

```python
# 创建一个简单的树
t1 = Tree(3, [Tree(5), Tree(7)])
print(t1)
# >>> Tree(3, [Tree(5), Tree(7)])

# 1. 修改根节点的标签
t1.label = 4
print(t1)
# >>> Tree(4, [Tree(5), Tree(7)])

# 2. 添加一个新的分支
t1.branches.append(Tree(9))
print(t1)
# >>> Tree(4, [Tree(5), Tree(7), Tree(9)])
```

**重要性**: 可变性使得对树的结构进行剪枝、嫁接等操作变得非常高效。但它也带来了副作用的风险：如果两部分代码共享对同一棵树（或子树）的引用，一方的修改会影响到另一方。



#### 4. 树的递归处理 (Recursive Processing)

对树进行操作的函数几乎都是递归的，这与树的递归定义完美契合。

**通用递归模式**:

1. **基线条件 (Base Case)**: 通常是当处理到叶节点 (`t.is_leaf()`) 时，直接返回一个结果。
2. **递归步骤 (Recursive Step)**:
   - 对当前节点的 `label` 进行某种处理。
   - 对 `t.branches` 中的**每一个子树**，递归地调用函数本身。
   - 将从递归调用中获得的结果进行合并或处理。

**经典递归函数示例**:

**A. `count_leaves(t)`: 计算叶节点的数量**

```python
def count_leaves(t):
    """计算树 t 中叶节点的数量。"""
    if t.is_leaf():
        return 1
    else:
        # 将所有分支的叶节点数量相加
        # branch_counts = [count_leaves(b) for b in t.branches]
        # return sum(branch_counts)
        # 更简洁的写法：
        return sum([count_leaves(b) for b in t.branches])
```

- **思路**: 如果当前节点是叶子，它自己就是1个叶子。如果不是，那它总的叶子数等于它所有子树的叶子数之和。

**B. `map_tree(fn, t)`: 对树中每个节点的标签应用一个函数（原地修改）** 这个函数体现了**可变性**。

```python
def map_tree(fn, t):
    """将函数 fn 应用于树 t 中每个节点的标签上（原地修改）。"""
    t.label = fn(t.label) # 修改当前节点的标签
    for b in t.branches:
        map_tree(fn, b) # 递归地对每个子树进行同样的操作
```

- **思路**: 先把自己节点的标签改了，然后“命令”每个子树也去做同样的事情。因为是原地修改，所以函数不需要 `return` 任何东西。

**C. `prune_tree(t, value)`: 剪枝（原地修改）** 这是一个更复杂的原地修改操作，也是面试中常见的问题。它会删除所有标签等于 `value` 的子树。

```python
def prune(t, value):
    """从树 t 中剪掉所有标签为 value 的子树（原地修改）。"""
    # 重点：不能在遍历列表的同时修改它。
    # 所以我们创建一个新的列表来存放需要保留的分支。
    t.branches = [b for b in t.branches if b.label != value]

    # 然后对保留下来的每一个分支，继续进行剪枝操作。
    for b in t.branches:
        prune(b, value)
```

- **详细解释**:
  - `t.branches = [b for b in t.branches if b.label != value]`: 这一行是关键。它使用列表推导式 (List Comprehension) 创建了一个**新的**分支列表，这个新列表只包含那些标签不等于 `value` 的子树。然后用这个新列表覆盖掉旧的 `t.branches`。
  - **为什么不能直接在 `for` 循环里 `remove`**？在 Python 中，当你在遍历一个列表时，如果从中删除元素，会导致迭代器混乱，可能会跳过某些元素，引发 bug。
  - `for b in t.branches:`: 在修剪完当前层的分支后，需要继续递归地进入到**未被剪掉的**子树中去执行相同的剪枝逻辑。



#### 5. 总结与备忘

- **树是递归的**：理解这一点是解决所有树问题的基础。
- **`Tree` 类封装了数据和行为**：`label` 和 `branches` 是数据，`is_leaf` 是行为。
- **可变性是双刃剑**：
  - **优点**：原地修改，高效，直观（如“剪枝”）。
  - **缺点**：容易产生意想不到的副作用（aliasing），当多个变量指向同一个 `Tree` 对象时要特别小心。
- **递归是处理树的“瑞士军刀”**：
  - 永远先思考**基线条件**（通常是叶节点）。
  - 然后思考如何将子问题的解（对子树的递归调用结果）合并成当前问题的解。
- **练习是关键**: 务必亲手实现 `count_leaves`, `map_tree`, `prune`, `find_path` (查找从根到某个值的路径) 等函数，才能真正掌握这些知识点。