## Lecture9 Trees

#### 1. 核心定义：树的递归本质 (The Recursive Definition of a Tree)

这是理解一切的基础，也是最重要的知识点。

- **定义**: 一棵树 (Tree) 包含一个 **标签 (Label)** 和一个由零或多个 **分支 (Branches)** 组成的列表。
- **递归关键点**: 每个 **分支 (Branch)** 本身也是一棵完整的树。

这个定义本身就是递归的。一棵大树是由许多小树（它的分支）组成的。这种结构决定了处理树的最佳方式就是使用递归函数。

**相关术语 (Key Terminology):**

- **节点 (Node)**: 树的任何一个位置都可以称为一个节点，每个节点都包含一个标签。
- **根节点 (Root)**: 整棵树最顶端的节点。
- **叶节点 (Leaf)**: 没有任何分支的节点（即其 `branches` 列表为空）。
- **父节点 (Parent) & 子节点 (Child)**: 如果树 A 的一个分支是树 B，那么 A 是 B 的父节点，B 是 A 的子节点。

------



#### 2. 在 Python 中表示树 (Representing a Tree in Python)

CS61A 中使用一个简洁的 `Tree` Class 来实现这个结构。这是将理论付诸实践的第一步。

**`Tree` 类的基本结构:**

```python
class Tree:
    def __init__(self, label, branches=[]):
        # 每个节点都有一个标签
        self.label = label
        # 验证每个分支确实是 Tree 对象
        for branch in branches:
            assert isinstance(branch, Tree)
        # 分支是一个包含 Tree 对象的列表
        self.branches = list(branches)

    def is_leaf(self):
        # 如果分支列表为空，它就是一个叶节点
        return not self.branches
```

**如何构建一棵树：** 这个过程是自底向上构建的。你先创建叶节点，然后用它们作为分支来创建它们的父节点，依此类推。

**示例:** 假设我们要构建如下结构的树：

```
      1
     / \
    2   3
       / \
      4   5
```

代码实现如下：

```python
# 先创建叶节点
leaf4 = Tree(4)
leaf5 = Tree(5)
leaf2 = Tree(2)

# 创建中间节点
node3 = Tree(3, [leaf4, leaf5])

# 创建根节点
root = Tree(1, [leaf2, node3])
```

------



#### 3. 【核心重点】树的递归处理 (Recursive Processing of Trees)

这是整个讲座的灵魂。几乎所有对树的操作都遵循一个固定的递归模式，这个模式被称为 **"Recursive Leap of Faith" (递归信任之跃)**。

**核心思想：**

1. **信任递归调用**: 假定你的递归函数已经能在**更小的同类问题**上完美工作。对于树来说，"更小的同类问题" 就是它的每一个**分支 (branch)**。
2. **处理当前层**: 你只需要考虑如何处理**当前节点 (current node)** 的标签，以及如何将所有分支（子问题）的处理结果组合起来，从而解决当前树（大问题）的问题。
3. **定义基本情况 (Base Case)**: 思考最简单的情况是什么。对于树来说，**叶节点 (leaf)** 通常就是基本情况。

下面通过几个经典函数来彻底理解这个模式。



##### 示例 1: 计算树中叶节点的数量 `count_leaves(t)`

- **目标**: 计算一棵树 `t` 中总共有多少个叶节点。
- **基本情况 (Base Case)**: 如果当前的树 `t` 是一个叶节点 (`t.is_leaf()` is True)，那么它自己就是一个叶子，所以结果是 1。
- **递归步骤 (Recursive Step)**: 如果 `t` 不是叶节点，它就没有贡献叶节点数。它的总叶节点数等于**其所有分支的叶节点数之和**。
  - **递归信任**: 相信 `count_leaves()` 能正确计算出每个分支的叶节点数。
  - **组合结果**: 将所有分支返回的结果用 `sum()` 加起来。

```python
def count_leaves(t):
    """计算树 t 中叶节点的数量"""
    if t.is_leaf():
        # 基本情况：如果当前是叶节点，返回 1
        return 1
    else:
        # 递归步骤：
        # 1. 对 t 的每一个分支 b，递归调用 count_leaves(b)
        # 2. 相信这个调用会返回分支 b 的叶节点数
        # 3. 将所有分支的结果加起来
        branch_counts = [count_leaves(b) for b in t.branches]
        return sum(branch_counts)

# 使用上面的 root 树
# count_leaves(root) -> count_leaves(leaf2) + count_leaves(node3)
#                   -> 1 + (count_leaves(leaf4) + count_leaves(leaf5))
#                   -> 1 + (1 + 1)
#                   -> 3
```



##### 示例 2: 将函数应用到树的每个节点 `map_tree(fn, t)`

- **目标**: 创建一棵与原树 `t` 结构完全相同的新树，但每个节点的标签都应用了 `fn` 函数。
- **基本情况**: 这里没有明显的“基本情况”，因为即使是叶节点也需要处理。递归的逻辑覆盖了所有情况。
- **递归步骤**:
  - **处理当前层**: 创建一个新标签，它是对当前节点 `t.label` 应用 `fn` 的结果，即 `fn(t.label)`。
  - **递归信任**: 相信 `map_tree(fn, b)` 能够为每一个分支 `b` 正确地创建出映射后的新分支。
  - **组合结果**: 将处理好的新标签和所有递归生成的新分支组合起来，创建一个新的 `Tree` 对象。

```python
def map_tree(fn, t):
    """返回一棵新树，其结构与 t 相同，但每个节点的标签都应用了 fn 函数。"""
    # 1. 处理当前节点的标签
    new_label = fn(t.label)

    # 2. 对每个分支 b，递归调用 map_tree(fn, b) 来生成新的分支
    new_branches = [map_tree(fn, b) for b in t.branches]

    # 3. 用新标签和新分支列表创建并返回一棵新树
    return Tree(new_label, new_branches)

# 示例：将树中每个节点的值都加倍
# doubled_tree = map_tree(lambda x: x * 2, root)
# doubled_tree 的结构会是:
#       2
#      / \
#     4   6
#        / \
#       8   10
```



##### 示例 3: 在树中查找值 `find(t, value)` (或者 `__contains__`)

- **目标**: 判断 `value` 是否存在于树 `t` 的任何一个节点的标签中。
- **基本情况**: 无需特定基本情况，逻辑统一。
- **递归步骤**:
  - **处理当前层**: 检查当前节点 `t.label` 是否等于 `value`。如果是，直接返回 `True`。
  - **递归信任**: 如果当前节点不匹配，那么需要检查它的任何一个分支是否包含 `value`。相信 `find(b, value)` 能正确判断。
  - **组合结果**: 如果当前节点不匹配，只要**任何一个 (any)** 分支的递归查找结果为 `True`，那么整个结果就是 `True`。

```python
def find(t, value):
    """如果 value 存在于树 t 的标签中，返回 True，否则返回 False。"""
    # 1. 检查当前节点的标签
    if t.label == value:
        return True

    # 2. 如果当前节点不匹配，则检查所有分支
    #    any() 函数会在遇到第一个 True 时立刻返回 True，非常高效
    return any([find(b, value) for b in t.branches])

# 我们可以把这个功能集成到 Tree 类中，成为 __contains__ 方法
# 这样就可以使用 'in' 关键字了
#
# class Tree:
#     ... (之前的代码) ...
#     def __contains__(self, value):
#         if self.label == value:
#             return True
#         return any([value in b for b in self.branches])
#
# # 使用:
# >>> 3 in root
# True
# >>> 10 in root
# False
```

------



#### 4. 学习建议与总结 (Summary & Study Tips)

1. **画图！画图！画图！**: 在处理任何树的问题时，先在纸上画出树的结构，以及递归调用栈的变化。这能帮你理清思路。
2. **牢记递归模式**: “处理当前节点 + 递归处理所有分支 + 组合结果”。练习用这个模式去解决各种问题，如“找到树的最大值”、“打印树的所有路径”等。
3. **自己动手实现**: 不要只看不练。亲手把 `Tree` 类和 `count_leaves`, `map_tree` 等函数敲一遍。尝试不看答案，独立写出来。
4. **理解数据抽象**: `Tree` 类本身是一个数据抽象。它的使用者不需要关心内部是如何用 `label` 和 `branches` 列表存储的，只需要调用提供的方法（如 `is_leaf()`）即可。这是CS61A强调的另一个重点。