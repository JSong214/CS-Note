---
date: 2025-06-18
tags:
  - Vue
aliases:
  - 插槽
---
## 插槽 (Slots)

### 什么是插槽？

插槽是 Vue 组件中的一个“内容占位符”，它允许父组件在使用子组件时，向子组件的特定区域“插入”自定义的 HTML 内容。

**详细解释：** 想象你做了一个通用的 `Card` 卡片组件。这个卡片可能有固定的边框、阴影样式，但里面的内容是多变的。有时你想放一张图片，有时想放一段文字和按钮。

如果没有插槽，你可能需要用 `props` 传递文本、图片URL等，然后在 `Card` 组件内部用 `v-if` 判断来显示不同的内容。这样做非常笨拙，且扩展性极差。

**插槽的核心价值**就在于**解耦**：

- **子组件 (如 `Card`)**：负责提供**布局和结构框架**（“坑”）。
- **父组件 (使用 `Card` 的地方)**：负责提供**具体内容**（“萝卜”）。

![image-20250227172805325](https://pic-go-image-1363881366.cos.ap-chengdu.myqcloud.com/Image/image-20250227172805325.png)

#### 匿名插槽

当一个组件只有一个需要被外部内容填充的区域时，就可以使用匿名插槽。

**知识点：**

- **子组件定义**：在子组件模板中使用 `<slot></slot>` 标签来定义插槽的位置。
- **父组件使用**：在父组件中，直接将内容写在子组件标签的内部即可。
- **插槽的默认内容**：可以在子组件的 `<slot>` 标签内放置一些默认内容。如果父组件没有提供任何内容，那么这个默认内容就会被显示。

**示例：**

子组件：`FancyButton.vue`

```vue
<template>
  <button class="fancy-btn">
    <slot>
      默认按钮文本 
    </slot>
  </button>
</template>

<style>
.fancy-btn {
  padding: 8px 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #f0f0f0;
}
</style>
```

父组件：`App.vue`

```vue
<template>
  <FancyButton>
    <span>✅ 点击我</span>	<!-- 使用插槽 -->
  </FancyButton>

  <FancyButton></FancyButton>	<!-- 不使用插槽，显示默认内容 -->
</template>

<script setup>
import FancyButton from './FancyButton.vue';
</script>
```

**渲染结果：** 第一个按钮显示 “✅ 点击我”，第二个按钮显示 “默认按钮文本”。



#### 具名插槽

当一个组件需要多个独立的插槽区域时，就需要为它们命名。

**知识点：**

- **子组件定义**：使用 `<slot>` 的 `name` 属性来定义具名插槽。一个不带 `name` 的 `<slot>` 会被隐式命名为 `default`。
- 父组件使用：
  - **使用 `<template>` 元素，并配合 `v-slot` 指令来指定要插入的插槽名。**
  - `v-slot:` 可以缩写为 `#`。例如 `v-slot:header` 可以**简写为 `#header`**。


**示例：**

子组件：`BaseLayout.vue`

```vue
<template>
  <div class="container">
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot></slot>
    </main>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>

<style>
.container { border: 1px solid gray; padding: 10px; }
header, footer { border-bottom: 1px dashed #ccc; padding-bottom: 5px; margin-bottom: 5px; }
</style>
```

父组件：`App.vue`

```vue
<template>
  <BaseLayout>
    <template v-slot:header>	
      <h1>这里是页头</h1>	<!-- 对应子组件的header slot -->
    </template>

    <p>这是主体内容。</p>	<!-- 对应子组件默认 slot -->
    <p>又一行主体内容。</p>	<!-- 对应子组件默认 slot -->

    <template #footer>
      <p>版权所有 © 2025</p>	<!-- 对应子组件的footer slot -->
    </template>
  </BaseLayout>
</template>

<script setup>
import BaseLayout from './BaseLayout.vue';
</script>
```



#### 作用域插槽 （重点）

这是插槽最强大、也是最重要的功能。

- **核心作用：**当父组件在填充插槽时，**需要用到子组件内部的数据**时，作用域插槽允许子组件在渲染插槽时，**将数据作为“属性”传递给父组件。**
- **知识点：**
  - **子组件传值**：
    - 在子组件的 `<slot>` 标签上，像绑定 `props` 一样绑定数据。
      - 例如 `<slot :user="currentUser" :index="i"></slot>`。
  - **父组件接收**：
    - 在父组件中，使用 `v-slot` 来接收这个数据对象。
      - 例如 `<template v-slot:default="slotProps">`。
    - `slotProps` 是一个对象，包含了子组件传递过来的所有属性 
      - 如 `{ user: ..., index: ... }`。
    - 可以使用 **ES6 解构** 来直接获取想要的属性，这是最常用的方式：
      - `<template #default="{ user, index }">`


**示例：**

子组件：`UserList.vue`

```vue
<template>
  <ul>
    <li v-for="(user, index) in users" :key="user.id">
      <slot name="item" :user="user" :index="index"></slot>
    </li>
  </ul>
</template>

<script setup>
import { ref } from 'vue';

const users = ref([
  { id: 1, name: '张三', email: 'zhangsan@example.com' },
  { id: 2, name: '李四', email: 'lisi@example.com' },
  { id: 3, name: '王五', email: 'wangwu@example.com' },
]);
</script>
```

父组件：`App.vue`

```vue
<template>
  <UserList>
    <template #item="{ user, index }">	<!-- 通过v-slot(#) 获取子组件属性 -->
      <div class="user-profile">
        <span>{{ index + 1 }}. {{ user.name }}</span>
        <small>({{ user.email }})</small>
      </div>
    </template>
  </UserList>
</template>

<script setup>
import UserList from './UserList.vue';
</script>

<style>
.user-profile {
  background-color: #eef;
  padding: 5px;
  margin-bottom: 5px;
}
</style>
```

**分析：**
1. `UserList` 组件拥有并管理 `users` 数据。
2. 它通过 `v-for` 循环，并通过 `<slot name="item" ...>` 将每个 `user` 对象和其 `index` 传递出去。
3. 父组件 `App.vue` 接收到这些数据 (`{ user, index }`)，然后完全自定义了每一项的显示逻辑和样式。
4. 这样就实现了**数据逻辑在子组件，渲染逻辑在父组件**的完美分离。



#### 动态插槽名

如果你需要根据一个变量来决定使用哪个插槽，可以使用动态参数。

```vue
<template>
  <BaseLayout>
    <template v-slot:[dynamicSlotName]>
      ...
    </template>
  </BaseLayout>
</template>

<script setup>
import { ref } from 'vue';
const dynamicSlotName = ref('header');
</script>
```

