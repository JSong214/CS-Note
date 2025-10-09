---
date: 2025-06-27
tags:
  - React
---
### 1. 核心技术：条件渲染 (Conditional Rendering)

动态 UI 的常见需求是：根据不同条件，显示或隐藏某些元素。

- **实现方式：**
    
    1. if/else 语句
        
        适用于逻辑复杂，需要在 return 语句外部准备 JSX 的场景。
        
        ```JavaScript
        function UserGreeting({ isLoggedIn }) {
          if (isLoggedIn) {
            return <h1>Welcome back!</h1>;
          } else {
            return <h1>Please sign up.</h1>;
          }
        }
        ```
        
    2. 三元运算符 (? :)
        
        最常用，非常适合在 JSX 中进行简单的“二选一”渲染。
        
        ```JavaScript
        function UserGreeting({ isLoggedIn }) {
          return (
            <div>
              {isLoggedIn ? <h1>Welcome back!</h1> : <h1>Please sign up.</h1>}
            </div>
          );
        }
        ```
        
    3. 逻辑与运算符 (&&)
        
        非常适合“满足条件则渲染，否则什么都不渲染”的场景。
        
        ```JavaScript
        function Mailbox({ unreadMessages }) {
          return (
            <div>
              <h1>Hello!</h1>
              {unreadMessages.length > 0 &&
                <h2>
                  You have {unreadMessages.length} unread messages.
                </h2>
              }
            </div>
          );
        }
        ```
        
        **原理：** 在 JavaScript 中，`true && expression` 总是返回 `expression`，而 `false && expression` 总是返回 `false`。React 不会渲染 `true`, `false`, `null` 和 `undefined`。
        

---

### 2. 核心技术：列表渲染 (Rendering Lists)

动态 UI 经常需要渲染一组数据，比如一个任务列表、一个商品目录等。

- **实现方式：** 使用数组的 `.map()` 方法。
    
    ```JavaScript
    const products = [
      { id: 'p1', title: 'Laptop' },
      { id: 'p2', title: 'Mouse' },
      { id: 'p3', title: 'Keyboard' },
    ];
    
    function ProductList() {
      return (
        <ul>
          {products.map(product => (
            <li key={product.id}>
              {product.title}
            </li>
          ))}
        </ul>
      );
    }
    ```
    
- **⭐ 重要知识点详解：`key` 属性**
    
    `key` 是 React 在渲染列表时用于**识别和追踪**每个元素的唯一标识。
    
    为什么必须要有 key？
    
    React 使用 key 来进行高效的 Diffing 算法（和解过程）。当列表数据更新时（例如增、删、改、排序），React 会通过 key 来判断：
    
    - 哪些元素是**新增**的？
        
    - 哪些元素是**被删除**的？
        
    - 哪些元素只是**位置移动**了？
        
    
    如果没有 `key`，React 只能逐个比较元素，效率低下，且容易在涉及组件 state 的列表中产生 bug。例如，一个带输入框的列表项，在列表排序后，输入框内的内容可能会错位。
    
    **如何选择 `key`？**
    
    - **最佳选择：** 使用数据中**独一无二且稳定**的 ID，比如数据库中的 `id`。
        
    - **次佳选择：** 如果数据没有稳定 ID，可以根据内容生成一个唯一的哈希值。
        
    - **最不推荐（但有时可用）：** 使用数组的索引 (`index`)。
        
        - **为什么不推荐使用 `index`？** 如果列表的顺序会改变（如新增、删除、排序），使用 `index` 作为 `key` 会导致性能问题和潜在的 state bug。因为元素的 `key` 会随着它在数组中的位置而改变，React 会误以为是元素内容发生了变化，而不是位置移动。
            
        - **什么时候可以用 `index`？** 只有当列表是完全静态的，永远不会被重新排序、过滤或增删时，才可以安全地使用 `index`。
            

---

### 3. 响应变化：副作用与 `useEffect` Hook

有时，我们的组件需要在渲染到屏幕之后，执行一些“额外操作”，比如请求数据、设置订阅、手动操作 DOM 等。这些操作被称为**副作用 (Side Effects)**。

- useEffect Hook
    
    它让你在函数组件中执行副作用操作。
    
    ```JavaScript
    import { useState, useEffect } from 'react';
    
    function UserProfile({ userId }) {
      const [user, setUser] = useState(null);
      const [loading, setLoading] = useState(true);
    
      useEffect(() => {
        // 副作用函数
        setLoading(true);
        fetch(`https://api.example.com/users/${userId}`)
          .then(res => res.json())
          .then(data => {
            setUser(data);
            setLoading(false);
          });
    
        // 清理函数 (可选)
        return () => {
          // 在组件卸载或下一次 effect 执行前运行
          console.log('Cleaning up previous effect...');
        };
      }, [userId]); // 依赖项数组
    
      if (loading) {
        return <p>Loading...</p>;
      }
    
      return <h1>{user.name}</h1>;
    }
    ```
    
- **⭐ 重要知识点详解：依赖项数组 (Dependency Array)**
    
    `useEffect` 的第二个参数是其行为的关键，它告诉 React **何时**应该重新执行副作用函数。
    
    1. 不提供依赖项数组：
        
        `useEffect(() => { ... })`
        
        Effect 会在每次组件渲染后执行。这通常不是我们想要的，容易造成性能问题或无限循环。
        
    2. 提供空数组 `[]`：
        
        `useEffect(() => { ... }, [])`
        
        Effect 只会在组件首次挂载 (mount) 后执行一次，类似于类组件的 componentDidMount。非常适合做一些只需要执行一次的初始化操作（如获取初始数据）。
        
    3. 提供包含依赖项的数组 `[prop, state]`：
        
        `useEffect(() => { ... }, [userId]`)
        
        Effect 会在组件首次挂载后执行，并且在依赖项 (userId) 发生变化后的每一次重新渲染时再次执行。这是最常用的模式，可以确保当相关数据变化时，副作用也同步更新。
        

---

### 总结与笔记要点

为了方便你整理，以下是核心要点的浓缩：

1. **核心理念**: 数据驱动视图更新。
    
2. **State (`useState`)**:
    
    - 组件内部的可变数据。
        
    - `const [value, setValue] = useState(initialValue)`。
        
    - **重点**: 必须遵循**不可变性**，更新时要创建新的对象/数组。
        
3. **条件渲染**:
    
    - 用 `if/else`, 三元运算符 (`? :`) 或逻辑与 (`&&`) 来控制 JSX 的输出。
        
    - 三元和 `&&` 是 JSX 内联写法的首选。
        
4. **列表渲染**:
    
    - 使用 `.map()` 将数据数组转换为 JSX 元素数组。
        
    - **重点**: 必须为每个列表项提供一个**稳定且唯一**的 `key` 属性，用于高效的 DOM 更新。最好用数据 ID，避免用 `index`。
        
5. **副作用 (`useEffect`)**:
    
    - 处理数据获取、订阅等组件渲染之外的操作。
        
    - `useEffect(effectFn, dependencyArray)`。
        
    - **重点**: 精确控制**依赖项数组**来决定 effect 的执行时机，避免不必要的执行。
        
        - `[]`: 仅首次渲染后执行。
            
        - `[dep1, dep2]`: 首次渲染及任何依赖项变化后执行。
            
        - 不传: 每次渲染后都执行（慎用）。
            