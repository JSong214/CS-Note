## Lecture5 Functional Abstraction

#### 核心思想：识别并泛化计算模式

函数抽象的核心思想是 **“Don't Repeat Yourself” (DRY)** 的升华。当你发现自己在写多个功能相似的函数时，就应该思考如何将它们共通的“计算模式”或“逻辑结构”抽象出来，创建一个更通用的函数。

> **学习目标：** 从编写解决特定问题的函数（例如：计算1到n的自然数之和），提升到编写可以解决一类问题的**高阶函数**（例如：计算1到n任意序列的和）。

------



### 一、重要知识点详解



#### 1. 高阶函数 (Higher-Order Functions, HOFs)

这是函数抽象这一讲**最核心、最重要的概念**。

> **定义：** 高阶函数是指满足以下至少一个条件的函数：
>
> 1. **接受一个或多个函数作为参数。**
> 2. **返回一个函数作为其结果。**

在 Python 中，函数是“一等公民 (First-class citizens)”，意味着它们可以像任何其他数据类型（如整数、字符串）一样被传来传去。这是实现高阶函数的基础。

**详细解释与示例：**

我们从一个具体的例子出发，理解为什么需要高阶函数。

**步骤 1：发现重复 (Identifying Repetition)**

假设我们需要写两个函数：

- `sum_naturals(n)`: 计算从 1 到 n 的自然数之和 (1 + 2 + 3 + ... + n)
- `sum_cubes(n)`: 计算从 1 到 n 的所有数的立方和 (1³ + 2³ + 3³ + ... + n³)

```python
# 特定功能的函数
def sum_naturals(n):
    total, k = 0, 1
    while k <= n:
        total, k = total + k, k + 1
    return total

def sum_cubes(n):
    total, k = 0, 1
    while k <= n:
        total, k = total + (k*k*k), k + 1
    return total

print(sum_naturals(5)) # 输出 15 (1+2+3+4+5)
print(sum_cubes(5))    # 输出 225 (1+8+27+64+125)
```

观察这两个函数，你会发现它们的**结构几乎完全一样**：

- 都有一个 `total` 累加器和 `k` 计数器。
- 都用 `while k <= n` 进行循环。
- 都在循环结束时 `k` 增加 1。

**唯一的不同点**在于每次循环中加到 `total` 上的值：

- `sum_naturals` 中是 `k` 本身。
- `sum_cubes` 中是 `k` 的立方，即 `k*k*k`。

**步骤 2：抽象与泛化 (Abstraction and Generalization)**

这个“每次要加上的值”实际上是一个基于 `k` 的计算。我们可以把这个计算本身变成一个参数，让它“插拔式”地替换。这个参数就是一个函数！

我们来创建一个通用的求和函数 `summation`：

```python
# 通用求和函数 (高阶函数)
def summation(n, term):
    """
    对序列 1, ..., n 中的每一项 k，应用 term(k) 函数，然后求和。
    
    n: 序列的上限
    term: 一个函数，接受一个数字 k 并返回一个值
    """
    total, k = 0, 1
    while k <= n:
        total, k = total + term(k), k + 1
    return total
```

在这个 `summation` 函数中：

- `term` 是一个**函数参数**。它代表了我们想要对每个 `k` 应用的“规则”。
- `summation` 函数本身并不知道 `term` 具体是什么，它只负责调用 `term(k)`。

**步骤 3：使用高阶函数重构代码**

现在，我们可以用这个通用的 `summation` 函数非常简洁地实现原来的两个功能：

```python
# 用于 term 参数的具体函数
def identity(k):
    return k

def cube(k):
    return k*k*k

# 使用高阶函数重定义 sum_naturals 和 sum_cubes
def sum_naturals_new(n):
    return summation(n, identity)

def sum_cubes_new(n):
    return summation(n, cube)

print(sum_naturals_new(5)) # 同样输出 15
print(sum_cubes_new(5))    # 同样输出 225
```

**优势总结：**

- **代码复用**：我们把公共的循环逻辑提取到了 `summation` 中，避免了重复代码。
- **逻辑清晰**：`summation` 的名字和参数清晰地表达了它的作用——“对一个序列按某种规则(term)求和”。
- **易于扩展**：如果现在需要计算平方和 `sum_squares`，我们只需要定义一个新的 `term` 函数 `square(k)`，而无需重写整个循环逻辑。

------



#### 2. Lambda 表达式 (Lambda Expressions)

当 `term` 函数非常简单时（比如 `identity` 或 `cube`），专门用 `def` 去定义一个有名字的函数显得有些繁琐，特别是如果这个函数只用一次。Lambda 表达式为此提供了便利。

> **定义：** Lambda 表达式是一种创建匿名函数（没有名字的函数）的简洁语法。

**语法格式：** `lambda <参数>: <表达式>`

- `lambda` 是关键字。
- `<参数>` 是函数的输入。
- `<表达式>` 是函数的返回值，它必须是一个单行表达式。

**详细解释与示例：**

使用 Lambda 表达式可以进一步简化我们上面的代码，不再需要 `def identity` 和 `def cube`。

```python
# 使用 Lambda 表达式直接作为参数传递
def sum_naturals_lambda(n):
    return summation(n, lambda k: k) # lambda k: k 等价于 identity 函数

def sum_cubes_lambda(n):
    return summation(n, lambda k: k*k*k) # lambda k: k*k*k 等价于 cube 函数

print(sum_naturals_lambda(5)) # 15
print(sum_cubes_lambda(5))    # 225
```

**要点：**

- Lambda 表达式创建的是一个**函数对象**，可以直接传递给高阶函数。
- 它让代码更紧凑，尤其是在函数式编程风格中非常常用。

------



#### 3. 纯函数 (Pure Functions) vs. 非纯函数 (Non-Pure Functions)

这是函数式编程中的一个重要概念，有助于写出更可预测、更可靠的代码。

> **纯函数 (Pure Function):**
>
> 1. **无副作用 (No Side Effects):** 函数的执行除了返回一个值之外，不改变任何外部状态。例如，不修改全局变量、不打印输出、不读写文件等。
> 2. **引用透明 (Referential Transparency):** 对于相同的输入，永远返回相同的输出。

**示例：**

- `cube(k)` 是一个纯函数。只要输入是 `3`，输出永远是 `27`。它不依赖任何外部变量，也不改变任何东西。
- `summation(n, term)` 如果 `term` 是纯函数，那么 `summation` 也是纯函数。

> **非纯函数 (Non-Pure Function):** 违反了纯函数定义的任何一条。最常见的例子是 `print()`。

**示例：**

```python
def print_and_return(x):
    print(x) # 这是一个副作用（向屏幕输出）
    return x
```

`print_and_return` 是非纯函数。即使对于相同的输入 `5`，它每次都会在屏幕上产生新的输出，改变了“世界的状态”。

**为什么纯函数很重要？**

- **可预测性与可靠性**：纯函数的行为完全由其输入决定，非常容易理解和预测。
- **易于测试**：测试纯函数只需要提供输入并检查输出即可，无需关心外部环境。
- **并行计算**：纯函数之间没有依赖和状态争用，非常适合并行执行。