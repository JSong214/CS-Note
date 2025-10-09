---
date: 2025-06-17
tags:
  - Vue
---
### 核心：

Vue 3 使用 `Proxy` 的核心思想是**创建一个原始对象的“代理”**。当程序对这个代理对象进行任何操作（如读取、修改、添加属性）时，`Proxy` 都能“拦截”这些操作。这就给了 Vue 一个机会，在操作发生前后执行一些额外的逻辑，从而实现**依赖追踪**和**变更通知**。


### 响应式实现过程

Vue 的响应式系统可以简化为两个核心部分：`reactive` 和 `effect`。

#### **`reactive` 函数**：创建代理

- 这个函数接收一个普通 JavaScript 对象（比如 `const data = { count: 1 }`），然后返回一个基于 `Proxy` 的代理对象。

  ```js
  import { reactive } from 'vue';
  
  const original = { name: 'Alice', age: 25 };
  const state = reactive(original); // state 就是一个代理对象
  
  // 现在操作 state 就等于在 Proxy 的监视下操作 original
  console.log(state.name); // 读取
  state.age = 26; // 修改
  state.city = 'New York'; // 新增
  delete state.age; // 删除
  ```

#### **`effect` 函数**：收集依赖 & 触发更新

`effect` 函数会立即执行一次传入的函数，并且在这个过程中，它会“追踪”所有被读取的响应式数据。

- **简化的内部工作流程：**
  - `effect` 运行 -> 读取数据 -> 触发 `get` -> **收集依赖** -> 修改数据 -> 触发 `set` -> **派发更新** -> `effect` 重新运行



### Proxy 的具体使用方法

`Proxy` 是 ES6 的一个原生特性，它的基本语法非常直接：

```js
const proxy = new Proxy(target, handler);
```
- `target`: 你想要代理的原始对象（可以是任何类型的对象，包括数组、函数等）。
- `handler`: 一个配置对象，里面定义了要拦截的操作，这些操作被称为“陷阱”（Trap）。



#### **Handler Traps**

- **`get(target, property, receiver)`**

  当读取代理对象的属性时触发。

  - `target`: 原始对象。
  - `property`: 被读取的属性名。
  - `receiver`: 代理对象本身（或者继承自代理的对象）。

  ```js
  const user = { name: 'John', age: 30 };
  
  const handler = {
    get(target, property) {
      console.log(`正在读取属性: ${property}`);
      // 在这里执行依赖收集
      return Reflect.get(target, property); // 使用 Reflect.get 来安全地获取原始值
    }
  };
  
  const userProxy = new Proxy(user, handler);
  
  console.log(userProxy.name); // 输出: "正在读取属性: name", 然后输出 "John"
  console.log(userProxy.age);  // 输出: "正在读取属性: age", 然后输出 30
  ```

  - 为什么要用 `Reflect`?
    在陷阱中使用 `Reflect` 方法可以确保正确处理 `this` 指向等边界情况，是最佳实践。



- `set(target, property, value, receiver)`

  当给代理对象的属性赋值时触发。

  - `target`: 原始对象。
  - `property`: 被设置的属性名。
  - `value`: 新的属性值。
  - `receiver`: 代理对象。
  - **返回值**: `set` 操作必须返回一个布尔值。`true` 表示赋值成功，`false` 表示失败（在严格模式下会抛出 `TypeError`）。

  ```js
  const user = { name: 'John', age: 30 };
  
  const handler = {
    set(target, property, value) {
      console.log(`正在设置属性: ${property}，新值为: ${value}`);
      // 在这里执行派发更新
      return Reflect.set(target, property, value); // 使用 Reflect.set 来安全地设置值
    }
  };
  
  const userProxy = new Proxy(user, handler);
  
  userProxy.age = 31; // 输出: "正在设置属性: age，新值为: 31"
  console.log(user.age); // 输出: 31
  ```

  

-  `deleteProperty(target, property)`

  当使用 `delete` 操作符删除代理对象的属性时触发。

  - `target`: 原始对象。
  - `property`: 被删除的属性名。
  - **返回值**: 必须返回一个布尔值，表示删除是否成功。

  ```js
  const user = { name: 'John', age: 30 };
  
  const handler = {
    deleteProperty(target, property) {
      console.log(`正在删除属性: ${property}`);
      // 在这里也可以执行派发更新
      return Reflect.deleteProperty(target, property);
    }
  };
  
  const userProxy = new Proxy(user, handler);
  
  delete userProxy.age; // 输出: "正在删除属性: age"
  console.log(user);   // 输出: { name: 'John' }
  ```

  