---
date: 2025-06-11
tags:
  - JavaScript
aliases:
  - 数组方法
  - 对象方法
  - 函数方法
  - 字符串方法
---
## 对象与数组方法

### **数组（Array）常用方法**

处理数据时**推荐使用不修改原数组的方法**

#### **修改原数组的方法**

| 方法             | 作用                  | 示例                         |
| :------------- | :------------------ | :------------------------- |
| **`push()`**   | 末尾添加元素，返回新长度        | `arr.push('a')`            |
| `pop()`        | 删除末尾元素，返回被删元素       | `arr.pop()`                |
| `unshift()`    | 开头添加元素，返回新长度        | `arr.unshift('a')`         |
| `shift()`      | 删除开头元素，返回被删元素       | `arr.shift()`              |
| **`splice()`** | 添加/删除元素（万能方法）       | `arr.splice(1, 0, 'x')`    |
| `reverse()`    | 反转数组顺序              | `arr.reverse()`            |
| **`sort()`**   | 排序（默认按字符串Unicode排序） | `arr.sort((a,b) => a - b)` |

#### **不修改原数组，返回新值**

| 方法               | 作用                    | 示例                          |
| :--------------- | :-------------------- | :-------------------------- |
| **`concat()`**   | 合并数组，返回新数组            | `arr1.concat(arr2)`         |
| **`join()`**     | 数组转字符串（默认逗号分隔）        | `arr.join('-')`             |
| **`slice()`**    | 截取子数组（`[start, end)`） | `arr.slice(1, 3)`           |
| `indexOf()`      | 查找元素索引（从前往后）          | `arr.indexOf('a')`          |
| **`includes()`** | 判断是否包含元素（ES6）         | `arr.includes('a')`         |
| `flat()`         | 数组扁平化（默认深度1）          | `[1,[2]].flat()` → `[1, 2]` |

#### **遍历方法（重要！）**

| 方法              | 作用                | 示例                                   |
| :-------------- | :---------------- | :----------------------------------- |
| **`forEach()`** | 遍历数组（无返回值）        | `arr.forEach(item => {})`            |
| **`map()`**     | 映射新数组（返回新数组）      | `arr.map(x => x * 2)`                |
| **`filter()`**  | 过滤符合条件的元素         | `arr.filter(x => x > 2)`             |
| **`reduce()`**  | 累计计算（从左到右）        | `arr.reduce((sum, x) => sum + x, 0)` |
| `every()`       | 是否所有元素满足条件        | `arr.every(x => x > 0)`              |
| `some()`        | 是否有元素满足条件         | `arr.some(x => x < 0)`               |
| **`find()`**    | 返回第一个满足条件的元素（ES6） | `arr.find(x => x.id === 2)`          |
| `findIndex()`   | 返回第一个满足条件的索引（ES6） | `arr.findIndex(x => x > 3)`          |


### **[[对象核心概念|对象]]（Object）常用方法**

| 方法                 | 作用                       | 示例                              |
| :----------------- | :----------------------- | :------------------------------ |
| `Object.keys()`    | 返回自身可枚举属性名数组             | `Object.keys(obj)`              |
| `Object.values()`  | 返回自身可枚举属性值数组（ES8）        | `Object.values(obj)`            |
| `Object.entries()` | 返回 `[key,value]` 数组（ES8） | `Object.entries(obj)`           |
| `Object.assign()`  | 合并对象（浅拷贝，ES6）            | `Object.assign({}, obj1, obj2)` |


### 字符串方法

| 方法                        | 作用                  | 示例                                |
| :------------------------ | :------------------ | :-------------------------------- |
| **`split(separator)`**    | 分割字符串为数组            | `'a-b'.split('-') → ['a', 'b']`   |
| `substring(start, end)`   | 截取子字符串（end - start） | `'abc'.substring(1, 3) → 'bc'`    |
| `slice(start, end)`       | 截取子字符串（支持负数）        | `'abc'.slice(-2) → 'bc'`          |
| `replace(search,replace)` | 替换内容                | `'abc'.replace('b', 'x') → 'axc'` |
| `toUpperCase()`           | 转大写                 | `'abc'.toUpperCase()`             |
| `toLowerCase()`           | 转小写                 | `'ABC'.toLowerCase()`             |
| **`trim()`**              | 去除两端空格              | `' a '.trim() → 'a'`              |
| **`includes(str)`**       | 是否包含子串              | `'abc'.includes('bc') → true`     |
| `indexOf(str)`            | 查找子串位置              | `'abc'.indexOf('b') → 1`          |
| `startsWith(str)`         | 是否以子串开头             | `'abc'.startsWith('ab') → true`   |
| `endsWith(str)`           | 是否以子串结尾             | `'abc'.endsWith('bc') → true`     |
| `charAt(index)`           | 获取指定位置字符            | `'abc'.charAt(0) → 'a'`           |


### **函数方法**

| 方法                            | 说明               | 示例                            |
| :---------------------------- | :--------------- | :---------------------------- |
| `call(thisArg, ...args)`      | 指定 `this` 调用函数   | `fn.call(ctx, arg1)`          |
| `apply(thisArg, [argsArray])` | 数组参数调用函数         | `fn.apply(ctx, [arg1, arg2])` |
| **`bind(thisArg, ...args)`**  | 创建绑定 `this` 的新函数 | `const newFn = fn.bind(ctx)`  |


### **其他全局方法**

| 方法                               | 说明                 | 示例                          |
| :--------------------------------- | :------------------- | :---------------------------- |
| `parseInt(string, radix)`          | 字符串转整数         | `parseInt('10', 10) → 10`     |
| `parseFloat(string)`               | 字符串转浮点数       | `parseFloat('3.14') → 3.14`   |
| `isNaN(value)`                     | 检查是否为 `NaN`     | `isNaN(NaN) → true`           |
| **`JSON.parse(string)`**           | JSON 字符串转对象    | `JSON.parse('{"a":1}')`       |
| **`JSON.stringify(obj)`**          | 对象转 JSON 字符串   | `JSON.stringify({a:1})`       |
| `Promise.resolve()`                | 创建已解决的 Promise | `Promise.resolve(42)`         |
| `Promise.reject()`                 | 创建已拒绝的 Promise | `Promise.reject(error)`       |
| **`Promise.all(iterable)`**        | 并行执行多个 Promise | `Promise.all([p1, p2])`       |
| **`setTimeout(callback, delay)`**  | 延时执行函数         | `setTimeout(() => {}, 1000)`  |
| **`setInterval(callback, delay)`** | 定时循环执行         | `setInterval(() => {}, 1000)` |

