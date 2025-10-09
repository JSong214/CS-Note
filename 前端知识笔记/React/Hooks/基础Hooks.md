---
date: 2025-06-27
tags:
  - React
---
### **Hooks的两条黄金规则** 
为了让Hooks能够正常工作，必须遵守两条规则：

1. **只在顶层调用Hooks**：不要在循环、条件或嵌套函数中调用Hooks。要确保Hooks在每次渲染时都以相同的顺序被调用。这是React能够正确地将内部状态与对应的Hook关联起来的基础 。  
    
2. **只在React函数中调用Hooks**：只能在React函数组件或自定义Hooks中调用Hooks，不要在普通的JavaScript函数中调用。

## 基础 Hooks 详解

以下是你在日常开发中最常用、最重要的几个基础 Hooks。

### 1. `useState`：状态管理 Hook

- **一句话总结**：让函数组件拥有自己的 state（状态），当状态改变时，组件会重新渲染。
    
- **详细解释**：
    
    - **语法**：`const [state, setState] = useState(initialState);`
        
        - `useState` 接收一个 `initialState`（初始状态）作为参数。这个参数只在组件的**首次渲染**时生效。
            
        - 它返回一个包含两个元素的数组：
            
            1. `state`：当前的状态值。
                
            2. `setState`：一个用于更新该状态的函数（我们称之为 "setter" 函数）。
                
    - **如何更新状态**：
        
        ```JavaScript
        import React, { useState } from 'react';
        
        function Counter() {
          // 声明一个名为 count 的 state 变量，初始值为 0
          const [count, setCount] = useState(0);
        
          return (
            <div>
              <p>You clicked {count} times</p>
              {/* 调用 setCount 来更新 count 的值 */}
              <button onClick={() => setCount(count + 1)}>
                Click me
              </button>
            </div>
          );
        }
        ```
        
    - **⭐ 重要知识点：函数式更新 (Functional Updates)**
        
        - **问题**：`setState` 的更新是**异步**的。如果你需要基于前一个状态来计算新状态，直接使用 `setCount(count + 1)` 可能会因为闭包陷阱而获取到旧的 `count` 值，尤其是在一次事件中多次调用 `setState` 时。
            
        - **解决方案**：给 `setState` 传递一个函数。这个函数会自动接收前一个状态（`prevState`）作为参数，并返回新的状态。React 会确保 `prevState` 是最新的。
            
        - **示例**：
            
            ```JavaScript
            // 推荐 ✅：安全可靠，尤其在处理快速、连续的更新时
            const increment = () => {
              setCount(prevCount => prevCount + 1);
            };
            
            // 不推荐 ❌：在某些情况下可能不准确
            // const increment = () => {
            //   setCount(count + 1);
            // };
            ```
            
        - **何时必须用函数式更新？** 当你的新状态依赖于旧状态时，强烈推荐使用。
            

### 2. `useEffect`：副作用 Hook

- **一句话总结**：用于处理函数组件中的副作用，如数据请求、DOM 操作、设置订阅、定时器等。
    
- **详细解释**：
    
    - **语法**：`useEffect(callback, [dependencies]);`
        
        - `callback`：一个包含了副作用逻辑的函数。
            
        - `dependencies`（依赖项数组）：一个可选数组。`useEffect` 会在**首次渲染后**执行 `callback`，并且**仅当**这个数组中的任何一个值发生变化时，才会在后续的渲染中**重新执行** `callback`。
            
    - ⭐ 重要知识点：理解依赖项数组
        
        这是 useEffect 最核心、最容易出错的部分。它控制着副作用的执行时机。
        
        1. **不传依赖项数组**：
            
            - `useEffect(() => { ... });`
                
            - **效果**：每次组件渲染后（包括首次渲染），`callback` 都会执行。这通常不是你想要的，因为它可能导致性能问题或死循环（如在副作用中更新了 state）。
                
        2. **传入空数组 `[]`**：
            
            - `useEffect(() => { ... }, []);`
                
            - **效果**：`callback` **仅在组件首次渲染后执行一次**。这完美模拟了 class 组件中的 `componentDidMount`。非常适合用于只执行一次的逻辑，比如获取初始数据、设置全局事件监听。
                
        3. **传入包含值的数组 `[dep1, dep2]`**：
            
            - `useEffect(() => { ... }, [count, userId]);`
                
            - **效果**：`callback` 会在首次渲染后执行，并且在之后的每一次渲染中，如果 `count` 或 `userId` 的值与上一次渲染时相比发生了变化，`callback` 就会再次执行。这模拟了 `componentDidMount` + `componentDidUpdate` 的组合。
                
    - **⭐ 重要知识点：清理副作用 (Cleanup)**
        
        - **问题**：有些副作用需要被清理以防止内存泄漏，比如定时器、WebSocket 连接、事件监听等。
            
        - **解决方案**：在 `useEffect` 的 `callback` 函数中返回一个**清理函数**。这个函数会在组件卸载时（`componentWillUnmount`），以及在下一次 `useEffect` 重新执行之前被调用。
            
        - **示例**：
            
            ```JavaScript
            useEffect(() => {
              // 副作用逻辑：设置一个定时器
              const timerId = setInterval(() => {
                console.log('Tick');
              }, 1000);
            
              // 返回一个清理函数
              return () => {
                console.log('Clearing interval');
                clearInterval(timerId); // 组件卸载或依赖项变化时，清除定时器
              };
            }, []); // 空数组表示只在挂载和卸载时执行
            ```
            

### 3. `useContext`：跨组件共享状态

- **一句话总结**：无需通过 props 层层传递，就能让子组件直接访问到顶层组件的状态。
    
- **详细解释**：
    
    - **解决的问题**：Prop Drilling（属性钻井），即一个深层嵌套的子组件需要某个状态，这个状态必须由其所有父组件一层一层地传递下来，非常繁琐。
        
    - **使用步骤**：
        
        1. **创建 Context**：使用 `React.createContext` 创建一个 Context 对象。
            
            ```JavaScript
            // MyContext.js
            import React from 'react';
            export const ThemeContext = React.createContext('light'); // 'light' 是默认值
            ```
            
        2. **提供 Context**：在父组件中使用 `Context.Provider`，并通过 `value` 属性提供要共享的值。
            
            ```JavaScript
            import { ThemeContext } from './MyContext';
            
            function App() {
              const [theme, setTheme] = useState('dark');
              return (
                <ThemeContext.Provider value={theme}>
                  <Toolbar />
                </Theme-Context.Provider>
              );
            }
            ```
            
        3. **消费 Context**：在任何子组件中，使用 `useContext` Hook 来订阅和获取 `value`。
            
            ```JavaScript
            import React, { useContext } from 'react';
            import { ThemeContext } from './MyContext';
            
            function ThemedButton() {
              const theme = useContext(ThemeContext); // 直接获取到 'dark'
              return <button style={{ 
              background: theme === 'dark' ? '#333' : '#FFF' 
              }}>I am a themed button</button>;
            }
            ```
            

### 4. `useRef`：访问 DOM 或持久化可变值

- **一句话总结**：获取 DOM 元素的引用，或者存储一个在多次渲染之间保持不变的、可变的值（且其变化**不会**触发组件重新渲染）。
    
- **详细解释**：
    
    - **语法**：`const myRef = useRef(initialValue);`
        
        - 返回一个可变的 ref 对象，其 `.current` 属性被初始化为 `initialValue`。
            
    - **两大用途**：
        
        1. **访问 DOM 元素**：这是最常见的用法。
            
            ```JavaScript
            import React, { useRef, useEffect } from 'react';
            
            function TextInputWithFocusButton() {
              const inputEl = useRef(null); // 初始化为 null
            
              useEffect(() => {
                // 当组件挂载后，让输入框自动获得焦点
                inputEl.current.focus();
              }, []);
            
              return <input ref={inputEl} type="text" />;
            }
            ```
            
        2. **存储一个可变值**：
            
            - `useRef` 创建的 `.current` 值在组件的整个生命周期内都保持不变。
                
            - **与 `useState` 的核心区别**：更新 `useRef` 的 `.current` 属性**不会**导致组件重新渲染。这在需要存储一个值，但又不希望这个值的变化影响视图时非常有用（比如存储上一次的 state 值，或者一个定时器的 ID）。
                

---

## 知识点总结表

| Hook             | 主要用途        | 何时使用？                            |
| ---------------- | ----------- | -------------------------------- |
| **`useState`**   | 管理组件内部的基础状态 | 绝大多数需要状态的场景，如计数器、表单输入、开关等。       |
| **`useEffect`**  | 处理副作用       | 数据请求、DOM操作、事件监听、定时器、订阅等。         |
| **`useContext`** | 跨组件共享状态     | 解决“属性钻井”，全局主题、用户登录信息等。           |
| **`useRef`**     | 访问DOM或持久化值  | 需要直接操作DOM（如焦点、媒体播放），或存储与渲染无关的数据。 |
