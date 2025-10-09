---
date: 2025-06-16
tags:
  - JavaScript
aliases:
  - 事件
---
### 事件处理

#### 核心概念：

- **事件 (Event)**
  - 用户或浏览器触发的动作（点击、鼠标移动、键盘按下、页面加载、表单提交等）。
  - 所有 DOM 节点均可触发事件。
- **事件目标 (Event Target)**
  - 事件发生的源头元素（如被点击的按钮）。
- **事件处理器 (Event Handler)**
  - 响应事件的 JavaScript 函数（也称为“事件监听器”）。
- **事件流（事件传播机制）**
  - 三个阶段
    - **捕获阶段 (Capturing Phase)**：从 `window` → 目标元素的父级
    - **目标阶段 (Target Phase)**：到达目标元素
    - **冒泡阶段 (Bubbling Phase)**：从目标元素父级 → `window`
  - **事件委托**：
    - 利用冒泡机制，在父元素监听子元素事件（例：动态列表项点击）。 
    - **优点：** 减少监听器数量（提高性能）



#### 注册事件监听器

- **`element.addEventListener(eventType, listener [, options])`**
  - **`eventType`**: 事件类型字符串（如 `'click'`, `'mouseover'`, `'keydown'`, `'submit'`）。
  - **`listener`**: **事件发生时调用的函数（回调函数）。**该函数会接收一个 `Event` 对象作为参数。
  - `options`: (可选) 一个对象，可以指定：
    - **`capture`**: `true` 表示在捕获阶段处理事件（默认为 `false`，在冒泡阶段处理）。
    - `once`: `true` 表示监听器执行一次后自动移除。
    - `passive`: `true` 表示监听器承诺**不会**调用 `event.preventDefault()`。这有助于浏览器优化滚动等性能敏感事件的响应。常用于 `'touchmove'`, `'wheel'` 事件。
    - `signal`: 一个 `AbortSignal` 对象，用于通过 `controller.abort()` 移除监听器。
  ```js
  button.addEventListener('click', function(event) {
    console.log('Button clicked!', event);
  });
  // 事件委托示例：
  document.getElementById('list').addEventListener('click', function(event) {
    if (event.target.tagName === 'LI') { // 检查点击的是否是<li>
      console.log('List item clicked:', event.target.textContent);
    }
  });
  ```



- **`element.removeEventListener(eventType, listener [, options])`**
  - 必须传入与 `addEventListener` 时**完全相同的** `eventType`, `listener` 函数引用和 `options` (特别是 `capture` 选项) 才能成功移除。
    ```js
    function handleClick(event) { ... }
    button.addEventListener('click', handleClick);
    // 稍后移除
    button.removeEventListener('click', handleClick);
    ```




#### 事件对象 (`Event`)

处理函数接收的参数（通常命名为 **`e`** 或 **`event`**）：
```js
element.addEventListener('click', function(e) {
  // 访问事件对象属性
});
```



**关键属性和方法：**

| **属性/方法**                      | **作用**                                     |
| :--------------------------------- | :------------------------------------------- |
| **`e.target` （重要） **           | 触发事件的**实际目标元素**                   |
| `e.currentTarget`                  | 绑定事件处理程序的元素（等同于 `this`）      |
| **`e.stopPropagation()` （重要）** | **阻止事件继续传播**（捕获/冒泡阶段）        |
| `e.stopImmediatePropagation()`     | 阻止当前元素后续**所有同类型事件监听器**执行 |
| **`e.preventDefault()` （重要）**  | **阻止默认行为**（如链接跳转、表单提交）     |
| **`e.type`**                       | 事件类型（如 `'click'`）                     |
| `e.clientX / e.clientY`            | 鼠标相对于**视口**的坐标                     |

- 示例
  ```js
  form.addEventListener('submit', function(event) {
      
      event.preventDefault(); // 阻止表单提交
      
  });
  ```



#### 事件委托 (Event Delegation)

**场景**：动态添加的子元素需绑定事件，或大量子元素需优化性能。

**实现**：

```js
document.getElementById('parent').addEventListener('click', function(e) {
  if (e.target.matches('.child')) { // 检查目标元素是否符合条件
    console.log('Child clicked!');
  }
});
```



### 常见事件类型

| **类别**     | **事件示例**                                                 | **说明**                   |
| :----------- | :----------------------------------------------------------- | :------------------------- |
| **鼠标事件** | `click`, `dblclick`, `mousedown`, `mouseup`, `mousemove`, `mouseenter`, `mouseleave` | `mouseenter/leave` 不冒泡  |
| **键盘事件** | `keydown`, `keyup`                                           | 建议使用 `keydown`/`keyup` |
| **表单事件** | `submit`, `input`, `change`, `focus`, `blur`                 | `input` 实时响应输入变化   |
| **窗口事件** | `load`, `resize`, `scroll`                                   | `window` 对象触发          |
| **触摸事件** | `touchstart`, `touchmove`, `touchend`                        | 移动端兼容                 |



### 结与最佳实践

- **优先使用 `addEventListener`**
  - 避免 HTML 属性和 DOM0 级绑定。
- **合理使用事件委托**
  - 减少内存占用，处理动态内容。
- **及时移除事件监听器**
  - 防止内存泄漏：`removeEventListener('click', handler)`。
- **性能优化**
  - 高频事件（如 `scroll`、`mousemove`）使用节流 (throttle) 或防抖 (debounce)。
  - 使用 `{ passive: true }` 提升滚动体验。