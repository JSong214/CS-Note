## Lecture7 Truth断言库



### 核心理念：`assertThat(actual).is...()`

所有断言都始于静态方法 `assertThat()`，它接受你想要测试的实际值（actual value）。然后，你可以链式调用各种方法来声明你对这个值的期望。

为了方便使用，通常会静态导入这个方法：

```java
import static com.google.common.truth.Truth.assertThat;
import static com.google.common.truth.Truth.assertWithMessage; // 用于自定义失败信息
```

------



### 1. 通用对象（Object）断言

这些方法适用于任何对象。

| 方法                                  | 描述                                     | 示例                                                     |
| ------------------------------------- | ---------------------------------------- | -------------------------------------------------------- |
| **`isEqualTo(expected)`**             | 判断两个对象是否 `equals()` 相等。       | `assertThat(response.getCode()).isEqualTo(200);`         |
| **`isNotEqualTo(unexpected)`**        | 判断两个对象是否 `equals()` 不相等。     | `assertThat(user.getRole()).isNotEqualTo("guest");`      |
| **`isSameInstanceAs(expected)`**      | 判断两个引用是否指向同一个对象（`==`）。 | `assertThat(singleton1).isSameInstanceAs(singleton2);`   |
| **`isNotSameInstanceAs(unexpected)`** | 判断两个引用是否指向不同的对象（`!=`）。 | `assertThat(newUser).isNotSameInstanceAs(cachedUser);`   |
| **`isInstanceOf(Class)`**             | 判断对象是否是某个类的实例。             | `assertThat(exception).isInstanceOf(IOException.class);` |
| **`isNotNull()`**                     | 判断对象是否不为 `null`。                | `assertThat(user).isNotNull();`                          |
| **`isNull()`**                        | 判断对象是否为 `null`。                  | `assertThat(optionalUser.orElse(null)).isNull();`        |



------



### 2. 字符串（String）断言

针对 `String` 类型的专用断言。

| 方法                            | 描述                                 | 示例                                       |
| ------------------------------- | ------------------------------------ | ------------------------------------------ |
| **`contains(substring)`**       | 判断字符串是否包含指定的子串。       | `assertThat(message).contains("Error");`   |
| **`doesNotContain(substring)`** | 判断字符串是否不包含指定的子串。     | `assertThat(url).doesNotContain(" ");`     |
| **`startsWith(prefix)`**        | 判断字符串是否以指定的前缀开头。     | `assertThat(filename).startsWith("IMG_");` |
| **`endsWith(suffix)`**          | 判断字符串是否以指定的后缀结尾。     | `assertThat(filename).endsWith(".jpg");`   |
| **`isEmpty()`**                 | 判断字符串是否为空字符串 `""`。      | `assertThat(input).isEmpty();`             |
| **`isNotEmpty()`**              | 判断字符串是否不为空字符串 `""`。    | `assertThat(name).isNotEmpty();`           |
| **`hasLength(length)`**         | 判断字符串的长度。                   | `assertThat(uuid).hasLength(36);`          |
| **`matches(regex)`**            | 判断字符串是否匹配指定的正则表达式。 | `assertThat(email).matches(".+@.+\\..+");` |



------



### 3. 数字（Number）断言

适用于 `Integer`, `Long`, `Double` 等数字类型。

| 方法                       | 描述                               | 示例                                       |
| -------------------------- | ---------------------------------- | ------------------------------------------ |
| **`isGreaterThan(value)`** | 判断是否大于某个值。               | `assertThat(count).isGreaterThan(0);`      |
| **`isLessThan(value)`**    | 判断是否小于某个值。               | `assertThat(temperature).isLessThan(100);` |
| **`isAtLeast(value)`**     | 判断是否大于或等于某个值（`>=`）。 | `assertThat(items.size()).isAtLeast(1);`   |
| **`isAtMost(value)`**      | 判断是否小于或等于某个值（`<=`）。 | `assertThat(retries).isAtMost(3);`         |



对于浮点数 `Double` 和 `Float`，还有一个重要的方法用于处理精度问题： | 方法 | 描述 | 示例 | | :--- | :--- | :--- | | **`isWithin(tolerance).of(expected)`** | 判断浮点数是否在期望值的误差范围内。 | `assertThat(result).isWithin(0.001).of(3.141);` |

------



### 4. 布尔值（Boolean）断言

| 方法            | 描述                       | 示例                                        |
| --------------- | -------------------------- | ------------------------------------------- |
| **`isTrue()`**  | 判断布尔值是否为 `true`。  | `assertThat(service.isEnabled()).isTrue();` |
| **`isFalse()`** | 判断布尔值是否为 `false`。 | `assertThat(user.isLocked()).isFalse();`    |



------



### 5. 集合（Collection / Iterable）断言

这是 Truth 非常强大和常用的部分。

| 方法                                        | 描述                                                         | 示例                                                         |
| ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`isEmpty()`**                             | 判断集合是否为空。                                           | `assertThat(errors).isEmpty();`                              |
| **`isNotEmpty()`**                          | 判断集合是否不为空。                                         | `assertThat(results).isNotEmpty();`                          |
| **`hasSize(size)`**                         | 判断集合的大小。                                             | `assertThat(users).hasSize(3);`                              |
| **`contains(element)`**                     | 判断集合是否包含某个元素。                                   | `assertThat(names).contains("Alice");`                       |
| **`doesNotContain(element)`**               | 判断集合是否不包含某个元素。                                 | `assertThat(permissions).doesNotContain("ADMIN");`           |
| **`containsAnyOf(e1, e2, ...)`**            | 判断集合是否包含给定元素中的至少一个。                       | `assertThat(tags).containsAnyOf("java", "python");`          |
| **`containsExactly(e1, e2, ...)`**          | 判断集合是否**有且仅有**给定的这些元素，**顺序不限**。       | `assertThat(ids).containsExactly(1L, 2L, 3L);`               |
| **`containsExactlyElementsIn(collection)`** | 功能同上，只是参数换成了另一个集合。                         | `List<Long> expected = List.of(1L, 2L, 3L); assertThat(ids).containsExactlyElementsIn(expected);` |
| **`... .inOrder()`**                        | 与 `containsExactly` 或 `containsExactlyElementsIn` 配合，要求**顺序也必须一致**。 | `assertThat(names).containsExactly("Bob", "Charlie").inOrder();` |
| **`isInOrder()`**                           | 如果集合元素是 `Comparable` 的，判断它们是否按自然顺序排列。 | `assertThat(numbers).isInOrder();`                           |



------



### 6. Map 断言

| 方法                             | 描述                            | 示例                                                         |
| -------------------------------- | ------------------------------- | ------------------------------------------------------------ |
| **`isEmpty()` / `isNotEmpty()`** | 判断 Map 是否为空。             | `assertThat(headers).isNotEmpty();`                          |
| **`hasSize(size)`**              | 判断 Map 的大小。               | `assertThat(params).hasSize(2);`                             |
| **`containsKey(key)`**           | 判断 Map 是否包含指定的键。     | `assertThat(config).containsKey("timeout");`                 |
| **`doesNotContainKey(key)`**     | 判断 Map 是否不包含指定的键。   | `assertThat(config).doesNotContainKey("debug");`             |
| **`containsEntry(key, value)`**  | 判断 Map 是否包含指定的键值对。 | `assertThat(headers).containsEntry("Content-Type", "application/json");` |



------



### 7. 异常（Throwable）断言

Truth 没有直接的 `assertThat(throwable)` 方法，而是推荐使用 `assertThrows()` 结合 Lambda 表达式，这与 JUnit 5 的方式类似。

```java
import static com.google.common.truth.Truth.assertThat;
import static org.junit.Assert.assertThrows; // 通常使用 JUnit 的 assertThrows

// ...

@Test
public void testDivisionByZero() {
    // 1. 执行一个会抛出异常的 Lambda 表达式
    ArithmeticException thrown = assertThrows(
        ArithmeticException.class,
        () -> { int result = 1 / 0; }
    );

    // 2. 使用 Truth 对捕获到的异常进行断言
    assertThat(thrown).isInstanceOf(ArithmeticException.class);
    assertThat(thrown).hasMessageThat().contains("/ by zero");
}
```

注意 `hasMessageThat()` 会返回一个 `StringSubject`，让你能继续使用所有字符串断言方法。

------



### 8. `Optional` 断言 (需要 `truth-java8-extension`)

| 方法                  | 描述                                   | 示例                                               |
| --------------------- | -------------------------------------- | -------------------------------------------------- |
| **`isPresent()`**     | 判断 `Optional` 包含一个非 `null` 值。 | `assertThat(userOptional).isPresent();`            |
| **`isEmpty()`**       | 判断 `Optional` 为空。                 | `assertThat(emptyOptional).isEmpty();`             |
| **`hasValue(value)`** | 判断 `Optional` 包含一个特定的值。     | `assertThat(userOptional).hasValue(expectedUser);` |