---
date: 2025-06-26
tags:
  - React
aliases:
  - JSX
---
## JSX语法

### 1. JSX 的核心定义

- **本质**: JavaScript 的一种**语法扩展** (Syntax Extension)。
- **核心功能**: 允许开发者在 JavaScript 文件中，直接使用类似 HTML 的语法来编写和描述用户界面 (UI)。

### 2. JSX 的工作原理：转换过程

这是 JSX 中**最重要**的知识点，深刻理解这一过程是掌握 React 的关键。

- **浏览器不识别 JSX**: 浏览器本身只能理解标准的 JavaScript、HTML 和 CSS。它无法直接运行含有 JSX 语法的代码。

- **编译 (Compilation)**: JSX 代码在被浏览器执行前，必须先通过一个名为**编译器 (Compiler)** 的工具进行转换。

  - **常用工具**: **Babel** 是最主流的 JSX 编译器。
  - **转换目标**: Babel 会将 JSX 语法转换为纯粹的、浏览器可以理解的 JavaScript 函数调用。

- **核心转换规则 (The "Syntactic Sugar")**:

  - JSX 本质上是 **`React.createElement()` 函数**的**语法糖 (Syntactic Sugar)**。语法糖意味着它提供了一种更甜美、更易读的写法，但其底层实现是更复杂的函数调用。

  - 转换示例:

    - **JSX 写法**:

      ```js
      <h1 className="header">Hello</h1>
      ```

    - **编译后的 JavaScript**:

      ```js
      React.createElement('h1', {className: 'header'}, 'Hello')
      ```

  - **`React.createElement()` 的作用**: 这个函数会创建一个 JavaScript **对象**（通常被称为 "React Element"）。这个对象是 UI 元素的轻量级描述。React 后续会使用这些对象来构建和更新**虚拟 DOM (Virtual DOM)**，并最终高效地更新真实的浏览器 DOM。



### 3. JSX 的核心威力：嵌入 JavaScript 表达式

这是 JSX 最强大、最灵活的特性，是实现动态渲染的基础。

- **语法**: 使用**花括号 `{}`**。

- **功能**: 你可以在 JSX 的标签内部，通过 `{}` 嵌入任何**有效**的 JavaScript **表达式** (JavaScript Expression)。

- **“表达式”的含义**: 任何可以计算出一个值的代码片段都是表达式。例如：

  - **变量**: `<h1>Hello, {name}</h1>` - 这里会显示 `name` 变量的值。

  - **函数调用**: `<p>Today is {new Date().toLocaleDateString()}</p>` - 直接调用函数并显示其返回值。

  - **数学运算**: `<div>{2 + 2}</div>` - 会渲染出 `<div>4</div>`。

  - **三元运算符**: `{isLoggedIn ? <Profile /> : <LoginButton />}` - 根据条件动态渲染不同的组件。

  - **数组方法 (如 .map)**:

    ```js
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
    ```
