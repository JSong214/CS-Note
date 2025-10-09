## Lecture10 迭代器与对象方法



### 1. Java中的Lists和Sets (Lists and Sets in Java)

在Java中，`List`和`Set`是两种常用的集合类型，它们都用于存储一组对象。

- **List (列表):** `List`是一个有序的集合，其中的元素可以重复。你可以通过索引来访问`List`中的元素。Java中`List`的常用实现类是`ArrayList`和`LinkedList`。
- **Set (集合):** `Set`是一个不包含重复元素的集合。它不保证元素的顺序。Java中`Set`的常用实现类是`HashSet`和`TreeSet`。

**重要知识点：**

- **接口和实现 (Interface and Implementation):** 在Java中，`List`和`Set`都是接口（Interfaces），你不能直接实例化它们。你需要实例化它们的实现类（Implementation Classes），例如`ArrayList`或`HashSet`。
- **泛型 (Generics):** `List`和`Set`都使用泛型来指定它们可以存储的元素类型。例如，`List<String>`表示一个只能存储`String`类型对象的`List`。

**代码示例：**

```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        // 创建一个ArrayList来存储整数
        List<Integer> numbers = new ArrayList<>();
        numbers.add(1);
        numbers.add(2);
        numbers.add(2); // List可以包含重复元素
        System.out.println("List: " + numbers);

        // 创建一个HashSet来存储字符串
        Set<String> names = new HashSet<>();
        names.add("Alice");
        names.add("Bob");
        names.add("Alice"); // Set会自动忽略重复元素
        System.out.println("Set: " + names);
    }
}
```



### 2. 异常 (Exceptions)

异常是在程序执行期间发生的、中断正常指令流的事件。Java提供了异常处理机制，让你可以优雅地处理这些错误，而不是让程序崩溃。

**重要知识点：**

- **抛出异常 (Throwing Exceptions):** 你可以使用`throw`关键字在代码中手动抛出异常。这通常用于在方法接收到无效参数或遇到无法处理的错误情况时通知调用者。
- **异常类型 (Exception Types):** Java中有许多内置的异常类型，例如`IllegalArgumentException`（非法参数异常）、`NullPointerException`（空指针异常）等。你也可以创建自己的异常类。
- **try-catch语句块 (try-catch Block):** `try-catch`语句块用于捕获和处理异常。你将可能抛出异常的代码放在`try`块中，并在`catch`块中处理这些异常。

**代码示例：**

```java
public class ArraySet<T> {
    private T[] items;
    private int size;

    public ArraySet() {
        items = (T[]) new Object[100];
        size = 0;
    }

    // 当尝试添加null时，抛出IllegalArgumentException
    public void add(T x) {
        if (x == null) {
            throw new IllegalArgumentException("不能向ArraySet中添加null");
        }
        // ... (添加元素的逻辑)
    }

    public static void main(String[] args) {
        ArraySet<String> set = new ArraySet<>();
        try {
            set.add(null);
        } catch (IllegalArgumentException e) {
            System.out.println("捕获到异常: " + e.getMessage());
        }
    }
}
```



### 3. 迭代 (Iteration)

迭代是遍历集合中所有元素的过程。Java提供了两种主要的迭代机制：`Iterator`接口和增强型`for`循环（for-each loop）。

**重要知识点：**

- **Iterable接口:** `Iterable`接口表示一个对象是可以被迭代的。它只有一个方法`iterator()`，该方法返回一个`Iterator`对象。所有Java的集合类（如`ArrayList`, `HashSet`）都实现了`Iterable`接口。
- **Iterator接口:** `Iterator`接口提供了迭代的具体方法：
  - `hasNext()`: 检查是否还有下一个元素。
  - `next()`: 返回下一个元素，并将迭代器向前移动一位。
  - `remove()`: （可选操作）从集合中移除`next()`方法返回的最后一个元素。
- **增强型for循环:** 这是Java提供的一种语法糖，可以让你更方便地遍历实现了`Iterable`接口的集合或数组。

**代码示例：**

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> fruits = new ArrayList<>();
        fruits.add("Apple");
        fruits.add("Banana");
        fruits.add("Orange");

        // 使用Iterator进行迭代
        Iterator<String> iterator = fruits.iterator();
        while (iterator.hasNext()) {
            String fruit = iterator.next();
            System.out.println(fruit);
        }

        // 使用增强型for循环进行迭代 (更简洁)
        for (String fruit : fruits) {
            System.out.println(fruit);
        }
    }
}
```



### 4. 对象方法 (Object Methods)

在Java中，所有的类都直接或间接地继承自`Object`类。这意味着每个类都继承了`Object`类的方法。其中，`toString()`和`equals()`是两个非常重要且经常需要被重写（override）的方法。

**重要知识点：**

- **`toString()`方法:** 该方法返回对象的字符串表示形式。默认情况下，`Object`类的`toString()`方法返回的是“类名@哈希码”的格式，这通常对我们来说没有太大意义。因此，我们应该在自己的类中重写`toString()`方法，以便返回更有用的信息。
- **`equals()`方法:** 该方法用于比较两个对象是否相等。默认情况下，`Object`类的`equals()`方法比较的是两个对象的内存地址（即它们是否是同一个对象）。在大多数情况下，我们关心的是两个对象的内容是否相等，因此需要重写`equals()`方法。

**代码示例：**

```java
public class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // 重写toString()方法
    @Override
    public String toString() {
        return "(" + x + ", " + y + ")";
    }

    // 重写equals()方法
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }
        Point other = (Point) obj;
        return x == other.x && y == other.y;
    }

    public static void main(String[] args) {
        Point p1 = new Point(1, 2);
        Point p2 = new Point(1, 2);
        Point p3 = new Point(3, 4);

        System.out.println("p1: " + p1); // 调用重写的toString()
        System.out.println("p1.equals(p2): " + p1.equals(p2)); // true
        System.out.println("p1.equals(p3): " + p1.equals(p3)); // false
    }
}
```



### 5. 章节总结 (Chapter Summary)

本章的核心内容围绕着Java的继承机制，以及如何利用它来实现更强大和灵活的代码。主要知识点包括：

- **`List`和`Set`:** 学习了Java中两种基本的集合类型，以及如何使用它们的实现类`ArrayList`和`HashSet`。
- **异常处理:** 了解了如何通过抛出和捕获异常来处理程序中的错误情况，使代码更健壮。
- **迭代:** 掌握了使用`Iterator`和增强型`for`循环来遍历集合中的元素。
- **`Object`类的方法:** 理解了`toString()`和`equals()`方法的重要性，并学会了如何在自定义类中重写它们以满足我们的需求。