## Lecture9 子类型多态性，比较器，泛型函数



### 核心知识点总览

1. **子类型多态 (Subtype Polymorphism)**：一个变量可以指向其声明类型的任何子类型的对象，并根据对象的实际类型调用相应的方法。
2. **封装 (Encapsulation)**：隐藏对象的内部状态和实现细节，仅通过公共方法（API）暴露有限的访问和操作。
3. **类型转换 (Casting)**：在继承体系中，将一个对象引用从一种类型显式地转换为另一种类型。
4. **接口与比较 (`Comparable` vs `Comparator`)**: 使用接口来定义对象的“自然排序”或提供多种外部排序规则。
5. **高阶函数与泛型 (Higher-Order & Generic Functions)**: 在 Java 中通过接口和泛型来模拟接受“函数”作为参数或返回“函数”的功能，编写更通用和灵活的代码。

------



### 1. 子类型多态 (Subtype Polymorphism)

这是面向对象编程中最强大、最重要的概念之一。

**核心思想**：允许你用一个父类型的引用（变量）来指向一个子类型的对象。当你通过这个父类型引用调用一个方法时，Java 虚拟机会在运行时（Runtime）确定这个对象的**实际类型**（Dynamic Type），并执行该实际类型所对应的方法。这个过程也称为**动态方法分派 (Dynamic Method Dispatch)**。

**两个关键类型**：

- **静态类型 (Static Type)**：变量在**编译时**被声明的类型。它决定了你能调用哪些方法（只能调用在静态类型中定义的方法）。
- **动态类型 (Dynamic Type)**：变量在**运行时**实际指向的对象的类型。它决定了当一个方法被调用时，**哪个版本**的方法会被执行。

**详细解释与代码示例**：

假设我们有一个动物园，里面有各种动物，它们都会叫，但叫声不同。

```java
// 1. 定义一个父类 Animal
public class Animal {
    public void makeSound() {
        System.out.println("动物发出声音");
    }
}

// 2. 定义两个子类 Dog 和 Cat，它们都重写了(override)父类的方法
public class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("汪汪！");
    }

    public void wagTail() {
        System.out.println("狗狗摇尾巴");
    }
}

public class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("喵喵~");
    }
}

// 3. 在主程序中使用多态
public class Zoo {
    public static void main(String[] args) {
        // aDog 的静态类型是 Animal，动态类型是 Dog
        Animal aDog = new Dog();

        // aCat 的静态类型是 Animal，动态类型是 Cat
        Animal aCat = new Cat();

        // 调用 makeSound 方法
        // 编译时：编译器检查 Animal 类有没有 makeSound() 方法。有，编译通过。
        // 运行时：JVM 发现 aDog 实际指向的是 Dog 对象，所以执行 Dog 类的 makeSound() 方法。
        aDog.makeSound(); // 输出: 汪汪！

        // 同理，aCat 实际指向 Cat 对象，执行 Cat 类的 makeSound() 方法。
        aCat.makeSound(); // 输出: 喵喵~

        // 重要：你不能通过 aDog 引用调用 Dog 类独有的方法
        // aDog.wagTail(); // 这行代码会导致编译错误！
        // 因为 aDog 的静态类型是 Animal，编译器在 Animal 类中找不到 wagTail() 方法。

        // 多态最强大的应用之一：处理集合
        Animal[] animals = new Animal[3];
        animals[0] = new Dog();
        animals[1] = new Cat();
        animals[2] = new Animal();

        for (Animal a : animals) {
            // 无需知道 a 到底是狗、是猫还是普通动物
            // Java 会在运行时自动选择正确的方法执行
            a.makeSound();
        }
        // 输出:
        // 汪汪！
        // 喵喵~
        // 动物发出声音
    }
}
```

**总结**：多态允许我们编写更通用、更灵活的代码。我们可以编写一个处理 `Animal` 的方法，而无需关心将来会添加多少种新的动物子类，只要它们都遵循 `Animal` 的接口（即 `makeSound` 方法）。

------



### 2. 封装 (Encapsulation)

**核心思想**：将数据（实例变量）和操作这些数据的方法（getter/setter等）捆绑在一个类中，并对外部隐藏内部的实现细节。这就像一个“黑匣子”，外部用户只知道它的功能，而不知道它内部是如何工作的。

**实现方式**：

- 使用 `private` 访问修饰符来隐藏实例变量。
- 提供 `public` 的方法（通常称为 getter 和 setter）作为唯一的访问和修改数据的途径。

**详细解释与代码示例**：

我们来设计一个 `BankAccount` 类。我们不希望任何人能随意修改账户余额（比如直接把余额改成负数）。

```java
public class BankAccount {
    // 1. 将余额设为 private，外部无法直接访问
    private double balance;

    public BankAccount(double initialBalance) {
        if (initialBalance >= 0) {
            this.balance = initialBalance;
        } else {
            this.balance = 0;
        }
    }

    // 2. 提供一个 public 的 getter 方法来查询余额
    public double getBalance() {
        return this.balance;
    }

    // 3. 提供 public 的方法来操作余额，并在方法中加入控制逻辑
    public void deposit(double amount) {
        if (amount > 0) {
            this.balance += amount;
            System.out.println("成功存款: " + amount);
        } else {
            System.out.println("存款金额必须为正数！");
        }
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            System.out.println("取款金额必须为正数！");
        } else if (this.balance >= amount) {
            this.balance -= amount;
            System.out.println("成功取款: " + amount);
        } else {
            System.out.println("取款失败，余额不足！");
        }
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount myAccount = new BankAccount(100.0);

        // 无法直接修改余额，这行代码会编译错误
        // myAccount.balance = -5000;

        // 只能通过我们提供的公共方法来操作
        System.out.println("当前余额: " + myAccount.getBalance()); // 100.0
        myAccount.deposit(50.0);    // 成功存款: 50.0
        myAccount.withdraw(200.0);  // 取款失败，余额不足！
        myAccount.withdraw(80.0);   // 成功取款: 80.0
        System.out.println("最终余额: " + myAccount.getBalance()); // 70.0
    }
}
```

**总结**：封装保证了数据的完整性和安全性，让类的设计者可以完全控制其内部状态，同时也降低了类与类之间的耦合度。

------



### 3. 类型转换 (Casting)

**核心思想**：当你需要将一个父类型的引用转换回它实际指向的子类型时，就需要用到类型转换，以便能调用子类型特有的方法。

- **向上转型 (Upcasting)**：从子类型到父类型，总是安全的，通常是隐式发生的。`Animal a = new Dog();`
- **向下转型 (Downcasting)**：从父类型到子类型，有风险，必须显式转换，且可能在运行时抛出 `ClassCastException` 异常。

**详细解释与代码示例**：

延续上面的动物园例子，我们知道一个 `Animal` 引用 `aDog` 实际指向的是 `Dog` 对象，我们想让它摇尾巴。

```java
public class Zoo {
    public static void main(String[] args) {
        Animal aDog = new Dog();

        // 编译错误，因为 Animal 类没有 wagTail 方法
        // aDog.wagTail();

        // 为了安全，先检查 aDog 的动态类型是不是 Dog 或者 Dog 的子类
        if (aDog instanceof Dog) {
            // 2. 向下转型：显式地将 Animal 引用转换为 Dog 引用
            Dog realDog = (Dog) aDog;
            // 3. 现在可以通过 realDog 引用调用 Dog 特有的方法了
            realDog.wagTail(); // 输出: 狗狗摇尾巴
        }

        // 危险的转型
        Animal anotherAnimal = new Animal();
        if (anotherAnimal instanceof Dog) {
            Dog riskyDog = (Dog) anotherAnimal; // 这段代码不会执行
            riskyDog.wagTail();
        } else {
            System.out.println("这个 Animal 不是一只 Dog，不能转型！");
        }
        // 如果没有 instanceof 检查直接转型，如下：
        // Dog riskyDog = (Dog) anotherAnimal; // 会在运行时抛出 ClassCastException
    }
}
```

**总结**：向下转型前，请务必使用 `instanceof` 操作符进行检查，这是良好的编程习惯，可以避免程序崩溃。

------



### 4. `Comparable` 与 `Comparator` 接口

当我们需要对一组对象（比如在一个数组或列表中）进行排序时，Java 需要知道如何比较它们。`Comparable` 和 `Comparator` 就是为此而生的两种标准方式。



#### `Comparable<T>`：内部比较器

- **用途**：定义一个类的**自然排序** (Natural Ordering)。例如，数字的自然排序是从小到大，字符串的自然排序是按字典序。
- **实现**：让需要被比较的类**自己实现 `Comparable<T>` 接口**，并实现 `compareTo` 方法。
- **方法**：`public int compareTo(T other)`
  - 返回负数：`this` 对象小于 `other` 对象。
  - 返回零：`this` 对象等于 `other` 对象。
  - 返回正数：`this` 对象大于 `other` 对象。

**代码示例**：让 `Student` 类可以按年龄进行自然排序。

```java
import java.util.Arrays;

// 让 Student 类实现 Comparable 接口，定义其自然排序（按年龄）
public class Student implements Comparable<Student> {
    public String name;
    public int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return name + "(" + age + ")";
    }

    // 实现 compareTo 方法
    @Override
    public int compareTo(Student other) {
        // this.age - other.age 是一个技巧
        // 如果 this.age < other.age, 结果为负
        // 如果 this.age == other.age, 结果为 0
        // 如果 this.age > other.age, 结果为正
        return this.age - other.age;
    }

    public static void main(String[] args) {
        Student[] students = {
            new Student("Alice", 20),
            new Student("Bob", 18),
            new Student("Charlie", 22)
        };
        // Arrays.sort 可以直接对实现了 Comparable 接口的数组进行排序
        Arrays.sort(students);
        System.out.println(Arrays.toString(students));
        // 输出: [Bob(18), Alice(20), Charlie(22)]
    }
}
```



#### `Comparator<T>`：外部比较器

- **用途**：当一个类没有实现 `Comparable`，或者你想要**多种不同的排序方式**（比如学生除了按年龄排，还想按名字排）时，就可以使用 `Comparator`。
- **实现**：创建一个**单独的类**来实现 `Comparator<T>` 接口，并实现 `compare` 方法。
- **方法**：`public int compare(T o1, T o2)`
  - 规则与 `compareTo` 相同，但比较的是两个参数 `o1` 和 `o2`。

**代码示例**：创建一个按名字排序 `Student` 的 `Comparator`。

```java
import java.util.Arrays;
import java.util.Comparator;

// 创建一个独立的比较器类，按名字排序
class NameComparator implements Comparator<Student> {
    @Override
    public int compare(Student s1, Student s2) {
        return s1.name.compareTo(s2.name); // String 类本身就实现了 Comparable
    }
}

public class School {
    public static void main(String[] args) {
        Student[] students = {
            new Student("Charlie", 22),
            new Student("Alice", 20),
            new Student("Bob", 18)
        };

        // 按自然顺序（年龄）排序
        Arrays.sort(students);
        System.out.println("按年龄排序: " + Arrays.toString(students));

        // 使用我们定义的 NameComparator 按名字排序
        Arrays.sort(students, new NameComparator());
        System.out.println("按名字排序: " + Arrays.toString(students));
        // 输出:
        // 按年龄排序: [Bob(18), Alice(20), Charlie(22)]
        // 按名字排序: [Alice(20), Bob(18), Charlie(22)]
    }
}
```



**总结 `Comparable` vs `Comparator`** 

| 特性   | `Comparable<T>`            | `Comparator<T>`                             |
| ------ | -------------------------- | ------------------------------------------- |
| 定义   | 定义类的**自然排序**       | 定义一种**特定**或**外部**的排序规则        |
| 位置   | 在类**内部**实现           | 在类**外部**的独立类中实现                  |
| 方法   | `compareTo(T other)`       | `compare(T o1, T o2)`                       |
| 灵活性 | 低，一种类只有一种自然排序 | 高，可以为一个类创建多种排序规则            |
| 使用   | `Arrays.sort(objArray)`    | `Arrays.sort(objArray, new MyComparator())` |

------



### 5. 高阶函数 (Higher-Order Functions) 与泛型 (Generics)

**核心思想**：

- **高阶函数**：可以接受函数作为参数，或者返回一个函数的函数。Java 本身不直接支持将函数作为一等公民传来传去，但通过**接口**可以完美地模拟这个行为。一个只包含一个方法的接口，其实例对象就可以被看作是一个“函数”。
- **泛型**：让你的代码（类、接口、方法）可以适用于多种数据类型，从而实现代码复用，同时保证类型安全。

**详细解释与代码示例**：

假设我们有一个 `Maximizer` 类，它的功能是找到一组对象中“最大”的那个。但是“最大”的定义是灵活的。对于狗，可以是体重最大；对于字符串，可以是长度最长。

我们先定义一个通用的 `OurComparator` 接口（和 Java 的 `Comparator` 类似），它就扮演了“函数”的角色，定义了比较两个对象的规则。

```java
// 1. 定义一个泛型接口，它代表一个可以比较两个 T 类型对象的“函数”
public interface OurComparator<T> {
    int compare(T o1, T o2);
}

// 2. 实现这个接口来创建具体的比较“函数”
class DogWeightComparator implements OurComparator<Dog> {
    @Override
    public int compare(Dog d1, Dog d2) {
        // 假设 Dog 类有 getWeight() 方法
        return d1.getWeight() - d2.getWeight();
    }
}

class StringLengthComparator implements OurComparator<String> {
    @Override
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
}

// 3. 编写一个泛型的高阶函数 max()
public class Maximizer {
    // 这是一个泛型方法，<T> 是类型参数声明
    // 它接受一个 T 类型的数组和一个 OurComparator<T> 类型的“函数”作为参数
    public static <T> T max(T[] items, OurComparator<T> comparator) {
        if (items == null || items.length == 0) {
            return null;
        }
        T maxItem = items[0];
        for (int i = 1; i < items.length; i++) {
            // 使用传入的 comparator “函数”来比较对象
            if (comparator.compare(items[i], maxItem) > 0) {
                maxItem = items[i];
            }
        }
        return maxItem;
    }

    public static void main(String[] args) {
        Dog[] dogs = {new Dog("A", 10), new Dog("B", 30), new Dog("C", 20)};
        // 传入 Dog 数组和对应的比较器
        Dog heaviestDog = max(dogs, new DogWeightComparator());
        System.out.println("最重的狗: " + heaviestDog.getName()); // 假设 Dog 有 getName()

        String[] strings = {"apple", "banana", "cat"};
        // 传入 String 数组和对应的比较器
        String longestString = max(strings, new StringLengthComparator());
        System.out.println("最长的字符串: " + longestString); // "banana"
    }
}
```

**代码分析**：

- `public static <T> T max(...)`：这是一个泛型方法。`<T>` 声明了一个类型参数 `T`，这个 `T` 可以在方法的参数列表和返回值中像一个真实类型一样使用。
- `max(T[] items, OurComparator<T> comparator)`：这个方法就是**高阶函数**。它接受一个 `OurComparator` 对象 `comparator` 作为参数，这个对象本质上就是一个“比较函数”。
- 通过这种方式，`max` 方法变得极其通用。它不关心 `T` 到底是什么类型，也不关心比较的逻辑是什么。所有的具体逻辑都被封装在传入的 `comparator` 对象中。这正是高阶函数和泛型结合的强大之处。