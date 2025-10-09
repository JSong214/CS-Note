---
date: 2025-06-13
tags:
  - JavaScript
---
> 💡 **最佳实践**：始终用 `.catch()` 处理错误，优先使用 `async/await` 提升可读性，并行操作时使用 `Promise.all()`。


#### **Promise 基础**

- **定义**：Promise 是异步编程的解决方案，表示一个异步操作的**最终完成（或失败）及其结果值**。

- **三种状态**
  - **Pending（待定）**：初始状态
  - **Fulfilled（已兑现）**：操作成功完成（调用 `resolve`）
  - **Rejected（已拒绝）**：操作失败（调用 `reject`）
  - 注意：***状态一旦改变（Fulfilled/Rejected）就不可逆。***

- **创建 Promise**
  ```js
  const promise = new Promise((resolve, reject) => {
    // 异步操作（如 API 请求、定时器等）
    if (/* 成功条件 */) {
      resolve(value); // 状态变为 Fulfilled
    } else {
      reject(error);  // 状态变为 Rejected
    }
  });
  ```


#### **Promise 方法**

- **`.then()`**
  注册 Fulfilled 和 Rejected 状态的回调函数。
  ```js
  promise.then(
    value => { /* Fulfilled 处理逻辑 */ },
  );
  ```

- **`.catch()`**
  捕获链式调用中的错误（等价于 `.then(null, errorHandler)`）。
  ```js
  promise
    .then(value => { ... })
    .catch(error => console.error("出错:", error));
  ```

- **`.finally()`**
  **无论成功/失败都会执行**（适合清理操作）。
  ```js
  promise
    .then(...)
    .catch(...)
    .finally(() => console.log("finally执行"));
  ```


#### **Promise 链式调用**

- **链式规则**
  - 每次 `.then()` 返回一个**新的 Promise**
  - 返回值类型决定新 Promise 状态：
    - 返回普通值 → 新 Promise 立即 Fulfilled
    - 返回 Promise → 继承该 Promise 状态
    - 抛出错误 → 新 Promise 变为 Rejected

- 示例
  ```js
  fetch("/api/data")
    .then(response => response.json()) // 返回新 Promise
    .then(data => processData(data))   // 处理数据
    .catch(error => handleError(error));
  ```  


#### **静态方法**

- `Promise.resolve(value)`
  - 创建立即 Fulfilled 的 Promise
    ```js
    Promise.resolve(42).then(v => console.log(v))
    ```

- `Promise.reject(error)`
  -  创建立即 Rejected 的 Promise
    ```js
    Promise.reject(new Error("失败"))
    ```

- `Promise.all(iterable)`
  - 所有 Promise**并行执行**
  - 所有 Promise 成功时**返回结果数组**，**任一失败立即拒绝，抛出错误**
    ```js
    // 模拟异步操作
    const task1 = () => new Promise(resolve => setTimeout(() => resolve('A'), 200));
    const task2 = () => new Promise(resolve => setTimeout(() => resolve('B'), 100));
    const task3 = 42; // 非 Promise 值
    
    Promise.all([task1(), task2(), task3])	// 同时发起三个请求
      .then(results => {
        console.log(results); // ['A', 'B', 42] （按输入顺序）
      })
      .catch(error => {
        console.error("某个任务失败:", error);
      });
    ```

  - 注意事项：
    - **适用场景**：多个**无依赖**的异步任务并行执行（如同时请求多个 API）。
    - **失败快速响应**：任一失败会立即终止，适合需要“全部成功”的场景。
    - **不适用场景**：需按顺序执行异步操作时（用 `async/await` 或链式调用更合适）。

- `Promise.race(iterable)`
  - 返回第一个完成（成功/失败）的 Promise 结果
    ```js
    Promise.race([promise1, promise2, promise3])
      .then(result => {
        // 处理第一个成功的 Promise 的结果
      })
      .catch(error => {
        // 处理第一个失败的 Promise 的错误
      });
    ```

  - 注意事项：
    - **忽略后续结果**：一旦有一个 Promise 完成（无论成功或失败），其他 Promise 的结果将被忽略
    - **不能取消任务**：即使某个 Promise 赢得了竞速，其他异步操作仍在后台继续执行
    - **错误处理**：第一个完成的是 rejected Promise 会触发 catch


#### **错误处理**

**推荐方式**
在链式末尾使用 `.catch()` 统一捕获错误：
```js
getData()
  .then(step1)
  .then(step2)
  .catch(error => console.error("链中任意步骤出错:", error));
```


#### **常见误区**

- **忘记 return**
  在 `.then()` 内部需返回 Promise 或值才能延续链式调用：
  ```js
  // 错误！nextStep 不会等待 setTimeout
  .then(() => {
    setTimeout(() => { ... }, 1000);
  })
  
  // 正确：返回 Promise
  .then(() => {
    return new Promise(resolve => {
      setTimeout(resolve, 1000);
    });
  })
  ```

- **错误未被捕获**
  未处理的 Rejected Promise 可能导致静默失败：
  ```js
  // 危险！没有 .catch()
  fetchData().then(data => ...);
  
  // 安全做法
  fetchData()
    .then(...)
    .catch(...); // 或使用 try/catch 包裹 await
  ```

  
