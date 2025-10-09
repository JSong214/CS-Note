## Lecture1 实例变量(Instance Variables)



### 什么是实例变量 (Instance Variables)？

**核心定义:** 实例变量是定义在类 (Class) 的内部，但在任何方法、构造器或代码块之外的变量。它们也被称为**非静态字段 (Non-Static Fields)** 或对象的**属性 (Properties/Attributes)**。



### Java 基础核心概念：



#### 1. Java 程序的基本结构：类 (Class)

在Java中，所有的代码都必须存在于 **类 (`Class`)** 的内部。你可以把类看作是构建一个程序的蓝图或模板。

```java
public class HelloWorld{
    // 这是一个特殊的“主”方法，是程序的入口点
    public static void main(String[] args){
        System.out.println("hello world!");
    }
}
```

- **核心要点：**
  - **代码容器**：任何代码，无论是变量还是函数，都必须被一个类所包裹。
  - **语法规则**：类的内容需要放在 `{}` 大括号内。
  - **程序入口**：为了让程序能够独立运行，必须包含一个格式为 `public static void main(String[] args)` 的 `main` 方法。





#### 2. 类的构成元素：方法 (Method) 与变量 (Variable)

##### **A. 方法 (Methods) - 定义行为**

方法（Method）是定义在类中的**函数**，它封装了一系列操作指令，用来完成特定的功能。在Java中，由于**所有函数都必须属于一个类**，所以我们通常直接称之为**“方法”**。

```java
public class LargerDemo{
     /**
     * 这是一个方法，用于比较两个整数并返回较大的一个。
     * @param x 第一个整数
     * @param y 第二个整数
     * @return 两者中较大的整数
     */
    public static int larger(int x, int y){
        if (x>y){
            return x;
        }
        return y;
    }
    public static void main(String[] args) {
        // 在 main 方法中调用 larger 方法
        System.out.println(larger(10,20));
    }
}
```

**核心要点：**

- **功能封装**：方法是实现特定功能的代码块。
- **明确的输入与输出**：
  - 方法的所有参数都必须声明类型（如 `int x`）。
  - 方法必须声明一个返回值的类型（如 `int`）。如果不需要返回值，则使用 `void`。
  - Java中的方法只能返回一个值。
- **静态方法**：使用 `public static` 声明的方法属于类本身，而不是类的某个具体实例。



##### B. 实例变量 (Instance Variables) - 定义状态

**核心定义：** 实例变量是**直接定义在类中，但在任何方法之外的变量**。它们代表了类的**属性（Properties/Attributes）或状态，每个类的实例（对象）都会拥有一套独立的实例变量。它们也被称为非静态字段 (Non-Static Fields)**。

```java
public class Dog {
    // 下面这些都是实例变量
    String name;
    int age;
    String breed;

    // 这是一个方法
    void bark() {
        System.out.println("Woof!");
    }
}
```

**与方法的区别：**

- **位置**：实例变量在类级别声明，而方法内部声明的是**局部变量**。
- **作用**：实例变量定义了“是什么”（对象的状态），而方法定义了“能做什么”（对象的行为）。
- **生命周期**：实例变量随着对象的创建而创建，随着对象的销毁而销毁。



### 编译与解释

![编译器与解释器](图片/编译器与解释器.png)

#### **第一阶段：编译 (Compilation)**

1. **起点: `Hello.java` 文件**
   - 这是一个我们用Java语言编写的**源文件 (Source File)**。它是人类可读的文本文件，包含了程序的逻辑、变量、方法等。
2. **工具: `javac` (Java Compiler)**
   - `javac` 是Java开发工具包(JDK)中提供的**编译器**。
   - 它的工作是读取 `.java` 源文件，并对其进行词法分析、语法分析和语义分析，检查代码是否符合Java语言规范。例如，它会检查你是否忘记了分号、变量类型是否正确等。
3. **产物: `Hello.class` 文件**
   - 如果源代码没有语法错误，`javac` 编译器就会将 `Hello.java` 文件**编译**成 `Hello.class` 文件。
   - 这个 `.class` 文件包含的不是可以直接在电脑硬件上运行的机器码，而是一种特殊的、平台无关的中间代码，我们称之为 **“Java字节码” (Bytecode)**。

**小结：** 编译阶段就是将程序员编写的 `.java` 源文件，通过 `javac` 编译器，转换成JVM可以理解的 `.class` 字节码文件的过程。



#### **第二阶段：解释与运行 (Interpretation & Execution)**

1. **起点: `Hello.class` 文件**
   - 现在我们有了平台无关的字节码文件，准备让它运行起来。
2. **工具: `java` (Java Interpreter / JVM)**
   - `java` 命令启动了 **Java虚拟机 (Java Virtual Machine, JVM)**。图中的 "Interpreter" (解释器) 实际上就是JVM最核心的一部分。
   - JVM是一个虚拟的计算机，它会在你的操作系统（如Windows, macOS, Linux）之上创建一个运行环境。
3. **过程: 解释执行**
   - JVM会加载 `Hello.class` 字节码文件。
   - 然后，JVM中的**解释器**会逐行读取字节码指令，并将其**翻译**成当前计算机硬件和操作系统能够理解的**本地机器码 (Native Machine Code)**，然后交给CPU去执行。
   - （补充知识：为了提高性能，现代的JVM还包含了**JIT编译器**（Just-In-Time Compiler，即时编译器），它可以将频繁执行的热点代码直接编译成高效的本地机器码并缓存起来，后续执行时就无需再解释，大大提升了运行速度。）
4. **产物: "stuff happens"**
   - 这代表程序最终运行起来，执行了你在代码中定义的逻辑，比如在控制台打印 "Hello, World!"，打开一个窗口，或者进行复杂的计算等。