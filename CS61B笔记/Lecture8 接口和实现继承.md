## Lecture8 接口和实现继承



### **核心思想：代码的通用性 (Generality)**

在编程中，我们经常会遇到一个问题：如何编写能够处理多种不同数据类型的通用代码？

例如，我们想写一个 `max` 方法来返回两个对象中较大的一个。如果我们为 `Dog` 类写了一个 `max` 方法，那么对于 `Cat` 类，或者其他任何可以比较大小的类，我们都需要重写一遍 `max` 方法。这严重违反了软件工程的 **DRY (Don't Repeat Yourself)** 原则。

```java
// 仅适用于Dog类的max方法
public static Dog max(Dog d1, Dog d2) {
    if (d1.size > d2.size) {
        return d1;
    }
    return d2;
}
```

为了解决这个问题，我们需要一种机制，能够告诉编译器：“**嘿，我保证所有传入这个方法的对象，都拥有一个可被调用的、用于比较大小的方法。**” 这就是**继承 (Inheritance)** 发挥作用的地方。

继承主要分为两种形式：接口继承和实现继承。

------



### **1. 接口继承 (Interface Inheritance) - "is-a" 关系的体现**

接口继承的核心是建立一种 **"是什么 (is-a)"** 的关系。它只关心一个类**能做什么 (what it can do)**，而不关心它**具体是怎么做的 (how it does it)**。



#### **1.1 Hypernyms（上位词）与 Hyponyms（下位词）**

- **Hypernym**：一个更宽泛、更一般的概念。例如，“Animal” 是 “Dog” 的 Hypernym。
- **Hyponym**：一个更具体、更特殊的概念。例如，“Dog” 是 “Animal” 的 Hyponym。

在面向对象编程中，这种关系通常通过父类/接口（Superclass/Interface）和子类（Subclass）来体现。我们可以说，子类是父类的一种（A subclass **is-a** superclass）。



#### **1.2 `interface` 关键字：定义一种规范或契约**

在 Java 中，`interface` 是实现接口继承的核心工具。它定义了一个**契约 (contract)**，规定了任何实现了该接口的类**必须**提供某些方法的具体实现。

一个接口只包含：

- **方法签名 (Method Signatures)**：定义了方法的名称、参数和返回类型，但没有方法体。
- **常量 (Constants)**：`public static final` 的变量。

**示例：创建一个 `OurComparable` 接口**

让我们创建一个接口，该接口保证任何实现它的类都具有比较大小的能力。

```java
public interface OurComparable {
    /**
     * 返回一个负整数、零或正整数，
     * 分别表示此对象小于、等于或大于指定对象。
     */
    int compareTo(Object o);
}
```

这个接口就像一份合同，它声明：“任何声称自己是 `OurComparable` 的类，都必须提供一个叫做 `compareTo` 的方法，该方法接受一个 `Object` 参数并返回一个 `int`。”



#### **1.3 `implements` 关键字：履行契约**

一个类通过 `implements` 关键字来表示它将遵守某个接口的契约。

**示例：让 `Dog` 类实现 `OurComparable` 接口**

```java
public class Dog implements OurComparable {
    private String name;
    public int size;

    public Dog(String n, int s) {
        name = n;
        size = s;
    }

    public void bark() {
        System.out.println(name + " says: bark");
    }

    /**
     * 履行 OurComparable 接口的契约
     */
    @Override
    public int compareTo(Object o) {
        // 我们需要将 Object 类型的 o 转换为 Dog 类型
        Dog otherDog = (Dog) o;
        // 直接返回 size 的差值，巧妙地满足了接口要求
        return this.size - otherDog.size;
    }
}
```

**关键点**：

- `Dog` 类现在**是 (is-a)** 一个 `OurComparable`。
- `Dog` 类**必须**提供 `compareTo` 方法的具体实现。
- `@Override` 注解是一个好习惯，它告诉编译器我们正在重写一个来自父类或接口的方法。如果方法签名有误，编译器会报错。



#### **1.4 接口继承带来的好处：通用的 `max` 方法**

现在，我们可以编写一个非常通用的 `max` 方法，它可以接受任何实现了 `OurComparable` 接口的对象。

```java
public class Maximizer {
    public static OurComparable max(OurComparable[] items) {
        int maxDex = 0;
        for (int i = 0; i < items.length; i += 1) {
            // 我们调用 item[i] 的 compareTo 方法
            if (items[i].compareTo(items[maxDex]) > 0) {
                maxDex = i;
            }
        }
        return items[maxDex];
    }
}
```

**工作原理剖析：静态类型 vs. 动态类型**

- **静态类型 (Static Type)**：变量在**编译时**的类型。在 `max` 方法中，`items[i]` 和 `items[maxDex]` 的静态类型都是 `OurComparable`。编译器只知道这些对象一定有 `compareTo` 方法，所以编译可以通过。
- **动态类型 (Dynamic Type)**：变量在**运行时**实际指向的对象的类型。`items[i]` 的动态类型可能是 `Dog`，也可能是 `Cat`（如果 `Cat` 也实现了 `OurComparable`）。

当 `items[i].compareTo(...)` 这行代码执行时，Java 运行时系统会检查 `items[i]` 的**动态类型**（比如是 `Dog`），然后调用该类中具体实现的 `compareTo` 方法。这个过程被称为**动态方法选择 (Dynamic Method Selection)**。

这正是接口继承的强大之处：**它允许我们编写依赖于接口（抽象规范）而不是具体实现的代码，从而实现极高的灵活性和可重用性。**

------



### **2. 实现继承 (Implementation Inheritance) - "extends" 的力量**

实现继承不仅继承了父类的**方法签名 (what)**，还继承了父类的**具体实现 (how)**。它主要用于代码复用。



#### **2.1 `extends` 关键字：继承代码**

在 Java 中，通过 `extends` 关键字来实现类与类之间的继承。子类（subclass）会自动获得父类（superclass）中所有 `public` 和 `protected` 的方法和变量。

**示例：创建一个 `RotatingSLList`**

假设我们已经有了一个 `SLList` (单向链表) 类。现在我们想创建一个 `RotatingSLList`，它的功能是每次添加元素后，都将最后一个元素移动到最前面。

```java
// 假设已有的 SLList 类
public class SLList<Item> {
    // ... 内部实现，如 addLast, getLast, removeLast 等方法
    public void addLast(Item x) { /* ... */ }
    public Item getLast() { /* ... */ }
    public Item removeLast() { /* ... */ }
    // ...
}

// 通过继承来创建 RotatingSLList
public class RotatingSLList<Item> extends SLList<Item> {
    /**
     * 旋转操作：将最后一个元素移动到最前面
     */
    public void rotateRight() {
        Item x = removeLast(); // 直接复用父类的 removeLast 方法
        addFirst(x);           // 复用父类的 addFirst 方法
    }
}
```

这里，`RotatingSLList` 无需重写 `removeLast` 和 `addFirst`，它直接**继承并复用**了 `SLList` 的代码实现。这就是实现继承的核心优势：**代码复用**。



#### **2.2 方法重写 (Method Overriding)**

子类可以提供一个与父类方法签名完全相同的新实现，来覆盖（override）父类的行为。

**示例：修改 `RotatingSLList` 的 `addLast` 行为**

我们希望 `RotatingSLList` 在每次调用 `addLast` 后自动进行旋转。

```java
public class RotatingSLList<Item> extends SLList<Item> {
    @Override
    public void addLast(Item x) {
        super.addLast(x); // 1. 调用父类（SLList）的 addLast 方法，完成添加操作
        rotateRight();    // 2. 执行自身的旋转操作
    }
    
    public void rotateRight() {
        Item x = removeLast();
        addFirst(x);
    }
}
```

**关键点**：

- `@Override` 注解再次出现，确保我们正确地重写了方法。
- `super` 关键字用于调用父类的版本。在这里，`super.addLast(x)` 确保了元素被实际添加到链表中，之后再执行子类特有的 `rotateRight` 逻辑。



#### **2.3 `default` 方法：接口中的实现继承**

从 Java 8 开始，接口中可以包含 `default` 方法。`default` 方法为接口中的方法提供了一个默认的实现。任何实现该接口的类都将自动继承这个默认实现，除非它自己选择重写（override）该方法。

这模糊了接口继承和实现继承的界限，允许接口也提供可复用的代码。

**示例：在接口中提供 `print` 方法**

```java
public interface List61B<Item> {
    // ... 其他抽象方法，如 addLast, get 等

    /**
     * 一个默认的打印方法
     */
    default public void print() {
        System.out.print("[");
        for (int i = 0; i < size(); i += 1) {
            System.out.print(get(i) + " ");
        }
        System.out.println("]");
    }
}

public class SLList<Item> implements List61B<Item> {
    // ... 必须实现 List61B 的所有抽象方法
    // 但是，SLList 自动获得了 print() 方法的实现，无需自己编写！
}
```

现在，`SLList` 的实例可以直接调用 `print()` 方法，因为它从 `List61B` 接口中**继承**了这个方法的**实现**。

------



### **3. 接口继承 vs. 实现继承：如何选择？**

| 特性       | 接口继承 (`implements`)                                  | 实现继承 (`extends`)                        |
| ---------- | -------------------------------------------------------- | ------------------------------------------- |
| **目的**   | 定义 "is-a" 关系，规定**能做什么 (what)**                | 复用代码，继承**怎么做 (how)**              |
| **关系**   | 类可以实现**多个**接口                                   | 类只能继承**一个**父类（Java 的单继承限制） |
| **耦合度** | 松耦合。依赖于抽象（接口），而非具体实现                 | 紧耦合。子类与父类的实现细节紧密相连        |
| **灵活性** | 非常高。任何类，无论其继承结构如何，都可以实现同一个接口 | 较低。子类的类型在继承时就被固定了          |

**经验法则**：

- **优先使用接口继承 (Favor Interface Inheritance)**。这使得你的代码更加灵活，耦合度更低。当你需要定义一个类型（比如 `OurComparable`, `List`），并且希望有多种不同的实现方式时，接口是最佳选择。
- **仅在明确的 "is-a" 关系并且需要大量代码复用时，才使用实现继承 (Use Implementation Inheritance for code reuse)**。例如，`Poodle` is-a `Dog`，它们共享大量代码，使用 `extends` 是合适的。

------



### **4. 抽象数据类型 (Abstract Data Types - ADT)**

这个概念与接口继承紧密相关。ADT 是对数据的一种**数学上的、抽象的描述**。它只关注：

1. **数据 (Data)**：它存储了什么信息？
2. **操作 (Operations)**：它可以进行哪些操作？
3. **行为 (Behavior)**：这些操作会产生什么效果？

ADT **完全不关心**数据的**具体实现方式**。

**示例：`List` ADT**

- **数据**：一个有序的元素序列。
- **操作**：`add(item)`, `get(index)`, `size()`, `remove(index)` 等。
- **行为**：`add` 会在列表末尾增加一个元素，`get(i)` 会返回第 `i` 个元素。

`SLList` (单向链表) 和 `AList` (动态数组) 是 `List` ADT 的两种**具体实现 (Concrete Implementations)**。它们都提供了 `List` ADT 所要求的所有操作，但内部的数据结构和算法完全不同。

**在 Java 中，`interface` 是定义 ADT 的完美工具。** 我们可以创建一个 `List61B` 接口来定义 `List` ADT，然后让 `SLList` 和 `AList` 都去 `implements` 这个接口。

```java
// 定义 List ADT
public interface List61B<Item> {
    void addLast(Item x);
    Item get(int i);
    int size();
    // ...
}

// List ADT 的一种实现
public class AList<Item> implements List61B<Item> {
    // 使用数组来实现
    private Item[] items;
    // ...
}

// List ADT 的另一种实现
public class SLList<Item> implements List61B<Item> {
    // 使用节点和指针来实现
    private class Node { /* ... */ }
    private Node sentinel;
    // ...
}
```

通过这种方式，我们可以编写操作 `List61B` 接口的代码，这些代码可以无缝地与 `AList` 或 `SLList` 的实例一起工作，而无需关心它们的内部实现差异。这就是抽象的力量。



### **总结**

1. **问题**：我们需要编写通用的代码来处理不同类型的数据。
2. **接口继承 (`implements`)**：定义一个 **"is-a"** 的契约，规定一个类**能做什么**。它通过**动态方法选择**让通用代码（如 `max` 方法）能够调用特定类型的具体实现，是实现**多态 (Polymorphism)** 的关键。
3. **实现继承 (`extends`)**：复用父类的代码，子类不仅知道父类**能做什么**，还继承了**怎么做**。主要用于代码复用和建立强类型层级。
4. **`default` 方法**：让接口也能提供代码实现，为接口继承增添了部分实现继承的能力。
5. **ADT (抽象数据类型)**：只关注“是什么”和“能做什么”，不关心“怎么实现”。Java 的 `interface` 是定义 ADT 的理想工具。
6. **最佳实践**：优先使用接口继承来降低耦合，提高灵活性。仅在代码复用和强 "is-a" 关系明确时才使用实现继承。