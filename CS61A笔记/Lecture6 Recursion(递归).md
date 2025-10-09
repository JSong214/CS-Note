## Lecture6 Recursion(递归)



#### **1. 什么是递归？(What is Recursion?)**

递归是一种解决问题的强大思想，它的核心是：**一个函数直接或间接地调用自身**。

从概念上讲，递归是将一个大问题分解为一个或多个与原问题性质相同、但规模更小的子问题。通过解决这些子问题，最终得到大问题的解。

可以把它想象成俄罗斯套娃：你想知道最里面的娃娃是什么样的，于是你打开了最外层的娃娃，发现里面还有一个小一点的娃娃。你继续打开，直到找到最后一个、无法再打开的娃娃。这个过程就是递归。

------



#### **2. 递归函数设计的两大核心要素**

任何一个正确的递归函数都必须包含两个关键部分：

1. **基本情况 (Base Case):**
   - **定义**: 这是递归的**停止条件**。它是一个或多个最简单、可以直接求解的情况，不需要再次进行递归调用。
   - **重要性**: **没有基本情况，递归将无限进行下去**，直到耗尽所有计算资源（在 Python 中通常表现为 `RecursionError: maximum recursion depth exceeded`，即“栈溢出”）。
   - **如何寻找**: 思考“这个问题最简单的版本是什么？”。例如，计算阶乘 `n!`，最简单的情况是 `0!` 或 `1!`，它们的值都是 1。
2. **递归步骤 (Recursive Step):**
   - **定义**: 在这一步，函数会**调用自身**来解决一个**规模更小**的子问题。
   - **重要性**: 这一步必须确保每次调用都在向“基本情况”靠近。如果问题规模没有减小，同样会导致无限递归。
   - **如何设计**: 思考“我如何利用一个更小规模问题的解来构建当前问题的解？”。例如，`n!` 可以通过 `(n-1)!` 得到，关系是 `n! = n * (n-1)!`。这里的 `factorial(n-1)` 就是递归调用。

------



#### **3. 【课程精髓】信任之跃 (The Leap of Faith) - 设计递归的核心思想**

这是 CS61A 教学中反复强调的一个概念，也是你必须掌握的思维方式。

**“信任之跃”指的是：在设计递归函数时，你要完全相信你的递归调用是能够正确工作的。**

具体步骤如下：

1. **处理基本情况**: 首先定义好最简单的、能直接出答案的 Base Case。
2. **做出“信任之跃”**: **假设** `recursive_function(smaller_problem)` 已经可以完美地返回你期望的、关于那个更小问题的正确答案。**你不需要去思考这个递归调用内部是如何一步步执行的**，那是计算机的工作。
3. **组合解**: 思考最后一个问题：“如果我已经拿到了子问题的答案，我需要做什么操作来得到当前这个更大问题的答案？”

**举例：`factorial(n)`**

1. **Base Case**: 如果 `n` 是 1，直接返回 1。
2. **Leap of Faith**: 我**相信** `factorial(n-1)` 能够正确地计算出 `(n-1)!` 的值。
3. **组合**: 如果我手里已经有了 `(n-1)!` 的结果，我只需要将它乘以 `n`，就能得到 `n!`。所以，`return n * factorial(n-1)`。

这种思想让你从繁琐的调用栈（Call Stack）追踪中解脱出来，专注于问题分解的逻辑，是递归编程的真正艺术所在。



#### **4. 递归的种类**

**A. 线性递归 (Linear Recursion)**

函数在每次调用中最多只调用自身一次。前面讲的阶乘就是典型的线性递归。其计算过程像一条链。

```python
def factorial(n):
  """Computes the factorial of n.

  >>> factorial(4)
  24
  """
  if n == 0 or n == 1: # Base Case
    return 1
  else: # Recursive Step
    return n * factorial(n - 1) # Leap of Faith: assume factorial(n-1) works
```

调用过程: `factorial(4) -> factorial(3) -> factorial(2) -> factorial(1)`

**B. 树形递归 (Tree Recursion)**

函数在一次执行中可能会**调用自身多次**。这种递归的计算过程像一棵树。

**经典案例：斐波那契数列 (Fibonacci Sequence)** `fib(n) = fib(n-1) + fib(n-2)`

```python
def fib(n):
  """Computes the nth Fibonacci number.

  >>> fib(8)
  21
  """
  if n == 0: # Base Case 1
    return 0
  elif n == 1: # Base Case 2
    return 1
  else: # Recursive Step
    # Leap of Faith: assume fib(n-1) and fib(n-2) both work
    return fib(n - 1) + fib(n - 2)
```

**重要解释**: 在计算 `fib(4)` 时，会发生以下情况：

```
          fib(4)
         /      \
      fib(3)   +   fib(2)
     /    \       /    \
 fib(2) + fib(1) fib(1) + fib(0)
 /    \
fib(1)+fib(0)
```

你可以看到 `fib(2)` 被重复计算了两次。这种朴素的树形递归虽然在逻辑上很清晰，但效率通常很低，因为它会进行大量的重复计算。这也是后续课程引入“记忆化 (Memoization)”和“动态规划 (Dynamic Programming)”的原因。

------



#### **5. 迭代式递归 (Iterative Recursion / Tail Recursion)**

这是一种特殊的递归模式，它通过传递额外的参数（通常称为累加器 `accumulator`）来维护状态，从而模拟迭代的过程。它的特点是递归调用是函数中**最后执行**的操作。

**对比：计算数字各位之和**

**标准递归 (非迭代式):**

```python
def sum_digits_recursive(n):
  """Sum the digits of a non-negative integer n."""
  if n < 10:
    return n
  else:
    # 先递归，再做加法
    return sum_digits_recursive(n // 10) + (n % 10)
```

这里，拿到 `sum_digits_recursive(n // 10)` 的结果后，还需要做一次 `+ (n % 10)` 的操作。计算过程是“先深入再回头累加”。

**迭代式递归 (使用 Helper Function):**

```python
def sum_digits_iterative(n):
  """Sum the digits of n using iterative recursion."""
  def helper(n, current_sum):
    if n == 0:
      return current_sum
    else:
      # 递归调用是最后一步
      return helper(n // 10, current_sum + (n % 10))
  return helper(n, 0)
```

在这里，`current_sum` 这个累加器在每次递归调用时都被更新。当到达 Base Case (`n == 0`) 时，最终结果已经计算好了，直接返回即可，不需要再做任何计算。这种方式通常更节省内存（在支持尾调用优化的语言中），因为它不需要在调用栈中保留中间状态。

------



#### **总结与建议**

1. **递归的核心是分解**: 把大问题分解为性质相同的小问题。
2. **三大法宝**:
   - **Base Case**: 停止的条件，最简单的情况。
   - **Recursive Step**: 如何让问题规模变小。
   - **Leap of Faith**: 相信你的递归调用，只思考如何利用子问题的解。
3. **识别递归类型**:
   - **线性递归**: 调用一次自身 (如阶乘)。
   - **树形递归**: 调用多次自身 (如斐波那契)。警惕其效率问题。