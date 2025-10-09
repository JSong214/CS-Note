---
date: 2025-06-20
tags:
  - TypeScript
aliases:
  - 枚举
---
### 什么是枚举 (Enum)？

简单来说，枚举是一个**相关常量的集合。**

未使用枚举时：

```ts
function handleStatus(status: number) {
  if (status === 0) {
    console.log("处理中");
  } else if (status === 1) {
    console.log("成功");
  } else if (status === 2) {
    console.log("失败");
  }
}
// 调用时可读性差，数字 0, 1, 2 的含义不明确
handleStatus(1);
```

**使用枚举后：**

```ts
enum Status {
  Processing, // 0
  Success,    // 1
  Failure     // 2
}

function handleStatus(status: Status) {
  if (status === Status.Processing) {
    console.log("处理中");
  } else if (status === Status.Success) {
    console.log("成功");
  } else if (status === Status.Failure) {
    console.log("失败");
  }
}
// 调用时代码意图清晰
handleStatus(Status.Success);
```



### 枚举的类型

TypeScript 中的枚举主要分为三类：数字枚举、字符串枚举和异构枚举。

#### 1.数字枚举 (Numeric Enums)

这是最常见的枚举类型。

- **默认行为**：第一个成员的默认值为 `0`，后续成员会在此基础上自动递增 `1`。

  ```ts
  enum Direction {
    Up,    // 0
    Down,  // 1
    Left,  // 2
    Right  // 3
  }
  console.log(Direction.Left); // 输出: 2
  ```

- **手动设置初始值**：你可以手动为成员设置值，未设置值的后续成员会从最后一个手动设置值的成员开始递增。

  ```ts
  enum HttpStatus {
    Ok = 200,
    NotFound = 404,
    InternalServerError = 500
  }
  console.log(HttpStatus.NotFound); // 输出: 404
  
  enum FileAccess {
    None,       // 0
    Read = 2,   // 2
    Write,      // 3 (从 2 开始递增)
    ReadWrite = Read | Write, // 6 (位运算)
  }
  console.log(FileAccess.Write); // 输出: 3
  ```

  
##### 【重要知识点】反向映射 (Reverse Mapping)

**这是数字枚举独有的特性。** TypeScript 会为数字枚举额外生成一个“反向映射”的查找对象，这意味着你不仅可以通过成员名获取值，还**可以通过值获取成员名。**

```ts
enum Direction {
  Up,
  Down
}

// 1. 通过名称获取值 (正向映射)
let upValue: number = Direction.Up; // upValue 的值为 0

// 2. 通过值获取名称 (反向映射)
let upName: string = Direction[0]; // upName 的值为 "Up"

console.log(upValue); // 0
console.log(upName);  // "Up"
```

**底层原理**：

上面的 `Direction` 枚举在编译成 JavaScript 后，大致是这样的：

```js
var Direction;
(function (Direction) {
    Direction[Direction["Up"] = 0] = "Up";
    Direction[Direction["Down"] = 1] = "Down";
})(Direction || (Direction = {}));
```

`Direction["Up"] = 0` 实现了正向映射，而 `Direction[0] = "Up"` 实现了反向映射。


#### 2.字符串枚举 (String Enums)

字符串枚举的每个成员都必须使用字符串字面量或者另一个字符串枚举成员进行初始化。

```ts
enum LogLevel {
  Info = "INFO",
  Warning = "WARN",
  Error = "ERROR"
}

console.log(LogLevel.Warning); // 输出: "WARN"
```

##### 【重要知识点】字符串枚举的特点

- **没有反向映射**：这是它与数字枚举最核心的区别。你不能通过值 `"INFO"` 来获取成员名 `Info`。
- **可读性更强**：编译后的 JavaScript 值是字符串本身，这在调试时非常有用。例如，当你 `console.log` 一个值为字符串枚举的变量时，你会直接看到有意义的字符串（如 `"INFO"`），而不是一个可能意义不明的数字（如 `0`）。
- **更明确的意图**：字符串枚举的值是固定的，不会像数字枚举那样“意外”地自动递增。

**编译后的 JavaScript：**

```js
var LogLevel;
(function (LogLevel) {
    LogLevel["Info"] = "INFO";
    LogLevel["Warning"] = "WARN";
    LogLevel["Error"] = "ERROR";
})(LogLevel || (LogLevel = {}));
```

可以看到，它就是一个简单的键值对对象，没有反向映射的逻辑。



### `const` 枚举【核心重点】

`const` 枚举是 TypeScript 提供的一种性能优化手段。

**定义方式**：

```ts
const enum Direction {
  Up,
  Down,
  Left,
  Right
}

let myDirection = Direction.Up;
```

**与普通枚举的区别**： 
​	`const` 枚举在编译后会**完全被移除**，所有对它的引用都会被**直接替换成对应的内联值**。


**编译结果对比**：

- 普通枚举 `enum`

  ```ts
  // TypeScript
  enum Status { Success = 1 }
  let code = Status.Success;
  
  // JavaScript (编译后)
  var Status;
  (function (Status) {
      Status[Status["Success"] = 1] = "Success";
  })(Status || (Status = {}));
  let code = Status.Success; // 运行时需要访问 Status 对象
  ```

- 常量枚举 `const enum`

  ```ts
  // TypeScript
  const enum Status { Success = 1 }
  let code = Status.Success;
  
  // JavaScript (编译后)
  let code = 1; // 直接内联，Status 对象完全消失
  ```

  
#### **`const enum` 的优缺点**：

- **优点**：
  - **性能提升**：由于没有额外的运行时对象和查找过程，代码执行效率更高。
  - **减少打包体积**：编译后没有生成枚举的 JavaScript 对象，代码量更少。
- **缺点**：
  - **没有反向映射**：因为它在编译后就消失了。
  - **无法在运行时迭代**：你不能用 `Object.keys(Direction)` 等方式遍历 `const` 枚举。



### 枚举作为类型使用

你可以将枚举名用作类型，限制变量只能被赋予该枚举的成员。

```ts
enum UserRole {
  Admin,
  Editor,
  Guest
}

interface User {
  name: string;
  role: UserRole; // 类型注解
}

const user: User = {
  name: "Alice",
  role: UserRole.Admin // 正确
};

// const invalidUser: User = {
//   name: "Bob",
//   role: 100 // 错误！Type '100' is not assignable to type 'UserRole'.
// };
```

