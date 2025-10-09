## Lecture3-Higher-Order Functions

#### **一、核心思想：函数是“一等公民” (Functions as First-Class Citizens)**

在 Python 中，函数与其他数据类型（如整数 `int`、字符串 `str`、列表 `list`）的地位是相同的。这意味着你可以像对待一个普通变量一样对待一个函数。具体体现在：

1. 可以把函数赋值给一个变量。
2. 可以把函数作为参数传递给另一个函数。
3. 可以把函数作为另一个函数的返回值。

**满足后面两条中任意一条的函数，就被称为高阶函数 (Higher-Order Function)。**

```python
def square(x):
    return x * x

# 1. 将函数赋值给变量
f = square 
print(f(5)) # 输出 25
```

------



#### **二、重要知识点 1：函数作为参数 (Functions as Arguments)**

这是 HOF 最直接的应用。通过将一个函数作为参数，我们可以把一个通用的计算框架和具体的操作逻辑分离开，从而实现强大的抽象。

**经典案例: `summation`**

`summation` 函数计算从 1 到 n 的一系列项的总和。它的特殊之处在于，每一项具体是什么，由一个作为参数传入的函数 `term` 来决定。

**代码示例:**

```python
def summation(n, term):
    """计算 term(1) + ... + term(n) 的和。
    
    >>> summation(5, lambda x: x) # 计算 1 + 2 + 3 + 4 + 5
    15
    >>> summation(5, lambda x: x * x) # 计算 1^2 + 2^2 + 3^2 + 4^2 + 5^2
    55
    """
    total = 0
    k = 1
    while k <= n:
        total = total + term(k)
        k = k + 1
    return total

# 为了配合 summation，我们可以定义各种 term 函数
def identity(x):
    return x

def square(x):
    return x * x

def cube(x):
    return x * x * x

# 使用
sum_of_naturals = summation(5, identity) # 计算 1 + 2 + 3 + 4 + 5
sum_of_squares = summation(5, square)   # 计算 1^2 + 2^2 + 3^2 + 4^2 + 5^2
```

**详细解释：**

- **抽象 (Abstraction):** `summation` 函数本身并不关心 `term` 到底是在做平方、立方还是其他任何操作。它只负责建立一个"从1到n循环并累加"的框架。这种将**“做什么”**（由 `term` 决定）和**“怎么做”**（由 `summation` 的循环和累加逻辑决定）分离的思想是编程中最重要的抽象手段之一。
- **通用性 (Generality):** 我们只写了一次 `summation`，就可以用它来解决求自然数和、求平方和、求立方和等一系列问题，只需要提供不同的 `term` 函数即可。这极大地提高了代码的复用性。

------



#### **三、重要知识点 2：函数作为返回值 (Functions as Return Values) 与闭包 (Closures)**

这是 HOF 更强大、也更难理解的部分。一个函数可以创建并返回另一个函数。

**经典案例: `make_adder`**

`make_adder(n)` 函数接受一个参数 `n`，然后返回**一个新的函数**。这个新的函数接受一个参数 `k`，并返回 `n + k`。

**代码示例:**

```python
def make_adder(n):
    """返回一个新的函数，这个函数接受一个参数 k，并返回 n + k。"""
    def adder(k):
        return n + k
    return adder

# 使用
add_five = make_adder(5) # add_five 现在是一个函数，它“记住”了 n=5
add_ten = make_adder(10) # add_ten 是另一个函数，它“记住”了 n=10

print(add_five(3)) # 输出 8 (因为 5 + 3)
print(add_ten(3))  # 输出 13 (因为 10 + 3)
```

**⭐ 详细解释 (这是本讲最核心的难点): 环境图 (Environment Diagrams) 与闭包 (Closure)**

为什么 `add_five(3)` 在被调用时，能够知道 `n` 是 `5`？明明 `make_adder(5)` 的调用已经结束了。答案就在于**环境图**和**闭包**。

1. **环境图 (Environment Diagram):** 这是 CS61A 用来追踪变量和函数作用域的工具。它由**帧 (Frame)** 组成。
   - **全局帧 (Global Frame):** 包含所有全局定义的变量和函数。
   - **函数帧 (Function Frame):** 每当一个函数被调用时，就会创建一个新的帧。这个帧包含函数的局部变量（包括参数）。
2. **闭包的关键：父指针 (Parent Pointer)**
   - 当你在一个函数内部 (`make_adder`) 定义了另一个函数 (`adder`) 时，内部函数 (`adder`) 会被创建一个**父指针**，指向其被定义的那个环境帧（在这里是 `make_adder` 的帧）。
   - 当 `make_adder(5)` 被调用时，会创建一个 `make_adder` 的帧，其中 `n` 被绑定到 `5`。
   - `make_adder` 返回了 `adder` 函数。虽然 `make_adder` 的调用结束了，但因为返回的 `adder` 函数的父指针还指向着 `make_adder` 的帧，所以这个帧**不会被销毁**。它会一直存在，以供 `adder` 函数使用。
   - 当你调用 `add_five(3)` (也就是 `adder(3)`) 时，会创建一个新的 `adder` 帧，其中 `k` 绑定到 `3`。
   - 在执行 `return n + k` 时，Python 在 `adder` 帧里找到了 `k` (值为3)，但找不到 `n`。于是它顺着 `adder` 函数的**父指针**找到了 `make_adder` 的帧，并在那里找到了 `n` (值为5)。
   - 最终计算出 `5 + 3` 并返回 `8`。
3. **什么是闭包 (Closure)?**
   - 一个**闭包**就是一个函数，它“记住”了自己被创建时的环境。
   - 在我们的例子中，`add_five` 就是一个闭包。它由 `adder` 函数本身和它对 `n=5` 这个环境的引用共同组成。

------



#### **四、重要知识点 3：Lambda 表达式 (Lambda Expressions)**

有时候，我们需要传递给 HOF 的函数非常简单，只用一次，专门用 `def` 来定义显得很麻烦。Lambda 表达式提供了一种创建**匿名 (anonymous) 短小函数**的简洁语法。

**语法:**

```
lambda <parameters>: <expression>
```

- `lambda` 是关键字。
- `<parameters>` 是参数列表，和普通函数一样。
- `:` 后面是一个**单一的表达式**。这个表达式的计算结果就是函数的返回值。**注意：Lambda 函数内部不能包含复杂的语句，只能有一个表达式。**

**代码示例:**

将 `summation` 的调用变得更简洁：

```python
# 使用 def 定义函数
def square(x):
    return x * x
sum_of_squares = summation(5, square)

# 使用 lambda 表达式，无需提前定义函数
sum_of_squares_lambda = summation(5, lambda x: x * x) 

# 两者结果完全相同
print(sum_of_squares == sum_of_squares_lambda) # 输出 True
```

Lambda 表达式本质上是语法糖 (syntactic sugar)，它让代码更紧凑，尤其是在配合 `map`, `filter` 等函数时非常方便。

------



#### **五、总结与学习建议**

1. **核心是抽象:** HOF 的威力在于它提供了一种强大的抽象机制，让你能够编写更通用、更灵活、更可复用的代码。
2. **区分两种 HOF:**
   - **接受函数为参数:** 相对容易理解，关键在于将通用逻辑与具体操作分离。
   - **返回值为函数:** 这是难点，关键在于理解**闭包**。
3. **画图！画图！画图！** 对于闭包，**亲手画环境图**是唯一的、也是最好的理解方式。务必练习课程作业中画环境图的题目，直到你能够不假思索地画出 `make_adder` 的调用过程。
4. **Lambda 是工具:** Lambda 表达式本身不难，它只是一个简化函数定义的工具，目的是为了更方便地使用 HOF。