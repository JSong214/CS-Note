## Lecture2-Control

在编程中，“控制”指的是指令（statements）的执行顺序。默认情况下，Python 解释器会从上到下一行一行地执行代码。但这种方式功能有限，我们需要更强大的工具来处理复杂的逻辑，例如根据不同条件执行不同代码，或者重复执行某段代码。这就是“控制流”（Control Flow）语句的作用。



### 1. 语句 (Statements)

在学习控制之前，首先要理解什么是“语句”。

- **赋值语句 (Assignment Statements):** 例如 `x = 5`，它将一个值绑定到一个名称上。
- **表达式语句 (Expression Statements):** 任何一个表达式本身都可以成为一个语句，比如 `2 + 3` 或者 `print('hello')`。执行它意味着对这个表达式求值。
- **复合语句 (Compound Statements):** 这是控制流的核心。它由一个“头部”（header）和一个或多个“子句”（clauses）组成。头部控制着子句的执行方式。

复合语句的通用结构是：

```python
<header>:
    <statement>
    <statement>
    ...
```

**关键点:**

- 头部以冒号 (`:`) 结尾。
- 头部下面的代码块（子句）必须 **缩进**。在 Python 中，缩进是语法的一部分，用来表示代码的层级和归属关系。

------



### 2. 条件执行: `if-elif-else` 语句

这是最基本的决策工具，它允许程序根据一个条件的真假来选择执行哪个代码块。



#### 2.1 `if` 语句

如果 `<condition>` 求值结果为真（`True`），则执行其下的代码块。

```python
if <condition>:
    <suite of statements>
```

**示例:**

```python
x = 10
if x > 5:
    print("x is greater than 5")  # 这行会被执行
if x < 0:
    print("x is negative")        # 这行不会被执行
```



#### 2.2 `if-else` 语句

提供了一个备用方案。如果 `if` 的条件为假（`False`），则执行 `else` 下的代码块。

```python
if <condition>:
    <suite for True>
else:
    <suite for False>
```

**示例:**

```python
age = 17
if age >= 18:
    print("You are an adult.")
else:
    print("You are a minor.")  # 这行会被执行
```



#### 2.3 `if-elif-else` 链

用于处理多个互斥的条件。Python 会从上到下依次检查每个条件，一旦找到一个为真的，就执行其对应的代码块，并忽略所有剩下的 `elif` 和 `else`。

```python
if <condition1>:
    <suite1>
elif <condition2>:
    <suite2>
...
else:
    <suite_final>
```

**示例:**

```python
score = 85
if score >= 90:
    print("Grade: A")
elif score >= 80:
    print("Grade: B")  # 这行会被执行，即使 score >= 60 也成立
elif score >= 60:
    print("Grade: C")
else:
    print("Grade: F")
```

**❗重要知识点详解: `if` 语句与环境模型 (Environment Diagrams)**

在 CS61A 的环境模型中，`if-elif-else` 语句 **不会** 创建新的帧 (Frame)。所有的条件判断和代码块的执行都发生在当前的帧中。这意味着在 `if` 或 `else` 语句块内部创建或修改的变量，会直接影响当前帧。这一点与函数调用（会创建新帧）有本质区别。

------



### 3. 布尔逻辑与表达式 (Boolean Logic & Expressions)

`if` 和 `while` 语句的头部 `<condition>` 的核心是布尔逻辑。



#### 3.1 布尔类型 (`bool`)

只有两个值：`True` 和 `False`。



#### 3.2 布尔上下文 (Boolean Context) 和 “Truthy/Falsy” 值

在需要布尔值的地方（如 `if` 语句的条件中），Python 会自动将其他类型的值转换为布尔值。

- **Falsy (被视为 `False`) 的值:** `None`, `False`, `0` (所有数值类型的零), 空字符串 `""`, 空列表 `[]`, 空字典 `{}` 等。
- **Truthy (被视为 `True`) 的值:** 其他所有值。

**示例:**

```python
if 0:
    print("This will not print.")
if "hello":
    print("This will print.")
```



#### 3.3 逻辑运算符 (Logical Operators)

- `not <expr>`: 如果 `<expr>` 为 `Falsy`，则返回 `True`；否则返回 `False`。
- `<left> and <right>`: **短路求值 (Short-Circuiting)**。
  1. 先对 `<left>` 求值。
  2. 如果 `<left>` 的结果是 `Falsy`，则整个表达式的结果就是 `<left>` 的值，并且 **不会** 对 `<right>` 求值。
  3. 如果 `<left>` 的结果是 `Truthy`，则对 `<right>` 求值，整个表达式的结果就是 `<right>` 的值。
- `<left> or <right>`: **短路求值 (Short-Circuiting)**。
  1. 先对 `<left>` 求值。
  2. 如果 `<left>` 的结果是 `Truthy`，则整个表达式的结果就是 `<left>` 的值，并且 **不会** 对 `<right>` 求值。
  3. 如果 `<left>` 的结果是 `Falsy`，则对 `<right>` 求值，整个表达式的结果就是 `<right>` 的值。

**❗重要知识点详解: 短路求值 (Short-Circuiting)**

这是布尔运算中一个至关重要的特性，经常用于编写更安全、更简洁的代码。

**`and` 的例子:**

```python
x = 5
y = 0
# 如果 y 是 0，x / y 会导致错误。但因为短路，它永远不会被执行。
if y != 0 and x / y > 1:
    print("This part is safe.")
else:
    print("y is zero, division was skipped.") # 这行会被执行
```

因为 `y != 0` 是 `False`，Python 立即停止计算，`x / y > 1` 完全被跳过。

**`or` 的例子:**

```python
def is_valid(x):
    print(f"Checking {x}...")
    return x > 0

# is_valid(5) 返回 True，Python 立即停止，不会再调用 is_valid(-1)。
# 所以你只会看到 "Checking 5..."
if is_valid(5) or is_valid(-1):
    print("At least one is valid.")
```

------



### 4. 循环控制: `while` 循环

`while` 循环用于在某个条件保持为真的情况下，重复执行一个代码块。



#### 4.1 `while` 语句结构

```python
while <condition>:
    <suite of statements>
```

**执行流程:**

1. **检查条件:** 对 `<condition>` 求值。
2. **判断:**
   - 如果条件为真 (`True`)，则执行 `while` 循环体内的代码块。执行完毕后，**返回步骤 1**。
   - 如果条件为假 (`False`)，则跳过循环体，继续执行 `while` 循环之后的代码。

**示例：从 1 加到 5**

```python
total, i = 0, 1
while i <= 5:
    total = total + i
    i = i + 1
print(total)  # 输出 15
```

**执行分析:**

- i=1, total=1
- i=2, total=3
- i=3, total=6
- i=4, total=10
- i=5, total=15
- i=6, `i <= 5` 为 `False`，循环结束。

**❗重要知识点详解: 避免无限循环 (Infinite Loops)**

使用 `while` 循环时，最常见的错误就是忘记在循环体内更新与条件相关的变量，导致条件永远为真，程序陷入无限循环。

**错误示例:**

```python
i = 0
while i < 5:
    print(i)
    # 错误！没有 i = i + 1，i 的值永远是 0，条件永远为真！
```

**必须确保** 循环体内的某些操作最终能让循环条件变为 `False`。

------



### **总结与 CS61A 思想连接**

- **控制是构建抽象的基石:** `if` 和 `while` 语句是程序员的“工具箱”。通过组合它们，我们可以构建出复杂的函数和算法，实现任何计算逻辑。例如，你可以写一个函数 `is_prime(n)`，内部通过 `while` 循环和 `if` 判断来检查一个数是否为素数。这本身就是一种 **过程抽象 (Procedural Abstraction)**。
- **万物皆可算:** 理论上，仅用赋值、`if` 和 `while` 语句（加上基本的算术和逻辑运算）就足以表达所有可计算的问题。这体现了计算机科学中一个深刻的思想：用简单的构建块组合出无限的可能性。
- **代码的可读性:** `if-elif-else` 的结构化设计让复杂的决策逻辑变得清晰。良好的缩进和结构是写出可维护代码的关键。