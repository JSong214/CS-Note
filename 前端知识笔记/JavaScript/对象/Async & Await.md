---
date: 2025-06-13
tags:
  - JavaScript
aliases:
  - Async
---
### 基础概念

- **`async` 关键字**
  - 用于声明异步函数：`async function foo() {}`
  - **返回值始终是 [[Promise]] 对象**

- **`await` 关键字**
  - 只能在 `async` 函数内部使用
  - **作用**：暂停异步函数执行，等待 Promise 状态变更，并获取它(`Promise` )兑现之后的值。


### 执行流程

```js
async function demo() {
  console.log("1");
  const result = await fetchData(); // 暂停执行，等待 Promise
  console.log("2", result);
}
```

- **输出顺序**：`1` → (等待) → `2`
- **关键特性**：
  - 遇到 `await` 时，函数暂停并跳出，主线程继续执行其他同步代码
  - Promise 完成后，函数从 `await` 处恢复执行


### 错误处理

- **`try/catch` 捕获异常**
  ```js
  async function loadData() {
    try {
      const data = await fetch("https://api.example.com");
    } catch (error) {
      console.error("请求失败:", error); // 捕获 Promise reject 或同步错误
    }
  }
  ```

- **链式调用 `.catch()`**
  ```js
  loadData()
    .then(result => { /*...*/ })
    .catch(error => console.error(error)); // 捕获整个 async 函数错误
  ```


### 并行优化策略

- **顺序执行（低效）**
  ```js
  const a = await task1(); // 等待完成
  const b = await task2(); // 再开始执行
  ```

- **并行执行（高效）**
  ```js
  // Promise.all
  const [a, b] = await Promise.all([task1(), task2()]);
  ```


### 常见误区

- **顶层 `await`**
  - 仅在 ES 模块中支持（`<script type="module">`）：
    ```js
    // 模块顶层直接使用
    const data = await fetchData();
    ```

- **`await` 不阻塞主线程**
  - 只暂停当前 `async` 函数的执行，不影响其他同步代码

- **避免过度顺序化**
  - 无依赖关系的异步操作应用 `Promise.all` 并行


### 与 [[Promise]] 的关系

| 特性     | Promise               | Async/Await           |
| :------- | :-------------------- | :-------------------- |
| 代码结构 | 链式调用（`.then()`） | 类同步写法            |
| 错误处理 | `.catch()`            | `try/catch`           |
| 可读性   | 回调嵌套较深          | 线性逻辑清晰          |
| 本质     | 基础异步方案          | 基于 Promise 的语法糖 |
