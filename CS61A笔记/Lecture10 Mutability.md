## Lecture10 Mutability



#### 1. 核心概念：可变性 (Mutability) vs. 不可变性 (Immutability)

这是整个章节的基石。

- **不可变对象 (Immutable Objects)**

  - **定义**: 一旦被创建，其内部的状态（值）就**不能被修改**。
  - **例子**: `int`, `float`, `bool`, `string`, `tuple`。
  - **详细解释**: 当你对一个不可变对象进行操作，看似“修改”了它，实际上 Python 是创建了一个**新的对象**，并将变量名重新绑定到这个新对象上。原来的对象如果没有任何变量名指向它，最终会被垃圾回收机制清理。

  ```python
  x = 10
  # 看起来是修改了 x，但实际上是创建了一个新对象 11，然后让 x 指向它
  # 原来的对象 10 可能还存在，也可能被回收
  x = x + 1
  
  s = "hello"
  # 字符串是不可变的，下面的代码会报错
  # s[0] = 'H'  # TypeError: 'str' object does not support item assignment
  
  # 这不是修改，而是创建了一个全新的字符串 "hello world"，并让 s 指向它
  s = s + " world"
  ```

- **可变对象 (Mutable Objects)**

  - **定义**: 创建之后，其内部的状态（值）**可以被修改**，而对象的身份 (identity) 保持不变。
  - **例子**: `list`, `dictionary`, `set`。
  - **详细解释**: 你可以直接修改对象的内容，比如给列表添加元素、修改字典的键值对。执行这些操作时，你操作的是对象本身，变量名仍然指向同一个内存地址。

  ```python
  my_list = [1, 2, 3]
  print(f"修改前的内存地址: {id(my_list)}")
  
  # 直接在原列表上进行修改
  my_list.append(4)
  my_list[0] = 99
  
  print(my_list)  # 输出: [99, 2, 3, 4]
  print(f"修改后的内存地址: {id(my_list)}") # 内存地址不变
  ```

------



#### 2. 重要知识点一：身份 (Identity) vs. 相等 (Equality) (`is` vs. `==`)

这是理解可变性带来的影响的关键。

- `==` **(相等性, Equality)**

  - **作用**: 比较两个对象的内容（值）是否相等。
  - **好比**: 两本书的内容完全一样。

- `is` **(身份, Identity)**

  - **作用**: 比较两个变量是否指向**同一个内存地址**，即它们是不是**同一个对象**。
  - **好比**: 两只手都指向同一本书。

- **详细解释与示例**:

  ```python
  list_a = [1, 2, 3]
  list_b = [1, 2, 3] # list_b 是一个新创建的列表，只是内容恰好与 list_a 相同
  list_c = list_a     # list_c 指向了 list_a 所指向的同一个对象
  
  # 比较内容
  print(f"list_a == list_b: {list_a == list_b}")  # True, 因为它们的内容相同
  print(f"list_a == list_c: {list_a == list_c}")  # True, 因为它们指向同一个对象，内容自然也相同
  
  # 比较身份（是否是同一个对象）
  print(f"list_a is list_b: {list_a is list_b}")  # False, 因为它们是内存中两个独立的对象
  print(f"list_a is list_c: {list_a is list_c}")  # True, 因为它们指向同一个对象
  ```

  **笔记要点**:

  - 对于可变对象，两个变量可能 `==` 但不 `is`。
  - 如果 `a is b` 为 `True`，那么 `a == b` 一定为 `True`。
  - 反之，如果 `a == b` 为 `True`，`a is b` 不一定为 `True`。

------



#### 3. 重要知识点二：别名 (Aliasing)

当多个变量名指向同一个可变对象时，就产生了别名。这是可变数据类型最需要注意的特性。

- **定义**: 多个名字（变量）引用了同一个对象。

- **后果**: 通过任何一个别名修改对象，都会影响到所有指向该对象的变量。

- **详细解释与示例**: 接上面的例子，`list_a` 和 `list_c` 就是别名关系。

  ```python
  list_a = [1, 2, 3]
  list_c = list_a     # 创建别名
  
  print(f"修改前 list_a: {list_a}") # [1, 2, 3]
  print(f"修改前 list_c: {list_c}") # [1, 2, 3]
  
  # 通过别名 list_c 修改对象
  list_c.append(4)
  
  print(f"修改后 list_a: {list_a}") # [1, 2, 3, 4] <--- list_a 也被改变了！
  print(f"修改后 list_c: {list_c}") # [1, 2, 3, 4]
  ```

  **笔记要点**: 别名是很多隐藏 Bug 的根源。当你把一个可变对象赋值给另一个变量时，你只是创建了一个新的“标签”，而不是对象的“副本”。

------



#### 4. 重要知识点三：函数传参与副作用 (Side Effects)

理解 Python 的传参机制（"Pass by Object Reference" 或 "Pass by Assignment"）对于掌握 Mutability至关重要。

- **机制**: 当你将一个变量作为参数传递给函数时，Python 传递的是**对象的引用**（可以理解为内存地址的副本）。

  - 在函数内部，形参和实参指向**同一个对象**。

- **对于不可变对象**:

  - 在函数内部如果试图“修改”它（例如 `x = x + 1`），实际上是让函数内的形参指向了一个**新的对象**，这**不会影响**函数外部的实参。

  ```python
  def modify_num(x):
      x = x + 1
      print(f"函数内 x 的值: {x}") # 输出 6
  
  num = 5
  modify_num(num)
  print(f"函数外 num 的值: {num}") # 输出 5, 未受影响
  ```

- **对于可变对象**:

  - 在函数内部直接修改对象（例如 `lst.append(item)`），因为形参和实参指向同一个对象，所以函数外部的实参也**会被修改**。这被称为**副作用 (Side Effect)**。

  ```python
  def modify_list(lst):
      lst.append(4)
      print(f"函数内 lst 的值: {lst}") # 输出 [1, 2, 3, 4]
  
  my_data = [1, 2, 3]
  modify_list(my_data)
  print(f"函数外 my_data 的值: {my_data}") # 输出 [1, 2, 3, 4], 被修改了！
  ```

  **笔记要点**:

  - 警惕函数的副作用！一个“好”的函数通常不应该意外地修改传入的可变参数，除非它的功能就是如此（例如 `list.sort()`）。
  - 如果不想让函数修改原始数据，应该在函数内部创建它的**副本**。

------



#### 5. 如何避免意外修改：创建副本 (Copying)

如果你不想因为别名或函数调用而修改原始的可变对象，你需要创建它的副本。

- **浅拷贝 (Shallow Copy)**

  - **方法**: `list.copy()`, `dict.copy()`, `[:]` (切片)
  - **作用**: 创建一个新的顶层容器（如新的 list），但容器内的元素如果是对象，则只复制它们的**引用**。
  - **详细解释**:
    - 如果列表里都是不可变对象（如数字、字符串），浅拷贝效果等同于深拷贝。
    - 但如果列表里包含其他可变对象（如子列表），修改副本中的子列表，依然会影响原始列表中的子列表。

  ```python
  original_list = [[1, 2], [3, 4]]
  shallow_copy = original_list.copy() # 或 original_list[:]
  
  # 1. 修改副本的顶层结构
  shallow_copy.append([5, 6])
  print(f"Original: {original_list}")   # [[1, 2], [3, 4]] -> 未受影响
  print(f"Shallow:  {shallow_copy}")    # [[1, 2], [3, 4], [5, 6]]
  
  # 2. 修改副本中的嵌套对象
  shallow_copy[0].append(99)
  print(f"Original: {original_list}")   # [[1, 2, 99], [3, 4]] -> 受到影响！
  print(f"Shallow:  {shallow_copy}")    # [[1, 2, 99], [3, 4], [5, 6]]
  ```

- **深拷贝 (Deep Copy)**

  - **方法**: 需要导入 `copy` 模块，使用 `copy.deepcopy()`
  - **作用**: 创建一个全新的对象，并**递归地**复制原始对象中包含的所有子对象。副本与原始对象完全独立。
  - **详细解释**:

  ```python
  import copy
  
  original_list = [[1, 2], [3, 4]]
  deep_copy = copy.deepcopy(original_list)
  
  # 修改深拷贝中的嵌套对象
  deep_copy[0].append(99)
  print(f"Original: {original_list}") # [[1, 2], [3, 4]] -> 完全不受影响！
  print(f"Deep:     {deep_copy}")     # [[1, 2, 99], [3, 4]]
  ```

------



### 总结与笔记建议

1. **建立心智模型**: 在脑海中或纸上画环境图。变量名是“标签”，对象是“盒子”。赋值 (`=`) 是贴标签，可变操作 (`.append()`) 是修改盒子里的东西。
2. **核心区分**: 牢记 `is` 和 `==` 的区别，它是检验你是否理解对象与引用的试金石。
3. **警惕别名**: 只要看到 `var_b = var_a` 并且 `var_a` 是一个可变类型，就要立刻警觉：它们是别名关系。
4. **函数副作用**: 传递列表或字典给函数时，要问自己：这个函数会修改它吗？我希望它被修改吗？如果不想，就传递它的副本 (`my_func(my_list.copy())`)。
5. **选择拷贝方式**: 如果你的数据结构只有一层（如 `[1, 2, 3]`），浅拷贝就足够了。如果包含嵌套的可变结构（如 `[[1], [2]]`），则需要使用深拷贝来获得完全的独立。