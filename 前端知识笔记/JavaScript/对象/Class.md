---
date: 2025-06-12
tags:
  - JavaScript
aliases: []
---
### **基本概念**

- **语法糖**
  - 本质是基于原型继承的语法糖
  - 替代了 ES5 的构造函数 + [[原型与原型链|原型链]]写法

  ```js
  class Person {
    constructor(name) {
      this.name = name;
    }
    
    sayHello() {
      console.log(`Hello, I'm ${this.name}`);
    }
  }
  
  // 等价于
  function Person(name) {
    this.name = name;
  }
  
  Person.prototype.sayHello = function() {
    console.log(`Hello, I'm ${this.name}`);
  };
  ```


- **必须用 `new` 调用**

  ```js
  class Person {
    constructor(name) {
      this.name = name;
    }
    
    sayHello() {
      console.log(`Hello, I'm ${this.name}`);
    }
  }
  
  const ming = new Person("ming")	// new 关键字
  ming.sayHello()
  ```



### **核心组成部分**

#### 构造函数 `constructor`

```js
class Person {
  constructor(name) {
    this.name = name; // 实例属性初始化
  }
}
```

#### 实例方法

```js
class Person {
  sayHello() {
    return `Hello, ${this.name}!`;
  }
}
```


#### 静态方法/属性 `static`

- **作用：**静态成员（方法/属性）**只能通过类名调用，不能通过实例访问**。
```js
class MyClass {
  static staticMethod() {
    return "静态方法";
  }
}

MyClass.staticMethod(); // ✅ 正确
new MyClass().staticMethod(); // ❌ TypeError: 实例无法访问
```



#### 私有字段（ES2022）

- **语法：**`#`
- **作用：**外部无法直接访问
```js
class Wallet {
  #balance = 0; // 私有字段

  deposit(amount) {
    this.#balance += amount;
  }
}
let wallet = new Wallet();
wallet.#balance; // SyntaxError: Private field must be declared in an enclosing class
```



#### Getter/Setter

**作用：**控制类属性访问的特殊方法
- **封装与验证**
  - 在设置属性值时进行有效性检查（如类型、范围验证）
  - 避免非法值被直接赋值
- **计算属性**
  - 动态生成属性值（基于其他属性计算）
  - 无需显式调用方法即可获取计算结果
- **访问控制**
  - 创建“伪属性”（实际无真实存储，通过逻辑生成）
  - 实现只读属性（只提供 Getter 不提供 Setter）
- **副作用管理**
  - 在属性被访问/修改时触发特定操作（如日志、事件通知）

```js
class User {
  constructor(firstName, lastName) {
    this._firstName = firstName;
    this._lastName = lastName;
  }

  // Getter：动态计算全名
  get fullName() {
    return `${this._firstName} ${this._lastName}`;
  }

  // Setter：拆分赋值并验证
  set fullName(value) {
    if (typeof value !== 'string') throw new Error("必须为字符串");
    const [first, last] = value.split(' ');
    if (!first || !last) throw new Error("格式：'名 姓'");
    
    this._firstName = first;
    this._lastName = last;
  }

  // 只读属性（无Setter）
  get initials() {
    return this._firstName[0] + this._lastName[0];
  }
}

// 使用示例
const user = new User("张", "三");

console.log(user.fullName); // "张 三" (触发Getter)
console.log(user.initials); // "张三" (只读)

user.fullName = "李 四"; // 触发Setter
console.log(user.fullName); // "李 四"

user.fullName = "Invalid"; // 报错：格式不符合要求
```


#### Class继承

##### 1. `extends` 关键字
使用 **`extends`** 关键字来实现继承。

```javaScript
// 父类 (Parent Class)
class Animal {
  constructor(name) {
    this.name = name;
    this.speed = 0;
  }

  eat() {
    console.log(`${this.name} is eating.`);
  }

  run(speed) {
    this.speed = speed;
    console.log(`${this.name} is running at ${this.speed} km/h.`);
  }
}

// 子类 (Child Class)
class Dog extends Animal {
  // Dog 自动拥有了 Animal 的所有属性和方法
  bark() {
    console.log('Woof! Woof!');
  }
}

// --- 使用 ---
const myDog = new Dog('Buddy');
myDog.eat();   // 输出: Buddy is eating. (继承自 Animal)
myDog.run(10); // 输出: Buddy is running at 10 km/h. (继承自 Animal)
myDog.bark();  // 输出: Woof! Woof! (Dog 自己的方法)
```

##### 2. `super` 关键字
`super` 是一个非常重要的关键字，它有两个主要用途：

**a) 作为函数调用 `super()`：调用父类的构造函数**

如果子类有自己的构造函数 (`constructor`)，那么**必须**在 `constructor` 的第一行调用 `super()`。这会执行父类的 `constructor`，确保父类的属性被正确初始化。
```JavaScript
class Dog extends Animal {
  constructor(name, breed) {
    // 1. 调用父类的构造函数，并传入父类需要的参数
    super(name); // 必须在 this 之前调用

    // 2. 初始化子类自己的属性
    this.breed = breed;
  }

  bark() {
    console.log('Woof! Woof!');
  }
}

const myDog = new Dog('Buddy', 'Golden Retriever');
console.log(myDog.name);  // 输出: Buddy (通过 super(name) 初始化)
console.log(myDog.breed); // 输出: Golden Retriever
```

> **重点**: 在子类的 `constructor` 中，`this` 关键字只有在 `super()` 被调用之后才能使用。

**b) 作为对象使用 `super`：调用父类的方法**

当子类想重写（Override）父类的方法，但又想在其中保留父类的行为时，可以使用 `super` 来调用父类的同名方法。

##### 3 方法重写（Method Overriding）

子类可以定义一个和父类同名的方法，从而“覆盖”掉父类的行为。这就是方法重写。
```JavaScript
class Cat extends Animal {
  // 重写父类的 eat 方法
  eat() {
    // 你可以在这里完全重写，或者...
    console.log('The cat is picky...');
    
    // ...使用 super 来调用父类的原始方法
    super.eat(); 
    
    console.log('...and now it is full.');
  }
}

const myCat = new Cat('Mitty');
myCat.eat();
// 输出:
// The cat is picky...
// Mitty is eating.
// ...and now it is full.
```