---
date: 2025-06-19
tags:
  - Vue
  - Pinia
---
### 核心概念

Pinia 的核心由三个部分组成：`State`, `Getters`, `Actions`。

#### State (状态)

**定义**：等同于组件中的 `data`。它是 Store 的“数据源”，存放着你需要全局管理的状态。

**写法**：必须是一个**返回对象的函数**，以避免在服务端渲染 (SSR) 和多实例之间造成状态污染。

```js
// stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  // state 是一个函数，返回一个对象
  state: () => ({
    count: 0,
    userName: 'Alice'
  }),
})
```


#### Getters (计算属性)

- **定义**：等同于组件中的 `computed`。它们是基于 `state` 派生出的新状态。
- **特性**：Getters 的值会被**缓存**，只有当它依赖的 `state` 发生变化时才会重新计算，有助于性能优化。
- **写法**：一个接收 `state` 作为第一个参数的函数。也可以在函数内部通过 `this` 访问其他的 getter。

```js
// stores/counter.js
export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 5,
    userName: 'Bob'
  }),
    
  getters: {
    // 基础 getter，接收 state
    doubleCount: (state) => state.count * 2,

    // Getter 也可以通过 `this` 访问其他 getter
    // 注意：如果使用 this，必须定义为普通函数，不能是箭头函数
    doubleCountPlusOne() {
      return this.doubleCount + 1;
    },

    // 也可以接收参数，返回一个函数
    addBy: (state) => {
      return (amount) => state.count + amount;
    }
  },
})
```



#### Actions (方法)

- **定义**：等同于组件中的 `methods`。它们是用来**修改 `state`** 的主要途径，并且可以包含任意的业务逻辑。
- **特性**：Actions 可以是**异步的** (`async/await`)。你可以在这里执行 API 请求、处理复杂逻辑等，然后再去修改 `state`。

```js
// stores/counter.js
export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0,
    userData: null
  }),
    
  actions: {
    // 同步 action
    increment() {
      // 在 action 中，通过 `this` 访问 state
      this.count++;
    },
    // 异步 action
    async fetchUserData() {
      try {
        const response = await fetch('https://api.example.com/user');
        const data = await response.json();
        this.userData = data; // 直接修改 state
      } catch (error) {
        console.error('Failed to fetch user data', error);
      }
    }
  }
})
```




### 如何使用 Pinia

#### 1.安装和注册

```bash
npm install pinia
```

在你的 `main.js` (或 `main.ts`) 中创建并挂载 Pinia 实例。

```js
// main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia' // 引入 createPinia
import App from './App.vue'

const app = createApp(App)
const pinia = createPinia() // 创建 Pinia 实例

app.use(pinia) // 将 Pinia 实例挂载到应用上
app.mount('#app')
```


#### 2.定义 Store

创建一个 `stores` 目录，并在其中定义你的 Store 文件，例如 `stores/counter.js`。

- `defineStore` 的第一个参数要求是一个**独一无二的**名字 (`'counter'`)。Pinia 内部用它来连接 Store 和开发者工具。**这个名字，也被用作 *id*， 必须是唯一的**。
- 第二个参数是包含 `state`, `getters`, `actions` 的 Options 对象。

```js
import { defineStore } from 'pinia'

//  `defineStore()` 的返回值的命名是自由的
// 但最好含有 store 的名字，且以 `use` 开头，以 `Store` 结尾。
// (比如 `useUserStore`，`useCartStore`，`useProductStore`)
// 第一个参数是你的应用中 Store 的唯一 ID。
export const useAlertsStore = defineStore('alerts', {
  // 其他配置...
})
```

##### Option Store (简单)

```js
export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0, name: 'Eduardo' }),
  getters: {
    doubleCount: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++
    },
  },
})
```

##### Setup Store (灵活)

```js
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const doubleCount = computed(() => count.value * 2)
  function increment() {
    count.value++
  }

  return { count, doubleCount, increment }
})
```

在 *Setup Store* 中：

- `ref()` 就是 `state` 属性
- `computed()` 就是 `getters`属性
- `function()` 就是 `actions`属性

**必须**在 setup store 中返回(return) **`state` 的所有属性**。


#### 3.在组件中使用 Store

在任何 Vue 组件中，你都可以**通过调用 `use...Store()` 函数来获取 Store 实例。**

```vue
<script setup>
import { useCounterStore } from '@/stores/counter'
import { storeToRefs } from 'pinia'

// 1. 获取 store 实例，可以直接使用
const counterStore = useCounterStore()

// 2. 如果你想从 store 中解构属性，同时保持其响应性，
//    必须使用 `storeToRefs`！
//    这对于 state 和 getters 非常重要。
const { count, doubleCount } = storeToRefs(counterStore)

// 3. Actions 可以直接从 store 实例中解构，因为它们是普通函数
const { increment } = counterStore

</script>

<template>
  <div>
    <p>Current Count: {{ counterStore.count }}</p>

    <p>Double Count: {{ doubleCount }}</p>

    <button @click="increment">Increment</button>
    <button @click="counterStore.increment()">Increment (另一种写法)</button>
  </div>
</template>
```

##### 重要知识点详解: `storeToRefs`

为什么需要 `storeToRefs`？

Pinia 的 `state` 本质上是一个 `reactive` 对象。如果你直接使用 `const { count } = counterStore` 来解构，`count` 会变成一个普通的 number 或 string，从而**失去响应性**。当 `store` 中的 `count` 改变时，你组件中的 `count` 不会更新。

`storeToRefs` 的作用就是将 `store` 中的 `state` 和 `getters` 的每个属性都转换成一个对应的 `.value` 的 `ref` 对象，这样就保证了响应性，让你可以在模板中安全地使用它们。

**简单记：要解构 `state` 或 `getters`，就用 `storeToRefs`。`actions` 则可以直接解构。**


#### 4.修改 State

##### **方式一：直接修改 (`store.count++`)**

这是最直观的方式，非常适合简单的修改。

```js
const counterStore = useCounterStore()
counterStore.count++
```


##### 方式二：通过 `actions` 修改 (最推荐的实践)

将所有业务逻辑都封装在 `actions` 中是最佳实践。这使得逻辑变得**可复用、集中化且更易于测试**。无论是简单的同步修改还是复杂的异步流程，都**推荐放在 `actions` 里。**

```js
// 在 action 中
actions: {
  incrementAndDoSomething() {
    this.count++;
    // ... 执行其他逻辑
  }
}

// 在组件中调用
counterStore.incrementAndDoSomething();
```

#### 5.Store 之间的互相调用

Pinia 的模块化设计让 Store 之间的调用变得极其简单。你可以在一个 Store 的 `action` 或 `getter` 中，直接引入并使用另一个 Store。

```js
// stores/user.js
import { defineStore } from 'pinia'
export const useUserStore = defineStore('user', {
  state: () => ({ isLoggedIn: false, name: 'Guest' }),
})

// stores/cart.js
import { defineStore } from 'pinia'
import { useUserStore } from './user' // 1. 引入 user store

export const useCartStore = defineStore('cart', {
  state: () => ({ items: [] }),
  actions: {
    addToCart(item) {
      const userStore = useUserStore() // 2. 在 action 内部获取 user store 实例

      if (userStore.isLoggedIn) {
        this.items.push(item)
      } else {
        alert('Please log in to add items to your cart.')
      }
    }
  }
})
```

