
## [[JSX语法|JSX]]核心规则
#### **1. 必须返回单一根元素 (Most Important!)**

这是 JSX 最核心的规则之一，所有组件返回的 JSX 都必须被一个父元素包裹。

- **规则说明**：`return` 语句后面不能直接跟多个并列的标签。

- **错误示例**：

  ```js
  // 错误代码 ❌
  return (
    <h1>标题</h1>
    <p>这是一个段落。</p>
  );
  ```

- **正确写法**：将所有元素包裹在一个父级容器中。

  - 方法一：使用 `<div>`

  - 方法二（推荐）：使用 React Fragment **(`<>...</>`)**

    - **讲解**：`React Fragment` 是一个特殊的组件，它允许你包裹多个元素，但在最终渲染到浏览器 DOM 时，它自身不会创建任何额外的 HTML 节点（如 `<div>`）。这使得 DOM 结构更干净、更轻量，是包裹元素的首选方式。

    ```js
    return (
      <>
        <h1>标题</h1>
        <p>这是一个段落。</p>
      </>
    );
    ```

    

#### **2. 属性命名采用驼峰式 (camelCase)**

由于 JSX 本质上是 JavaScript，一些 HTML 属性名与 JavaScript 的保留关键字冲突，因此需要转换。

- **规则说明**：大部分由两个或多个单词组成的 HTML 属性（如 `onclick`, `tabindex`）在 JSX 中需要写成驼峰式。

- **重要转换（必记）**：

  - `class` (HTML) → **`className`** (JSX)
    - **原因**：`class` 是 JavaScript 中用于定义类的关键字。
  - `for` (在 `<label>` 中使用) → `htmlFor` (JSX)
    - **原因**：`for` 是 JavaScript 中用于循环的关键字。

- **示例**：

  ```js
  // HTML
  // <div class="container">
  //   <label for="username">Username</label>
  // </div>
  
  // JSX
  return (
    <div className="container">
      <label htmlFor="username">Username</label>
    </div>
  );
  ```

  

#### 3.**`style` 属性的特殊用法**

在 JSX 中为元素添加内联样式时，不能直接使用字符串，而必须使用一个 JavaScript 对象。

- **规则说明**：`style` 属性接收一个对象，对象的 `key` 是 CSS 属性的驼峰式命名，`value` 是对应的样式值。

- 格式：`style={{ css属性名: '值' }}`

    - **注意**：这里有两层花括号 `{{...}}`。第一层 `{}` 表示这是一个 JSX 表达式，第二层 `{}` 代表这是一个 JavaScript 对象。



- **CSS 属性转换**：CSS 属性名中的短横线 (`-`) 需要去掉并转换为驼峰式。


  - `background-color` →  `backgroundColor`
  - `font-size` →  `fontSize`

- **示例**：

  ```js
  // 将背景色设为蓝色，字体颜色设为白色
  return (
    <div style={{ backgroundColor: 'blue', color: 'white' }}>
      这是一个样式化的 Div
    </div>
  );
  ```

  