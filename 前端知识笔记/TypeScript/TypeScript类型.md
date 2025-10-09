---
date: 2025-06-20
tags:
  - TypeScript
---
### 原始类型与特殊类型

#### 原始类型

- `string` (字符串)
- `number` (数字)
- `boolean` (布尔)
- `null`
- `undefined`
- `symbol`
- `bigint`


#### 特殊类型

- **`any` (任意类型)**
  - 将一个变量的类型指定为 `any` 时，TypeScript**不会对这个变量进行任何类型检查**
  - **风险**: 滥用 `any` 会使 TypeScript 失去其核心优势，让你的代码充满潜在的运行时错误。**应尽可能避免使用 `any`。**
  - 示例：

    ```ts
    let anything: any = "hello";
    anything = 123; // OK
    anything = { a: 1 }; // OK
    
    // 以下代码在编译时都不会报错，但第二行在运行时会崩溃
    console.log(anything.a); // 访问属性，OK
    anything.nonExistentMethod(); // 调用不存在的方法，编译通过，但运行时 TypeError
    ```

    

- **`unknown` (未知类型)**
  `unknown` 是 `any` 的类型安全版本，它表示一个我们不确定其类型的变量。
  - 重要知识点: `any` vs `unknown` 的核心区别
    - **`any`**: 放弃检查，你可以对它做任何事。
    - **`unknown`**: 保持检查，你**不能**对它做任何事，除非你先**收窄 (narrow)** 它的类型。

  - **如何收窄 `unknown` 类型**:
    1. **使用 `typeof` 类型守卫**:
       ```ts
       let value: unknown = "hello world";
       if (typeof value === "string") {
           // 在这个代码块内，TypeScript 知道 value 是 string 类型
           console.log(value.toUpperCase());
       }
       ```

    2. **使用 `instanceof` 类型守卫** (用于类实例):
       ```ts
       class Dog { bark() {} }
       let pet: unknown = new Dog();
       if (pet instanceof Dog) {
           pet.bark(); // OK
       }
       ```

    3. **使用类型断言 (`as`)**: (当你自己能百分百确定类型时)
       ```ts
       let val: unknown = "this is a string";
       let strLength = (val as string).length;
       ```

  - **最佳实践**: **在不确定类型时，优先使用 `unknown` 而不是 `any`。** 


- **`void` (无返回)**

  `void` 主要用于表示一个函数**没有任何返回值**。

  - **知识点**:
    - 一个不带 `return` 语句的函数，或者 `return;`，或者 `return undefined;` 的函数，其返回值类型都应被声明为 `void`。
    - 声明为 `void` 的变量没什么用，只能被赋值为 `undefined` 或 `null` (在 `strictNullChecks` 关闭时)。

    ```ts
    function logMessage(message: string): void {
        console.log(message);
        // 这个函数没有 return 语句
    }
    ```


- **`never` (永不返回)**

  `never` 类型表示**永远不会发生**的值的类型。这听起来很抽象，主要用于以下两种情况：

  - **总是抛出错误的函数**:
    ```ts
    function throwError(message: string): never {
        throw new Error(message);
        // 函数在这里就中断了，永远不会有“返回值”
    }
    ```

  - **无限循环的函数**:
    ```ts
    function infiniteLoop(): never {
        while (true) {
            // 这个函数永远不会结束，因此也永远不会返回任何东西
        }
    }
    ```

  - 重要知识点: `void` vs `never`
    - `void`: 函数**可以**正常执行完毕，只是**没有**返回值。
    - `never`: 函数**永远无法**正常执行完毕。