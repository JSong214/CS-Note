---
date: 2025-06-19
tags:
  - Vue
---
### 核心概念

`<KeepAlive>` 是 Vue 提供的一个**内置抽象组件**。它的核心功能是**缓存组件实例**，而不是在组件切换时直接销毁它们。


###  为什么需要 `<KeepAlive>`？

使用 `<KeepAlive>` 主要有两个目的：

1. **性能优化**：对于一些创建成本较高的组件，或者需要频繁切换的组件，避免反复创建和销毁可以显著提升性能。组件的创建和销毁过程涉及到 DOM 操作、数据初始化、事件监听等，这些都是有开销的。
2. **提升用户体验**：
   - **保留用户输入**：用户在一个复杂的表单中填写了很多信息，切换到其他页面再切回来，如果表单被重置了，体验会非常糟糕。`<KeepAlive>` 可以保留这些输入。
   - **保留浏览位置**：在一个很长的列表中，用户滚动到了中间位置，点击进入详情页，再返回时，希望列表能停留在之前的位置。
   - **避免重复请求数据**：对于一些不常变化的数据，组件被缓存后可以避免每次显示时都重新去请求 API。



### 如何使用 `<KeepAlive>`？

`<KeepAlive>` 的使用非常直接，就是用它包裹住需要被缓存的动态组件。最常见的两种场景是：

#### 场景一：配合动态组件 `<component>`

```vue
<template>
  <button @click="activeComponent = 'ComponentA'">Component A</button>
  <button @click="activeComponent = 'ComponentB'">Component B</button>

  <KeepAlive>
    <component :is="activeComponent" />
  </KeepAlive>
</template>

<script setup>
import { ref, shallowRef } from 'vue'
import ComponentA from './ComponentA.vue'
import ComponentB from './ComponentB.vue'

// 注意：对于动态组件，建议使用 shallowRef 优化性能
const activeComponent = shallowRef(ComponentA)
</script>
```

#### 场景二：配合 `<router-view>` (最常用)

在 Vue Router 中，你可以用 `<KeepAlive>` 包裹 `<router-view>`，来实现对路由组件的缓存。

```vue
<template>
  <router-view v-slot="{ Component }">
    <KeepAlive>
      <component :is="Component" />
    </KeepAlive>
  </router-view>
</template>
```

**注意**：在 Vue Router 4.x (Vue 3) 中，推荐使用这种 `v-slot` 的语法。它能让你更灵活地控制 `<router-view>` 渲染的组件。



### 【重点】生命周期钩子变化

这是 `<KeepAlive>` 最核心的知识点。当一个组件被 `<KeepAlive>` 管理后，它的生命周期会发生变化。

- **常规生命周期**：`created`, `mounted`, `unmounted` (Vue 3) 或 `destroyed` (Vue 2)。
- **`<KeepAlive>` 下的生命周期**：
  - `created` 和 `mounted`：**只会在第一次加载时触发一次**。因为组件实例没有被销毁，所以不会重复创建和挂载。
  - 新增了两个钩子：`activated` 和 `deactivated`。


**详细解释 `activated` 和 `deactivated`：**

- **`activated` (被激活时)**
  - **触发时机**：当被缓存的组件从“失活”状态变为“激活”状态时（即再次被显示到屏幕上时）触发。
  - **首次加载**：组件首次被创建和挂载时，`mounted` 会在 `activated` 之前触发。
  - **用途：**非常适合在这里执行那些“每次组件可见时都需要做”的逻辑。例如：
    - 从服务器获取最新的数据，以保证缓存页面的数据不是过时的。
    - 开启定时器、添加事件监听等。

- **`deactivated` (失活时)**
  - **触发时机**：当组件从“激活”状态变为“失活”状态时（即被隐藏并移入缓存时）触发。
  - 用途：非常适合在这里执行清理工作，与 `activated`对应。例如：
    - 清除在 `activated` 中开启的定时器。
    - 移除在 `activated` 中添加的事件监听，防止内存泄漏。

```vue
<script setup>
import { onMounted, onUnmounted, onActivated, onDeactivated } from 'vue'

onMounted(() => {
  console.log('Component Mounted. (只会执行一次)')
})

onUnmounted(() => {
  // 如果组件最终被移出缓存（例如使用 include/exclude 或 max），这个钩子会被触发
  console.log('Component Unmounted. (组件被销毁时执行)')
})

onActivated(() => {
  console.log('Component Activated. (每次进入该组件时执行)')
  // 在这里可以获取最新数据或开启定时器
})

onDeactivated(() => {
  console.log('Component Deactivated. (每次离开该组件时执行)')
  // 在这里可以清除定时器
})
</script>
```




### 【重点】三大核心 Prop：`include`, `exclude`, `max`

`<KeepAlive>` 提供了三个 Prop，让你能更精细地控制哪些组件被缓存，以及如何缓存。

1. **`include` - 包含**
   - **作用**：只有名称匹配 `include` 规则的组件才会被缓存。
   - **值类型**：字符串、正则表达式、或包含这两者的数组。
   - **匹配目标**：**组件的 `name` 选项**。这一点非常重要！
2. **`exclude` - 排除**
   - **作用**：名称匹配 `exclude` 规则的组件**不会**被缓存。
   - **值类型**：与 `include` 相同。
   - **优先级**：如果一个组件同时被 `include` 和 `exclude` 匹配，`exclude` 的优先级更高。
3. **`max` - 最大缓存数**
   - **作用**：指定最多可以缓存多少个组件实例。
   - **值类型**：数字。
   - **缓存策略**：当缓存的组件数量达到 `max` 的上限时，Vue 会采用 **LRU (Least Recently Used) 算法**，即“最近最少使用”策略，销毁最久没有被访问过的组件实例，为新组件腾出空间。



##### **如何为组件设置 `name` 选项？**

在 Vue 3 的 `<script setup>` 语法中，你需要使用 `defineOptions` 宏：

```vue
<script setup>
// 必须这样定义 name，才能被 include/exclude 识别
defineOptions({
  name: 'MyComponent'
})
</script>
```

使用示例：

```vue
<KeepAlive :include="['Article', 'News']">
  <router-view />
</KeepAlive>

<KeepAlive :exclude="['Profile']">
  <router-view />
</KeepAlive>

<KeepAlive :include="/.*Page$/">
  <router-view />
</KeepAlive>

<KeepAlive :max="10">
  <router-view />
</KeepAlive>
```




### 注意事项与最佳实践

1. **数据刷新问题**：被缓存的组件不会重新执行 `created` 或 `mounted`，因此写在这两个钩子里的数据获取逻辑将不会再次运行。如果需要每次进入页面都刷新数据，请将数据获取逻辑移至 `activated` 钩子中。

2. **动态 `include`/`exclude`**：`include` 和 `exclude` 的值可以是动态的。例如，你可以根据用户的操作来动态决定哪些页面需要被缓存，这提供了极大的灵活性。

3. **`<router-view>` 的 `key`**：如果你希望同一个路由（但参数不同，如 `/user/1` 和 `/user/2`）被当作不同的实例缓存，可以给 `<router-view>` 渲染的组件绑定一个唯一的 `key`。

   ```vue
   <router-view v-slot="{ Component, route }">
     <keep-alive>
       <component :is="Component" :key="route.fullPath" />
     </keep-alive>
   </router-view>
   ```
   
4. **内存管理**：缓存是用内存换取性能。不要滥用 `<KeepAlive>`，对于不需要缓存或者数据实时性要求极高的组件，不要缓存它。使用 `max` 属性可以有效防止内存占用无限增长。
