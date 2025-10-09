## Lecture11 Iterators



### 1. 核心思想：什么是迭代？ (The Big Idea: What is Iteration?)

迭代是一种**遍历数据集合**中所有元素的统一方法。无论是列表（list）、元组（tuple）、字典（dict）还是自定义的序列，我们都希望有一种通用的方式来按顺序访问其中的每一个成员。`for` 循环就是我们最熟悉的用户接口，而其背后支撑这一切的就是**迭代器协议（Iterator Protocol）**。

------



### 2. 迭代器协议 (The Iterator Protocol) - ⭐重要核心⭐

这是整个讲座的基石。Python 的 `for` 循环之所以能作用于任何序列类型，就是因为它们都遵守了这个协议。该协议包含两个核心概念：



#### a. 可迭代对象 (Iterable)

- **定义**：一个对象如果拥有 `__iter__` 方法，那么它就是**可迭代的 (Iterable)**。

- **作用**：`__iter__` 方法的唯一职责就是**返回一个迭代器对象**。

- **例子**：

  - 常见的内置可迭代对象：`list`, `tuple`, `dict`, `str`, `range` 对象, 文件对象。
  - 你可以把可迭代对象想象成一个**数据容器**或**数据源**，它本身知道如何“生产”一个能帮你逐个访问其元素的工具（也就是迭代器）。

- **如何检查**：你可以通过调用内置函数 `iter()` 来获取一个可迭代对象的迭代器。如果一个对象不支持 `__iter__` 方法，调用 `iter(obj)` 会抛出 `TypeError`。

  ```python
  my_list = [1, 2, 3]
  # my_list 是一个可迭代对象 (Iterable)
  # 调用 iter() 会返回一个迭代器 (Iterator)
  my_iterator = iter(my_list)
  print(my_iterator) # <list_iterator object at ...>
  ```



#### b. 迭代器 (Iterator)

- **定义**：一个对象如果拥有 `__next__` 方法，那么它就是**迭代器 (Iterator)**。
- **作用**：`__next__` 方法负责返回序列中的**下一个**元素。它有两个关键行为：
  1. 返回下一个值。
  2. 当没有更多元素可以返回时，必须**抛出 `StopIteration` 异常**。这个异常不是一个错误，而是一个信号，用来通知调用者（比如 `for` 循环）迭代已经结束。
- **特性**：
  - **有状态 (Stateful)**：迭代器必须记住它在序列中的当前位置。每次调用 `__next__` 都会更新这个状态。
  - **一次性 (Single-use)**：一旦迭代器遍历完所有元素（即抛出 `StopIteration` 后），它就“耗尽”了，不能再次使用。如果想重新迭代，必须从原始的可迭代对象那里重新获取一个新的迭代器。

------



### 3. `for` 循环的幕后工作原理 (How the `for` Loop Really Works)

当我们写下 `for element in my_list:` 时，Python 解释器在背后执行了以下步骤：

1. **获取迭代器**：首先，它调用 `iter(my_list)` 来获取 `my_list` 这个可迭代对象的迭代器。
2. **循环调用 `next`**：然后，它进入一个无限循环，在每一次循环中调用这个迭代器的 `__next__` 方法，并将返回值赋给 `element`。
3. **捕获 `StopIteration`**：`for` 循环内部有一个 `try...except` 块。当 `__next__` 方法抛出 `StopIteration` 异常时，`for` 循环会捕获这个异常并优雅地退出循环，而不是让程序崩溃。

下面是 `for` 循环的等价 `while` 循环实现，这能更好地帮助你理解其机制：

```python
my_list = [1, 2, 3]

# 1. 从可迭代对象 my_list 获取迭代器
iterator = iter(my_list)

# 2. 用 while 循环模拟 for 循环
while True:
    try:
        # 3. 不断调用 next() 获取下一个元素
        element = next(iterator) # next(iterator) 等价于 iterator.__next__()
        print(element)
    except StopIteration:
        # 4. 当接收到 StopIteration 信号时，退出循环
        break
```

------



### 4. ⭐重要区别：可迭代对象 (Iterable) vs. 迭代器 (Iterator)

这是初学者最容易混淆的地方，必须清晰地区分。

| 特性         | 可迭代对象 (Iterable)                                        | 迭代器 (Iterator)                                      |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| **核心方法** | `__iter__()`                                                 | `__next__()` 和 `__iter__()`                           |
| **作用**     | 表示一个数据集合，可以被重复迭代。                           | 控制迭代过程，是有状态的，并且通常是一次性的。         |
| **比喻**     | 一本书（数据源）                                             | 一个书签（记录阅读位置的工具）                         |
| **使用**     | 你可以多次对一本书（Iterable）放置书签（获取 Iterator）来从头阅读。 | 一个书签（Iterator）只能在书中向前移动，读完就结束了。 |

**注意**：所有的迭代器自身也是可迭代的。这是因为迭代器也必须实现 `__iter__` 方法，并且这个方法通常只返回 `self`。这使得迭代器可以直接用在 `for` 循环中。

------



### 5. 创建自定义迭代器 (Creating Custom Iterators)

我们可以通过编写一个类来实现自己的迭代器。这在 CS61A 中是理解协议的重要练习。

**例子**：创建一个倒计时迭代器 `Countdown`。

```python
class Countdown:
    """
    一个从 start 倒数到 0 的自定义迭代器。
    """
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        # 迭代器需要 __iter__ 方法，通常返回它自己
        return self

    def __next__(self):
        # 核心逻辑：返回当前值，或在结束时抛出 StopIteration
        if self.current < 0:
            raise StopIteration
        else:
            value = self.current
            self.current -= 1
            return value

# 使用自定义迭代器
print("使用自定义 Countdown 迭代器:")
for i in Countdown(3):
    print(i) # 会依次打印 3, 2, 1, 0
```



### 6. 生成器 (Generators) - ⭐创建迭代器的优雅方式⭐

手动创建一个包含 `__init__`, `__iter__`, `__next__` 的类很繁琐。Python 提供了**生成器 (Generator)** 这一强大而简洁的工具来自动创建迭代器。



#### a. 生成器函数 (Generator Functions)

- **定义**：任何包含 `yield` 关键字的函数都是**生成器函数**。
- **工作原理**：
  - 调用一个生成器函数并不会立即执行它，而是返回一个**生成器对象 (generator object)**。这个对象就是一个功能完备的迭代器。
  - `yield` 关键字像 `return` 一样返回一个值，但它会**暂停**函数的执行并保存其所有局部状态（变量值、指令指针等）。
  - 当外界（如 `for` 循环）下一次调用 `next()` 时，函数会从它上次暂停的地方**恢复**执行，直到遇到下一个 `yield` 或者函数结束。
  - 如果函数执行完毕而没有更多的 `yield`，它会自动在底层抛出 `StopIteration`。

**例子**：用生成器函数重写 `Countdown`。

```python
def countdown_generator(start):
    """
    一个从 start 倒数到 0 的生成器函数。
    """
    current = start
    while current >= 0:
        yield current # 暂停并产生一个值
        current -= 1

# 使用生成器
print("\n使用 Countdown 生成器:")
for i in countdown_generator(3):
    print(i) # 效果完全相同：3, 2, 1, 0
```

可以看到，代码量大大减少，逻辑也更清晰。



#### b. 生成器表达式 (Generator Expressions)

- **定义**：它看起来像一个列表推导式（List Comprehension），但使用圆括号 `()` 而不是方括号 `[]`。
- **特点**：
  - **惰性求值 (Lazy Evaluation)**：它不会立即创建所有元素并放入内存，而是返回一个生成器对象。只有在你迭代它时，它才会逐个计算并产生值。
  - **内存高效**：对于处理大数据集非常有用，因为它不需要一次性将所有结果加载到内存中。

```python
# 列表推导式：立即创建列表，占用内存
list_comp = [x * x for x in range(10)] # [0, 1, 4, ..., 81]

# 生成器表达式：返回一个生成器对象，几乎不占内存
gen_exp = (x * x for x in range(10)) # <generator object <genexpr> at ...>

# 只有在迭代时才会计算值
print("\n使用生成器表达式:")
for num in gen_exp:
    print(num, end=' ')
```

------



### 7. 总结 (Key Takeaways)

1. **迭代器协议**是 Python 迭代的核心，由 `__iter__` 和 `__next__` 两个方法定义。
2. **可迭代对象 (Iterable)** 是数据源（如 `list`），有 `__iter__` 方法来返回一个迭代器。
3. **迭代器 (Iterator)** 是处理迭代过程的工具，有 `__next__` 方法来获取下一个元素，并在结束时抛出 `StopIteration`。
4. **`for` 循环**是迭代器协议的语法糖，它自动处理了 `iter()` 和 `next()` 的调用以及 `StopIteration` 的捕获。
5. **生成器**是通过 `yield` 关键字实现的函数，是创建迭代器的最常用、最高效的方式。
6. **生成器表达式** `(x for x in ...)` 提供了一种简洁的、内存高效的方式来创建一次性的生成器对象。