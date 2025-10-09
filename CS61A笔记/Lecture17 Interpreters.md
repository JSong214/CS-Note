## Lecture17 Interpreters



#### 1. 宏大视角：什么是解释器？ (The Big Picture)

解释器本质上是一个**程序**，它的**输入是另一种语言写成的代码（字符串或数据结构），输出是那段代码执行后的结果**。

在CS61A中，你将构建一个“元循环解释器”（Metacircular Evaluator）。这个名字听起来很酷，它的意思是：

> **我们用一种语言（比如 Python）去实现另一种语言（Scheme 的一个子集）的解释器。**

这就像一本用中文写的“如何理解英文语法”的书。这个过程揭示了编程语言工作的底层魔法，让你明白你写的代码是如何被计算机一步步理解和执行的。

**核心思想**：**代码即数据 (Code as Data)**。Scheme 语言的 `(+ 1 2)` 既可以被看作是一段需要被执行的代码，也可以被看作是一个包含三个元素（符号`+`, 数字`1`, 数字`2`）的列表数据。解释器就是在操作这些数据结构。

------



#### 2. 核心引擎：Eval-Apply 循环 (The Heart: Eval-Apply Cycle)

这是整个解释器的心脏。它由两个相互递归的函数组成：`eval` 和 `apply`。

- `eval` (求值): 它的任务是**对一个表达式（expression）在一个特定的环境（environment）下进行求值**。
- `apply` (应用): 它的任务是**将一个过程（procedure）应用到一组实际参数（arguments）上**。

这两个函数相互调用，形成一个循环，驱动整个解释过程。

**简化的流程图：**

```
                  +-----------------+
                  | eval(exp, env)  |
                  +-----------------+
                         |
  (如果是组合/call expression, e.g., (+ 1 2))
                         |
                         v
1. eval a operator to get a procedure / 求运算符以获取过程
2. eval all operands to get arguments / 求所有操作数以获取参数
                         |
                         v
                  +----------------------+
                  | apply(proc, args)    |
                  +----------------------+
                         |
        (如果是我们自己定义的函数/compound procedure)
                         |
                         v
1. Create a new environment (frame) / 创建一个新环境（框架）
2. Bind formal parameters to arguments / 将形式形参绑定到实参
3. eval the function's body in this NEW environment / 在这个NEW环境中求函数体的值
                         |
                         |  (又回到了 eval)
                         +------------------> ...
```

------



#### 3. `eval` 函数的内部逻辑 (Inside `eval`)

`eval` 函数是总指挥，它拿到一个表达式后，会判断表达式的类型，然后采取不同的策略。



##### **3.1 基本表达式 (Base Cases / Primitive Expressions)**

这是递归的终点，`eval` 可以直接给出结果。

- **Self-evaluating:** 数字、布尔值等。对它们求值的结果就是它们自身。
  - `eval(5, env)`  =>  `5`
- **Symbols (变量名):** 需要在**环境（environment）**中查找这个符号绑定的值。
  - `eval('x', env)`  =>  在 `env` 中查找 `x` 的值。



##### **3.2 特殊形式 (Special Forms)**

这是最关键的部分！特殊形式**不遵循**“先对所有子表达式求值”的常规规则。`eval` 必须能识别它们并特殊处理。

- `('if', <predicate>, <consequent>, <alternative>)`
  - **为什么特殊?** `if` 不会立即对 `<consequent>` 和 `<alternative>` 都求值。它会先对 `<predicate>` 求值，根据结果是真还是假，**只选择其中一个分支**去 `eval`。这是实现分支逻辑的基础。
  - **处理流程:**
    1. `eval(<predicate>, env)`
    2. 如果结果为真, `eval(<consequent>, env)` 并返回其结果。
    3. 否则, `eval(<alternative>, env)` 并返回其结果。
- `('define', <name>, <value_expr>)`
  - **为什么特殊?** `define` 的目的不是返回一个值，而是**修改当前环境**，在当前环境中创建一个新的绑定（把 `<name>` 和 `<value_expr>` 求值后的结果绑定起来）。
  - **处理流程:**
    1. `eval(<value_expr>, env)` 得到结果 `value`。
    2. 在当前环境的第一帧（current frame）中，将 `name` 和 `value` 绑定。
    3. 返回一个确认符号（例如 `ok`）。
- `('lambda', <parameters>, <body>)`
  - **为什么特殊?** `lambda` 也不会立即执行 `<body>`。它的作用是**创建一个过程（procedure）对象**。这个对象是一个数据结构，它打包了三样东西：
    1. 函数的**形式参数 (parameters)**
    2. 函数的**代码体 (body)**
    3. **函数被定义时所在的环境的指针** (这一点至关重要，是实现词法作用域/闭包的关键！)
  - `eval` 看到 `lambda`，就打包这三样东西，创建一个 procedure 对象并返回。
- `('quote', <expression>)`
  - **为什么特殊?** `quote` 是最简单的特殊形式，它**阻止** `eval` 对其内部的 `<expression>` 进行求值，直接将其原样返回。
  - `eval(('quote', '(+ 1 2)'), env)` => `(+ 1 2)` (返回的是一个列表，而不是数字3)



##### **3.3 组合/调用表达式 (Combinations / Call Expressions)**

如果一个表达式不是基本表达式，也不是特殊形式，那么它就是一个常规的函数调用，例如 `(+ 1 (* 2 3))`。

**处理流程 (这正是 Eval-Apply 循环的核心体现):**

1. **递归调用 `eval`** 来处理操作符（operator），得到一个**过程（procedure）**。
   - `eval('+', env)` => 得到内置的加法过程。
2. **递归调用 `eval`** 来处理所有操作数（operands），得到一个**实际参数（arguments）列表**。
   - `eval(1, env)` => `1`
   - `eval((' * 2 3'), env)` => `6` (这里会再次进入 eval-apply 循环)
3. 将第1步得到的过程和第2步得到的参数列表，传递给 `apply` 函数。
   - `apply(加法过程, [1, 6])` => `7`

------



#### 4. `apply` 函数的内部逻辑 (Inside `apply`)

`apply` 的工作是执行一个函数。它也需要区分两种情况：



##### **4.1 应用内置过程 (Primitive Procedures)**

这些是解释器预先用 Python 实现好的函数（如 `+`, `-`, `*`, `/`, `print` 等）。`apply` 直接调用相应的 Python 函数，并返回结果。



##### **4.2 应用复合过程 (Compound Procedures)**

这些是用 `lambda` 创建的、我们自己定义的函数。`apply` 必须模拟函数调用的过程：

1. **创建新环境 (Create a new environment/frame):**
   - 每个函数调用都会创建一个属于它自己的**新帧 (frame)**。
   - 这个新帧的**父指针 (parent pointer)** 指向该函数**被定义时**所在的环境。（这就是 `lambda` 创建 procedure 对象时保存的环境指针的作用！）
2. **绑定参数 (Bind parameters to arguments):**
   - 在新帧中，将函数的形式参数（parameters）与 `apply` 接收到的实际参数（arguments）一一对应地绑定起来。
3. **求值函数体 (Evaluate the body):**
   - 在刚刚创建的**这个新环境中**，**调用 `eval`** 来执行函数的代码体（body）。

------



#### 5. 数据结构：环境与帧 (Data Structures: Environments and Frames)

环境是理解变量作用域（scoping）的关键。

- **帧 (Frame):** 可以看作是一个**哈希表或字典**，存储着当前作用域的**变量名到值的绑定 (bindings)**。
- **环境 (Environment):** 是一个由**帧组成的链表**。它代表了一个完整的、可供查找的作用域链。

**变量查找 (Lookup) 规则:** 当 `eval` 需要查找一个变量的值时（例如 `eval('x', env)`），它会：

1. 首先在当前环境的**第一帧**（current frame）里查找。
2. 如果找到了，就返回对应的值。
3. 如果没找到，就通过**父指针**移动到**父环境/父帧**中继续查找。
4. 这个过程一直持续，直到找到变量，或者到达**全局帧 (Global Frame)**。如果全局帧里也没有，就抛出一个 "name not found" 的错误。

这个查找规则完美地实现了**词法作用域 (Lexical Scoping)**：一个函数内部的变量查找，取决于这个函数在哪里被*定义*，而不是在哪里被*调用*。

------



#### 示例演练：`(define (square x) (* x x))` 和 `(square 3)`

1. `eval(('define', ('square', 'x'), ...), global_env)`
   - 这是一个 `define` 特殊形式。
   - 它会创建一个 `lambda` 过程对象，该对象包含：
     - 参数: `('x')`
     - 函数体: `(' * ', 'x', 'x')`
     - 环境指针: 指向 `global_env`
   - 然后在 `global_env` 中添加绑定：`'square'` -> `(这个 procedure 对象)`
2. `eval(('square', 3), global_env)`
   - 这是一个组合。
   - **Eval Operator:** `eval('square', global_env)` -> 找到了上面定义的 procedure 对象。
   - **Eval Operand:** `eval(3, global_env)` -> `3`。
   - **进入 Apply:** `apply(procedure 对象, [3])`
3. 在 `apply` 内部 (应用复合过程):
   - **创建新帧 (f1):** 这个新帧的父指针指向 `global_env`（因为 `square` 是在 `global_env` 中定义的）。
   - **绑定参数:** 在 `f1` 中，创建绑定 `'x'` -> `3`。
   - **在新环境中求值函数体:** `eval((' * ', 'x', 'x'), env_with_f1)`
     - 这是一个新的 `eval` 调用，环境是 `f1 -> global_env`。
     - 它是一个组合，于是又走组合的流程：
       - `eval(' * ', env_with_f1)` -> 在 `global_env` 中找到内置的乘法过程。
       - `eval('x', env_with_f1)` -> 在 `f1` 中找到 `x` 的值是 `3`。
       - `eval('x', env_with_f1)` -> 再次在 `f1` 中找到 `x` 的值是 `3`。
       - **进入 Apply:** `apply(内置乘法过程, [3, 3])`
4. 在 `apply` 内部 (应用内置过程):
   - 直接执行 Python 的乘法，`3 * 3`，返回 `9`。
   - 这个结果 `9` 逐层返回，成为整个 `(square 3)` 表达式的最终结果。