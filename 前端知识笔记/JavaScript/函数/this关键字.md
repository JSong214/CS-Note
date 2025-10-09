---
date: 2025-06-12
tags:
  - JavaScript
aliases:
  - this
---
**核心原则：`this` 的值取决于函数的调用方式**
### **`this` 绑定的基本规则（优先级从低到高）：**

#### **默认绑定：**

- **规则：**当函数作为**独立函数调用**（不是作为对象方法、不是通过 `new`、`call/apply/bind` 调用）时，`this` 指向**全局对象**。
- **严格模式：** 在严格模式 (`'use strict'`) 下，独立函数调用中的 `this` 是 `undefined`，避免了意外指向全局对象。

- **示例：**
  ```js
  function showThis() {
      console.log(this);
  }
  showThis(); // 浏览器: Window {...},  严格模式: undefined
  ```


#### **隐式绑定：**

- **规则：** 当函数作为**对象的方法**被调用时,`this` 指向**调用该方法的对象**（`obj`）。
- **关键：** 关注点是函数调用时**点号 `.` 或方括号 `[]` 前面的那个对象**。

- **示例：**
  ```js
  const person = {
      name: 'Alice',
      greet: function() {
          console.log(`Hello, my name is ${this.name}`);
      }
  };
  person.greet(); // 'Hello, my name is Alice' - `this` 指向 `person`
  ```


#### **显式绑定：**

- **规则：** 使用函数的 `call()`, `apply()`, 或 `bind()` 方法，可以**显式指定**函数执行时 `this` 的值。

- **`call(thisArg, arg1, arg2, ...)`：** 立即调用函数，第一个参数是 `this` 值，后续参数是函数调用时传入的参数列表（逗号分隔）。

- **`apply(thisArg, [argsArray])`：** 立即调用函数，第一个参数是 `this` 值，第二个参数是一个数组（或类数组对象），包含函数调用时传入的参数。

- **`bind(thisArg, arg1, arg2, ...)`：** **不会立即调用函数**，而是返回一个**新的函数**。这个新函数被调用时，其 `this` 值被永久绑定到 `bind` 的第一个参数 (`thisArg`)，并且可以预先传入部分参数（柯里化）。`bind` 是解决隐式丢失的常用方法。

- **示例：**
  ```js
  function introduce(lang) {
      console.log(`I speak ${lang}. My name is ${this.name}`);
  }
  
  const person1 = { name: 'Bob' };
  const person2 = { name: 'Charlie' };
  
  // call
  introduce.call(person1, 'English'); // 'I speak English. My name is Bob'
  
  // apply
  introduce.apply(person2, ['Spanish']); // 'I speak Spanish. My name is Charlie'
  
  // bind - 创建新函数
  const introduceBob = introduce.bind(person1, 'French');
  introduceBob(); // 'I speak French. My name is Bob' (无论何时调用，this 始终指向 person1)
  ```


#### **`new` 绑定：**

- **规则：** 当使用 `new` 关键字调用一个**构造函数**时，会发生以下事情
  - 创建一个全新的空对象 `{}`。
  - 这个新对象的 `[[Prototype]]`（即 `__proto__`）被链接到构造函数的 `prototype` 对象。
  - 构造函数内部的 `this` 被**绑定到这个新创建的对象**。
  - 执行构造函数内部的代码（通常用于初始化新对象）。
  - 如果构造函数没有显式返回一个对象，则自动返回这个新创建的对象（`this` 指向的对象）。

- **示例：**
  ```js
  function Person(name) {
      this.name = name;
      this.greet = function() {
          console.log(`Hi, I'm ${this.name}`);
      };
  }
  const alice = new Person('Alice');
  alice.greet(); // 'Hi, I'm Alice' - `this` 指向新创建的 `alice` 对象
  ```  



### **箭头函数中的 `this`**

- **核心特点：** 箭头函数 **没有自己的 `this`**。它内部的 **`this` 值根据外层[[作用域与闭包|作用域]]来决定**。
- **行为：** 相当于在**定义**箭头函数**时**，就**捕获了**它**外层（非箭头）函数**的 `this` 值。
- **示例与对比：**
  ```js
  const obj = {
      
      traditionalFunc: function() {
          console.log('Traditional:', this); // `this` 指向 obj (隐式绑定)
          setTimeout(function() {
              console.log('Traditional in setTimeout:', this); // `this` 指向 window/global (默认绑定，隐式丢失!)
          }, 100);
      },
      
      arrowFunc: function() {
          console.log('Arrow outer:', this); // `this` 指向 obj (隐式绑定)
          setTimeout(() => {
              console.log('Arrow in setTimeout:', this); // `this` 继承自 arrowFunc 的 this (即 obj)
          }, 100);
      }
  };
  
  obj.traditionalFunc();
  obj.arrowFunc();
  ```

- **重要限制：**
  - 箭头函数**不能**用作构造函数（使用 `new` 调用会报错）。
  - `call()`, `apply()`, `bind()` **无法改变**箭头函数的 `this`（因为它没有自己的 `this`）。