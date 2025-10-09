---
date: 2025-06-20
tags:
  - TypeScript
aliases:
  - interface
  - type
---
### 基础概念：定义对象形态

`interface` (接口) 和 `type` (类型别名) 最常见的共同用途是**描述一个对象的结构或形状（Shape）**。

假设我们要定义一个“用户”对象，它应该包含 `id` 和 `name`。

**使用 `interface`:**
```ts
interface User {
  readonly id: number; // 只读属性，一旦赋值后不能修改
  name: string;
  age?: number; // 可选属性，对象可以有也可以没有这个属性
}

const userA: User = {
  id: 1,
  name: "Alice",
};
```

**使用 `type`:**
```ts
type User = {
  readonly id: number;
  name: string;
  age?: number;
};

const userB: User = {
  id: 2,
  name: "Bob",
  age: 30,
};
```


在这个**基础用法上，它们几乎完全一样**，都支持只读属性 (`readonly`) 和可选属性 (`?`)。



### 核心区别(重点)

#### 区别一：扩展性 (Extensibility)

这是两者最本质的区别之一。`interface` 是“开放的”，而 `type` 是“封闭的”。

- **`interface`：支持声明合并 (Declaration Merging)** 你可以多次定义同名的 `interface`，TypeScript 会自动**将它们的属性合并成一个接口。**

  - **示例：**
    ```ts
    // 第一次声明
    interface Person {
      name: string;
    }
    
    // 第二次声明（在项目的任何其他地方）
    interface Person {
      age: number;
    }
    
    // TypeScript 会将它们合并
    const person: Person = {
      name: "Charlie",
      age: 25, // 如果不提供 age，会报错
    };
    ```



- `type`：不支持声明合并

  `type` 创建的是一个类型别名，它不支持被重复定义。如果你尝试这样做，TypeScript 会抛出一个“重复定义标识符”的错误。

  - **示例：**
    ```ts
    type Animal = {
      name: string;
    };
    
    // 尝试再次定义会直接报错：Duplicate identifier 'Animal'.
    // type Animal = {
    //   legs: number;
    // };
    ```

    如果想扩展一个 `type`，你必须使用[[组合类型|交叉类型]]**(`&`)** 来实现，这会创建一个新的类型，而不是修改原来的类型。
    ```ts
    type Animal = {
      name: string;
    };
    
    type FourLeggedAnimal = Animal & { legs: number }; // 使用交叉类型扩展
    
    const dog: FourLeggedAnimal = {
      name: "Buddy",
      legs: 4,
    };
    ```


**小结：** `interface` 是开放的，可以随时被“注入”新的属性；`type` 是封闭的，一旦定义就无法更改，只能通过交叉或联合类型创建新的类型。



#### 区别二：定义的类型范围

- `interface`：主要用于声明**对象形状**

  `interface` 的核心使命是定义对象的结构，包括对象的属性、方法以及索引签名。它也可以用来定义函数类型，但语法稍显繁琐。

  ```ts
  // 定义函数
  interface LogFunction {
    (message: string): void;
  }
  ```



- `type`：可以定义**任何类型**

  `type` 的功能更加宽泛，它不仅仅能定义对象，还可以为任何类型起一个别名，包括**原始类型、联合类型、元组、交叉类型**等。

  **详细解释与示例：** 这是 `type` 比 `interface` 灵活得多的地方。

  - **联合类型 (Union Types):**
    ```ts
    type Status = "success" | "failure" | "pending";
    // interface 无法做到
    ```

  - **原始类型别名 (Primitive Aliases):**
    ```ts
    type UserID = string;
    // interface 无法做到
    ```

  - **元组 (Tuples):**
    ```ts
    type Point = [number, number];
    // interface 无法做到
    ```

  - **函数类型（更简洁的语法）:**
    ```ts
    type LogFunction = (message: string) => void;
    ```

  - **高级类型（映射类型、条件类型等）:** `type` 是使用 TypeScript 高级类型（如 `Partial`, `Readonly`, `Record`, `Pick` 等）的基础。
    ```ts
    type PartialUser = Partial<User>; // 将 User 的所有属性变为可选
    ```


**小结：** 如果你需要定义的不仅仅是一个对象，比如一个联合类型 `string | number`，那么你只能使用 `type`。





