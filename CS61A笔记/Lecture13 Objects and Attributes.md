## Lecture13 Objects and Attributes

本次讲座的核心思想是引入一种新的数据和逻辑组织方式——**面向对象编程**。其主旨在于将**数据**（状态）和操作这些数据的**函数**（行为）捆绑在一起，形成一个独立、完整的“对象”。



### 1. 核心概念：类 (Class) 与对象 (Object/Instance)

这是整个OOP思想的基石。

- **类 (Class)**
  - **定义**：类是一个**蓝图**或**模板**，它定义了一类事物共有的属性（是什么）和方法（能做什么）。
  - **比喻**：如果你想盖房子，那么“房子”的设计图纸就是**类**。它规定了房子应该有几间卧室、几个卫生间、有没有车库等（属性），以及门可以打开、窗户可以关闭等功能（方法）。
  - **代码中**：使用 `class` 关键字来定义。
- **对象 (Object) / 实例 (Instance)**
  - **定义**：对象是根据**类**这个蓝图创建出来的**具体实体**。每个对象都拥有类所定义的属性和方法，但每个对象的属性值可以是独立的。
  - **比喻**：根据同一张设计图纸（类），你可以建造出许多栋具体的房子（对象）。每一栋房子都是一个独立的实例，它们有自己的地址、自己的颜色、住着不同的人。
  - **代码中**：通过调用类来创建，这个过程称为**实例化 (Instantiation)**。例如 `my_house = House()`。

**总结**：**类是抽象的定义，对象是具体的实现。**

------



### 2. 【重点】创建和使用对象



#### 2.1 `class` 语句

这是定义一个类的语法。

```python
class Account:
    # class body
    ...
```

这里我们定义了一个名为 `Account` 的类。按照Python的风格指南，类名通常采用首字母大写的驼峰命名法（CamelCase）。



#### 2.2 【重点】构造函数 (Constructor): `__init__`

- **作用**：当你从一个类创建（实例化）一个对象时，`__init__` 方法会被**自动调用**。它的主要职责是**初始化**这个新创建对象的状态（即设置其初始属性）。
- **语法**：它是一个特殊的方法，名字固定为 `__init__`（前后都有两个下划线）。

```python
class Account:
    def __init__(self, account_holder):
        # ... a special method that initializes the object
```



#### 2.3 【极其重要】`self` 参数

- **是什么？**：`self` 是类中所有方法的**第一个参数**。它是一个**约定俗成的名称**（你也可以用其他名字，但强烈不建议）。

- **作用**：`self` 代表**对象实例本身**。当你调用一个对象的方法时，Python会自动将这个对象作为第一个参数传递给该方法。

- **详细解释**： 想象一下你有两个银行账户对象：`jim_account` 和 `tom_account`。

  ```python
  jim_account = Account('Jim')
  tom_account = Account('Tom')
  ```

  当你调用 `jim_account.deposit(100)` 时，Python在内部实际上执行的是 `Account.deposit(jim_account, 100)`。那个被自动传入的 `jim_account` 就是方法中的 `self`。 因此，在方法内部，你可以通过 `self` 来访问或修改**当前这个对象**的属性。`self` 是连接方法和对象数据的桥梁。没有它，`deposit` 方法就不知道要给哪个账户存钱。



#### 2.4 实例属性 (Instance Attributes)

- **定义**：属于**单个对象实例**的数据。每个实例的属性值都可能不同。
- **创建**：通常在 `__init__` 方法内部，通过 `self.attribute_name = value` 的语法来创建和初始化。`self.` 这个前缀至关重要，它告诉Python这个属性是属于这个对象实例的。

```python
class Account:
    def __init__(self, account_holder):
        self.balance = 0  # 创建一个实例属性 balance，并初始化为0
        self.holder = account_holder # 创建实例属性 holder
```

在上面的例子中，每个 `Account` 对象都会有自己的 `balance` 和 `holder`。`jim_account.balance` 和 `tom_account.balance` 是相互独立的。



#### 2.5 方法 (Methods)

- **定义**：在 `class`内部定义的函数。它们是对象的**行为**。
- **特点**：方法的第一个参数必须是 `self`，用于接收调用该方法的对象实例。

```python
class Account:
    def __init__(self, account_holder):
        self.balance = 0
        self.holder = account_holder

    def deposit(self, amount):
        """存款"""
        if amount > 0:
            self.balance = self.balance + amount
            return self.balance

    def withdraw(self, amount):
        """取款"""
        if amount > self.balance:
            return 'Insufficient funds'
        self.balance = self.balance - amount
        return self.balance
```

这里的 `deposit` 和 `withdraw` 就是方法。它们通过 `self` 来访问和修改**当前对象**的 `balance` 属性。



#### 2.6 点操作符 (Dot Notation)

点操作符 (`.`) 是访问对象**属性**和调用对象**方法**的统一方式。

- **访问属性**: `object.attribute`
- **调用方法**: `object.method(arguments)`

```python
# 1. 实例化对象
my_account = Account('John')

# 2. 访问属性
print(my_account.holder)  # 输出: John
print(my_account.balance) # 输出: 0

# 3. 调用方法
my_account.deposit(100)
print(my_account.balance) # 输出: 100

my_account.withdraw(30)
print(my_account.balance) # 输出: 70
```

------



### 3. 核心原则：数据抽象 (Data Abstraction)

这是为什么要使用对象的深层原因。

- **定义**：数据抽象是一种隐藏实现细节，只向外部暴露必要接口的编程思想。
- **在OOP中的体现**：
  - **使用者 (User)**：对于一个 `Account` 对象的使用者来说，他只需要知道有 `deposit` 和 `withdraw` 这两个方法可用。他**不需要关心** `balance` 是如何存储和计算的（比如，内部可能是一个整数，也可能是一个更复杂的数据结构）。
  - **实现者 (Implementer)**：类的设计者可以自由地修改内部实现（比如优化`withdraw`的逻辑），只要不改变方法的名称和参数，就不会影响到外部使用者。
- **好处**：
  - **简化复杂性**：使用者只需关注“能做什么”，而不用关心“怎么做的”。
  - **提高代码可维护性**：内部实现的变化被限制在类的内部，不会影响到程序的其他部分。
  - **增强代码复用性**：定义好的类可以在程序的任何地方被重复使用。



### 总结与笔记要点

为了方便你整理，以下是本次讲座最核心的笔记要点：

1. **类 (Class)** vs **对象 (Object)**：蓝图 vs 实体。
2. **创建对象（实例化）**: `my_obj = MyClass()`。
3. **构造函数 `__init__`**:
   - 在实例化时自动调用。
   - 用于初始化对象的属性。
   - 第一个参数总是 `self`。
4. **`self`**:
   - 代表**实例本身**。
   - 在方法内部，用 `self` 来访问该实例的属性和方法 (`self.attribute`, `self.method()`)。
5. **属性 (Attribute)**:
   - 对象的数据（状态）。
   - 通过 `self.attribute = value` 在 `__init__` 中创建。
6. **方法 (Method)**:
   - 对象的行为（功能）。
   - 在 `class` 中定义的函数，第一个参数必须是 `self`。
7. **点操作符 `.`**:
   - 统一的访问工具：`object.attribute` 和 `object.method()`。
8. **数据抽象**:
   - OOP的核心思想：捆绑数据和行为，隐藏实现细节。
   - 使用者只关心**接口**（能调用的方法），不关心**内部实现**。