---
date: 2025-06-16
tags:
  - JavaScript
aliases:
  - DOM树
  - Node
  - 节点
---
## **核心概念：**

- **DOM 是什么？**
  - **文档对象模型 (Document Object Model)。**
  - 浏览器将加载的 HTML 或 XML 文档解析成一个**树形结构（DOM树）**的模型。
  - 这个树形结构中的每个部分（元素、属性、文本、注释等）都是一个**节点 (Node)** 对象。


- **DOM 树结构：**
  - **根节点 (Root Node)**: `document` 对象（代表整个文档）。
  - **元素节点 (Element Node)**: 如 `<html>`, `<div>`, `<p>`, `<span>`, `<img>` 等 HTML 标签。
  - **属性节点 (Attribute Node)**: 如 `id`, `class`, `src`, `href` 等元素的属性。
  - **文本节点 (Text Node)**: 包含在元素内的文本内容（包括空格和换行符）。
  - **注释节点 (Comment Node)**: HTML 注释 `<!-- ... -->`。
  - **文档类型节点 (DocumentType Node)**: `<!DOCTYPE html>`。
  - **关系**:
    - **父节点 (Parent Node)**: 直接包含当前节点的节点。
    - **子节点 (Child Node)**: 直接被当前节点包含的节点。
    - **兄弟节点 (Sibling Node)**: 拥有相同父节点的节点。
    - **祖先节点 (Ancestor Node)**: 当前节点父节点、父节点的父节点等直到根节点的所有节点。
    - **后代节点 (Descendant Node)**: 当前节点子节点、子节点的子节点等所有节点。


![Node节点](https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/Node%E8%8A%82%E7%82%B9.png)




## **核心 API**

### 获取元素

- **`document.getElementById(id)`** **（常用）**
  - 通过元素的唯一 `id` 属性获取**单个元素**
  - 返回：匹配的元素对象 (`Element`) 或 `null`。
    ```js
    const header = document.getElementById('main-header')
    ```


- **`document.getElementsByClassName(className)`**
  - 通过元素的 `class` 属性获取**元素集合**
  - 返回：包含所有匹配元素的**实时**集合
    ```js
    const buttons = document.getElementsByClassName('btn')
    ```


- **`document.getElementsByTagName(tagName)`**
  - 通过元素的标签名获取**元素集合**（`HTMLCollection`）
  - 返回：包含所有匹配标签元素的**实时**集合。
    ```js
    const paragraphs = document.getElementsByTagName('p')
    ```


- **`document.querySelector(cssSelector)`** **（常用）**
  - 使用 **CSS 选择器** 获取匹配的**第一个元素**。
  - 非常强大灵活，可以组合各种选择器。
  - 返回：匹配的第一个元素对象 (`Element`) 或 `null`。
    ```js
    const firstListItem = document.querySelector('ul li:first-child');
    
    const specialDiv = document.querySelector('#container .special');
    ```


- **`document.querySelectorAll(cssSelector)`**
  - 使用 **CSS 选择器** 获取匹配的**所有元素**。
  - 返回：包含所有匹配元素的**静态**集合 (`NodeList`)。
    ```js
    const allImages = document.querySelectorAll('img.thumbnail')
    ```



### 操作元素内容与属性

- **`element.innerHTML`**
  - 获取或设置元素内的 **HTML 字符串**。
  - 设置时会**解析字符串中的 HTML 标签**并替换元素原有内容。
    ```js
    console.log(div.innerHTML); // 获取
    div.innerHTML = '<strong>New</strong> content!'; // 设置 (会解析HTML)
    ```


- **`element.textContent`** **（推荐使用）**
  - 获取或设置元素及其所有后代节点的**纯文本内容**。
  - 设置时会将传入的字符串**作为纯文本处理**
    ```js
    console.log(div.textContent); // 获取所有文本（忽略标签）
    div.textContent = '<strong>This will be shown as text, not bold!</strong>'; // 设置纯文本
    ```


- **操作 HTML 属性：**
  - **`element.getAttribute(attrName)`** **（常用）**
    - 获取元素上指定 HTML 属性的**值**（字符串）。
      ```js
      const link = element.getAttribute('href');
      ```


  - **`element.setAttribute(attrName, value)`**
    - 设置元素上指定 HTML 属性的值。
    - 如果属性不存在，则创建它。
      ```js
      img.setAttribute('src', 'new-image.jpg');
      button.setAttribute('disabled', 'disabled'); // 禁用按钮
      ```
 

  - **`element.removeAttribute(attrName)`**
    - 从元素上移除指定的 HTML 属性。
      ```js
      input.removeAttribute('readonly');
      ```
 

  - **`element.hasAttribute(attrName)`** **（常用）**
    - 检查元素是否拥有指定的 HTML 属性。
    - 返回 `true` 或 `false`。
      ```js
      if (checkbox.hasAttribute('checked')) { ... }
      ```

      

- **操作 DOM 属性：**

  - **`element.classList`**
    - `add(className1, className2, ...)`: 添加一个或多个类名。
    - `remove(className1, className2, ...)`: 移除一个或多个类名。
    - `toggle(className [, force])`: 切换类名（存在则移除，不存在则添加）。`force` 为布尔值时强制添加或移除。
    - `contains(className)`: 检查是否包含某个类名。
    - `replace(oldClass, newClass)`: 替换类名。

      ```js
      div.classList.add('active', 'highlight');
      div.classList.remove('inactive');
      div.classList.toggle('visible');
      if (div.classList.contains('warning')) { ... }
      ```



### 遍历节点关系

- **父节点：**
  - `element.parentNode`: 直接父节点（可能是 `Element`, `Document`, `DocumentFragment`）。
  - **`element.parentElement`**: 直接父**元素**节点 (`Element`)。（如果父节点不是元素则返回 `null`）。


- **子节点：**
  - `element.childNodes`
    - 返回包含所有子节点的**实时**集合 (`NodeList`)，包括元素节点、文本节点、注释节点等。
  - **`element.children`** 
    - 返回包含所有子**元素**节点的**实时**集合 (`HTMLCollection`)。（只包含 `Element` 节点）。
  - `element.firstChild`
    - 第一个子节点（任何类型）。
  - `element.lastChild`
    - 最后一个子节点（任何类型）。
  - **`element.firstElementChild`:**
    - 第一个子**元素**节点。
  - **`element.lastElementChild`:**
    - 最后一个子**元素**节点。


- **兄弟节点：**
  - `element.previousSibling`
    - 前一个兄弟节点（任何类型）。
  - `element.nextSibling`
    - 后一个兄弟节点（任何类型）。
  - **`element.previousElementSibling`**
    - 前一个兄弟**元素**节点。
  - **`element.nextElementSibling`**
    - 后一个兄弟**元素**节点。



## 重要注意事项

- **DOM 操作是昂贵的：**
  - 频繁地修改 DOM（尤其是引起页面布局和样式重新计算的修改）会导致**重排 (Reflow)** 和**重绘 (Repaint)**，严重影响性能。


1. **XSS 安全：** 
   - 永远不要将**未经转义的用户输入**直接插入到 `innerHTML` 中。
   - **使用 `textContent`** 或对用户输入进行严格的验证和转义。