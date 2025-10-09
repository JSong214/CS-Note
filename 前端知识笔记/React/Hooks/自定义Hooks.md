---
date: 2025-06-27
tags:
  - React
---
### React 自定义 Hooks 知识点清单

#### 一、 什么是自定义 Hook？ (What)

**核心定义**：自定义 Hook 是一个 JavaScript 函数，其名称以 `use` 开头，函数内部可以调用其他的 React Hooks (如 `useState`, `useEffect`, `useContext` 等)。

它是一种让你在不同的函数组件之间**复用状态逻辑 (stateful logic)** 的机制。


```JavaScript
// 一个简单的自定义 Hook 示例
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  }, [friendID]); // 依赖项

  return isOnline;
}
```

---

#### 二、 为什么要使用自定义 Hook？ (Why)

在自定义 Hook 出现之前，我们通常使用**高阶组件 (Higher-Order Components, HOCs)** 和 **Render Props** 模式来复用逻辑。自定义 Hook 提供了更简洁、更直观的替代方案。

**主要优点：**

1. **逻辑复用 (Logic Reusability)**
    
    - **痛点**：多个组件可能需要执行相同的逻辑，比如：从 API 获取数据、订阅事件、操作 `localStorage` 等。如果每个组件都写一遍，代码会非常冗余且难以维护。
        
    - **解决方案**：将这部分可复用的“状态逻辑”提取到自定义 Hook 中。任何组件都可以通过一行代码来使用这份逻辑。
        
2. **代码整洁与关注点分离 (Clean Code & Separation of Concerns)**
    
    - **痛点**：组件的 `useEffect` 中常常混合了各种不相关的逻辑（比如，一个 `useEffect` 获取数据，另一个设置页面标题，还有一个监听窗口大小），导致组件变得臃肿复杂。
        
    - **解决方案**：将特定的逻辑封装到对应的自定义 Hook 中。例如，创建一个 `useFetch` 来处理数据获取，创建一个 `usePageTitle` 来处理页面标题。这使得组件本身的代码更专注于 UI 渲染，逻辑清晰。
        
3. **简化测试 (Simplified Testing)**
    
    - 由于自定义 Hook 是一个独立的函数，你可以脱离组件对其进行单独的单元测试，验证其内部逻辑是否正确，而无需渲染整个 UI。
        

---

#### 三、 【重要】如何创建和使用自定义 Hook？ (How)

这是自定义 Hook 的核心，我们分步详解。

##### 1. 创建规则

- **必须以 `use` 开头**：这是强制性约定。例如 `useFetch`、`useLocalStorage`。
    
    - **详细解释**：这个命名规则不仅仅是个建议。React 的 Linter 插件 (`eslint-plugin-react-hooks`) 依赖这个命名约定来静态分析代码，以判断一个函数是否是 Hook。只有这样，它才能帮你检查是否违反了 **Hooks 的两条规则**（只能在顶层调用 Hook，只能在 React 函数组件或自定义 Hook 中调用 Hook）。如果你的函数不以 `use` 开头，Linter 就无法知道它是一个 Hook，从而无法进行规则检查，可能导致难以察觉的 Bug。
        
- **内部可以调用其他 Hooks**：这是自定义 Hook 的“超能力”所在。你可以在其中自由组合 `useState`, `useEffect`, `useCallback` 等，来构建你需要的状态逻辑。
    

##### 2. 【重要】核心思想：共享的是逻辑，而非 State

这是初学者最容易混淆的一点，必须深刻理解。

- **详细解释**：
    
    - 当你调用一个自定义 Hook 时，它内部的 `useState` 和 `useEffect` 等 **状态和副作用是完全独立的**。
        
    - 把自定义 Hook 想象成一个“逻辑蓝图”或“工厂函数”。每次调用它，它都会为你“生产”出一套全新的、独立的 state 和 effects。
        
    - **示例**：假设我们有一个计数器 Hook `useCounter`。
    
    ```JavaScript
    import { useState } afrom 'react';
    
    function useCounter(initialValue = 0) {
      const [count, setCount] = useState(initialValue);
      const increment = () => setCount(count + 1);
      const decrement = () => setCount(count - 1);
      return { count, increment, decrement };
    }
    
    // --- 在组件中使用 ---
    
    function ComponentA() {
      const { count, increment } = useCounter(0); // 这里的 count state 属于 ComponentA
      return <button onClick={increment}>Component A Count: {count}</button>;
    }
    
    function ComponentB() {
      const { count, increment } = useCounter(10); // 这里的 count state 属于 ComponentB
      return <button onClick={increment}>Component B Count: {count}</button>;
    }
    ```
    
    - 在上面的例子中，`ComponentA` 和 `ComponentB` 都使用了 `useCounter`。但是，`ComponentA` 的 `count` 状态从 0 开始，`ComponentB` 的 `count` 状态从 10 开始。你点击 `ComponentA` 的按钮，只会影响 `ComponentA` 的 `count`，与 `ComponentB` **毫无关系**。
        
    - **结论**：自定义 Hook 帮助你复用的是**创建和管理 state 的那段逻辑代码**，而不是 state 本身。每个使用该 Hook 的组件都拥有自己的一份 state。
        

##### 3. 参数传递与灵活性

为了让自定义 Hook 更通用，我们通常会给它设计参数。

- **详细解释**：
    
    - 通过参数，我们可以配置 Hook 的初始状态或行为。
        
    - 在上面的 `useCounter(initialValue)` 例子中，`initialValue` 就是一个参数，允许我们为不同组件设置不同的初始计数值。
        
    - 在 `useFetch(url)` 例子中，`url` 参数让我们能用同一个 Hook 去获取不同 API 端点的数据。
        

##### 4. 【重要】返回值的设计

自定义 Hook 的返回值决定了组件如何使用它。常见的设计有两种：返回**数组**或返回**对象**。

- **返回数组 (Array)**
    
    - **示例**：React 内置的 `useState` 就是最好的例子 `const [state, setState] = useState()`。
        
    - **优点**：
        
        - **命名自由**：调用者可以使用数组解构任意命名返回的值，非常灵活。比如 `const [count, setCount] = useState(0)` 和 `const [name, setName] = useState('')`。
            
        - **简洁**：当返回值不多时（通常是 2-3 个），语法很紧凑。
            
    - **缺点**：
        
        - **顺序敏感**：必须按固定的顺序解构。
            
        - **可读性差**：当返回值增多时，`result[0]`, `result[1]` 这样的形式很难理解其含义。
            
- **返回对象 (Object)**
    
    - **示例**：在上面 `useCounter` 和 `useFetch` 的例子中，我们返回了对象。
        
        ```JavaScript
        const { count, increment, decrement } = useCounter();
        const { data, loading, error } = useFetch(url);
        ```
        
    - **优点**：
        
        - **高可读性**：返回值的 key (`data`, `loading`) 是自解释的，非常清晰。
            
        - **顺序无关**：解构时不需要关心顺序。
            
        - **易于扩展**：未来给 Hook 增加新的返回值时，不会破坏已有的组件（因为旧组件没有解构新的 key）。
            
    - **缺点**：
        
        - **命名固定**：调用者必须使用 Hook 内部定义的 key，除非使用别名 `const { data: productsData } = useFetch(...)`。
            
- **最佳实践建议**：
    
    - 当你的 Hook 模拟 `useState` 的行为，返回一个“值”和它的“修改器”时，**优先使用数组**。
        
    - 当你的 Hook 返回多个独立的状态值时，**优先使用对象**，以增强代码的可读性和可维护性。
        

---

#### 四、 实战演练：创建一个 `useFetch`

让我们从头构建一个实用的 `useFetch` Hook，这是最常见的自定义 Hook 之一。

**第一步：识别重复逻辑**

假设没有自定义 Hook，获取数据的组件可能长这样：

```JavaScript
function ProductList() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch('https://api.example.com/products')
      .then(res => res.json())
      .then(data => setData(data))
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, []); // 依赖项为空，只在挂载时执行一次

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data && data.map(product => <li key={product.id}>{product.name}</li>)}
    </ul>
  );
}
```

可以看到，`useState` 的定义和 `useEffect` 里的整个获取流程，在每个需要获取数据的组件里都会重复。

**第二步：抽取到自定义 Hook**

我们把这部分逻辑抽取出来，并用 `url` 作为参数。

```JavaScript
import { useState, useEffect } from 'react';

function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // 使用 AbortController 来处理组件卸载时取消请求的情况
    const controller = new AbortController();
    
    setLoading(true);
    setData(null); // 每次 url 变化时重置数据
    setError(null);

    fetch(url, { signal: controller.signal })
      .then(res => {
        if (!res.ok) {
          throw new Error(`HTTP error! status: ${res.status}`);
        }
        return res.json();
      })
      .then(data => setData(data))
      .catch(err => {
        if (err.name !== 'AbortError') {
          setError(err);
        }
      })
      .finally(() => setLoading(false));

    // 清理函数
    return () => {
      controller.abort();
    };
  }, [url]); // 依赖项是 url，当 url 变化时，重新获取数据

  // 以对象形式返回所有状态
  return { data, loading, error };
}
```

_注意：这个版本更健壮，处理了请求取消和错误情况。_

**第三步：在组件中使用 Hook**

现在，`ProductList` 组件变得极其简洁：

```JavaScript
import useFetch from './useFetch'; // 引入自定义 Hook

function ProductList() {
  const { data: products, loading, error } = useFetch('https://api.example.com/products');
  // 使用了解构别名，将 data 重命名为 products，增加可读性

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {products && products.map(product => <li key={product.id}>{product.name}</li>)}
    </ul>
  );
}
```

---

#### 五、 总结与最佳实践

1. **命名**：永远用 `use` 开头。
    
2. **单一职责**：尽量让一个自定义 Hook只做一件事。例如，`useFetch` 就只管获取数据，`useLocalStorage` 就只管读写本地存储。不要创建一个包罗万象的 `useEverything` Hook。
    
3. **依赖项数组**：在自定义 Hook 内部使用 `useEffect`, `useCallback`, `useMemo` 时，一定要正确处理它们的依赖项数组，以避免 stale closures (闭包陷阱) 或无限循环。
    
4. **通用性**：设计时考虑通用性，通过参数让 Hook 变得更灵活。
    
5. **返回值的选择**：根据场景，明智地选择返回数组还是对象。