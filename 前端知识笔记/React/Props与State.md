---
date: 2025-06-26
tags:
  - React
---
## Props与State

### 1. Props (属性)

#### **核心定义**

可以把**`props`**想象成是**传递给React组件的函数参数**。父组件通过`props`将数据和行为（函数）传递给它的子组件，子组件接收这些`props`并据此渲染UI。

**示例代码：**

```js
// 父组件 ParentComponent
function ParentComponent() {
  return (
    // 向子组件 Welcome 传递名为 name 的 prop
    <Welcome name="张三" />
  );
}

// 子组件 Welcome
// 通过参数接收 props 对象
function Welcome(props) {
  // 使用 props.name 来访问数据
  return <h1>你好, {props.name}</h1>;
}
```

在上面的例子中，`ParentComponent`将一个值为`"张三"`的`name`属性传递给了`Welcome`组件。`Welcome`组件通过其函数参数`props`接收到一个对象`{ name: '张三' }`，并使用它来渲染内容。



#### 【重要】Props是只读的 (Immutability)

这是`Props`最核心、最重要的特性。**子组件永远不能、也不应该直接修改它接收到的`props`。**

- 为什么是只读的？

  - **单向数据流 (Unidirectional Data Flow):** React强制实行自上而下的单向数据流。数据只能从父组件流向子组件。
  - **可预测性与稳定性:** 保证了父组件的状态不会被子组件意外修改，使得组件的行为更加可预测。

- 错误的做法：

  ```js
  function Welcome(props) {
    // ❌ 错误！绝对禁止这样做！
    props.name = "李四"; 
    return <h1>你好, {props.name}</h1>;
  }
  ```

  

#### 其他特性

- **`props.children`**: 这是一个特殊的`prop`，它包含了组件开闭标签之间的所有内容。这使得组件可以像HTML原生标签一样嵌套内容。

  ```js
  function Card(props) {
    return (
      <div className="card">
        {props.children}
      </div>
    );
  }
  
  // 使用
  <Card>
    <h2>我是一个标题</h2>
    <p>我是一些嵌套在Card组件里的内容。</p>
  </Card>
  ```

  

- **默认Props (Default Props)**: 你可以为`props`定义默认值，以防止在父组件未传递该`prop`时出现`undefined`错误。

  ```js
  function Welcome({ name = "访客" }) { // 使用解构和默认参数
    return <h1>你好, {name}</h1>;
  }
  
  // 当不传递 name 时，会使用默认值 "访客"
  <Welcome /> 
  ```

  

### 2. State (状态)

如果说`props`是来自外部的“配置”，那么`state`就是组件内部自己管理和维护的数据。当`state`发生变化时，React会自动重新渲染组件以反映这些变化。



#### 核心定义

**`State`是组件私有的、被完全控制的数据**。它代表了组件在某个时间点的“状态”。`state`的存在使得组件能够“记住”信息并在用户交互中动态更新自己。



#### 【重要】如何使用State (`useState` Hook)

在函数组件中，我们使用`useState` Hook来创建和管理`state`。

- `useState`的调用：

  ```js
  import React, { useState } from 'react';
  
  const [stateVariable, setStateFunction] = useState(initialValue);
  ```

  - `useState`返回一个包含两个元素的数组：
    1. **`stateVariable`: 当前的状态值。**
    2. **`setStateFunction`: 用于更新这个状态值的函数。**
  - **`initialValue`: 状态的初始值**，只在组件的第一次渲染时使用。

- **示例：一个计数器**

  ```js
  import React, { useState } from 'react';
  
  function Counter() {
    // 1. 初始化 state，初始值为 0
    const [count, setCount] = useState(0);
  
    // 2. 定义一个事件处理函数来更新 state
    const handleIncrement = () => {
      // 使用 setCount 函数来更新 count 的值
      setCount(count + 1);
    };
  
    return (
      <div>
        <p>你点击了 {count} 次</p>
        <button onClick={handleIncrement}>
          点击我
        </button>
      </div>
    );
  }
  ```





#### 【重要】State的更新是异步的

这是一个非常容易出错的知识点。**调用`setState`函数并不会立即改变`state`的值，React会“计划”一次更新。**

- 为什么是异步的？

  - **性能优化:** React会将短时间内的多个`setState`调用合并（Batching，批处理）成一次更新，以避免不必要的重复渲染，从而提高性能。

- **带来的问题与解决方案：** 如果你在调用`setState`后立即尝试访问`state`，你会得到旧的值。

  ```js
  const handleIncrement = () => {
    setCount(count + 1);
    // ❌ 这里打印出的 count 仍然是旧的值！
    console.log(count); 
  };
  ```

  - **解决方案1: 函数式更新。** 如果你的新`state`依赖于旧的`state`，最佳实践是向`setState`传递一个函数。这个函数会接收前一个`state`作为参数，并返回新的`state`。

    ```js
    // ✅ 推荐做法：当新状态依赖旧状态时
    setCount(prevCount => prevCount + 1);
    ```

  - **解决方案2: `useEffect` Hook。** 如果你需要在`state`更新后执行某些操作（如API调用），应该使用`useEffect` Hook来监听`state`的变化。

    ```js
    useEffect(() => {
      // 这个函数会在 count 更新并重新渲染后执行
      document.title = `你点击了 ${count} 次`;
    }, [count]); // 依赖数组中放入 count
    ```

    



#### 其他特性

- **State是局部的和封装的:** 一个组件的`state`对其他组件是不可见的，除非它通过`props`将`state`或操作`state`的函数传递下去。





### 3.状态提升 (Lifting State Up)

这是一个非常重要的模式，它完美地结合了`props`和`state`。

**场景:** 当多个子组件需要共享或响应同一个变化的数据时，我们应该将这个共享的`state`移动到它们最近的公共父组件中。

**工作流程:**

1. 在父组件中定义`state`和修改`state`的函数。
2. 父组件通过`props`将`state`的值传递给需要读取该数据的子组件。
3. 父组件通过`props`将修改`state`的函数传递给需要触发数据变化的子组件。

这样，数据源是唯一的（在父组件中），数据流是清晰的（自上而下），子组件通过调用从`props`接收的函数来“请求”父组件更新状态，从而实现兄弟组件间的通信。



**示例：**

```js
// 父组件：管理共享的状态
function SharedParent() {
  const [inputValue, setInputValue] = useState('');

  return (
    <div>
      {/* 将 state 和 setState 函数都通过 props 传递下去 */}
      <ChildA value={inputValue} onValueChange={setInputValue} />
      <ChildB displayValue={inputValue} />
    </div>
  );
}

// 子组件A：可以修改状态
function ChildA({ value, onValueChange }) {
  return (
    <input 
      value={value} 
      onChange={e => onValueChange(e.target.value)} 
    />
  );
}

// 子组件B：只显示状态
function ChildB({ displayValue }) {
  return <p>显示的值是: {displayValue}</p>;
}
```



### 总结

- **Props:** 像管道，负责将数据从高处（父）输送到低处（子）。水流方向是固定的。
- **State:** 像组件内部的水库，存储着自己的水。可以通过特定的阀门（`setState`）来改变水位，水位变化会影响组件自身的样子。
- **状态提升:** 当几个组件需要共享同一个水源时，就把水库建在它们共同的上游（父组件），再通过管道（`props`）把水和控制阀门的方法（`setState`函数）送下去。