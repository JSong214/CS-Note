---
date: 2025-06-19
tags:
  - Vue
  - Router
aliases:
  - 路由
---
### 1.核心概念

| 概念                | 解释                                                         |
| ------------------- | ------------------------------------------------------------ |
| **`router`**        | **路由器实例**。通过 `createRouter` 创建，是整个路由系统的管理者。你需要将它挂载到 Vue 应用实例上。 |
| **`routes`**        | **路由配置数组**。定义了“URL路径”与“Vue组件”之间的映射关系。 |
| **`<router-link>`** | **导航组件**。用于生成一个 `<a>` 标签，让用户可以点击切换路由。 |
| **`<router-view>`** | **出口/占位符组件**。匹配到的路由组件将在这里被渲染。        |

**详细解释：**

想象一下，你的单页面应用（SPA）就是一个大房子。

- **`router` (路由器)** 就是这栋房子的**中央控制系统**。
- **`routes` (路由表)** 就是一张**地图**，上面写着“`/living-room` 对应客厅组件”、“`/bedroom` 对应卧室组件”。
- **`<router-link>` (导航链接)** 就是墙上的**开关或门**，你点击“去卧室”的开关，就会触发导航。
- **`<router-view>` (视图出口)** 就是房子里**真正展示内容的那个空间**。你去哪里，这个空间就展示相应房间的内容。


示例代码 (main.js):

```js
import { createApp } from 'vue'
import { createRouter, createWebHistory } from 'vue-router'
import App from './App.vue'
import Home from './views/Home.vue'
import About from './views/About.vue'

// 1. 定义路由表 (routes)
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
]

// 2. 创建路由器实例 (router)
const router = createRouter({
  history: createWebHistory(), // 使用 History 模式
  routes, // `routes: routes` 的缩写
})

const app = createApp(App)
app.use(router) // 3. 挂载到 Vue 应用
app.mount('#app')
```

示例代码 (App.vue):

```vue
<template>
  <header>
    <nav>
      <router-link to="/">首页</router-link> |
      <router-link to="/about">关于</router-link>
    </nav>
  </header>

  <main>
    <router-view></router-view>
  </main>
</template>
```




### 2.路由配置

Vue Router 的核心配置。

- **路由表 (Routes Array)**: 一个包含多个路由对象的数组。每个路由对象通常包含 `path`、`name` 和 `component`。
  - `path`: 路由的路径，如 `/home`。
  - `component`: 当路由被匹配时需要渲染的组件。
  - `name`: 路由的唯一命名，方便在编程式导航或 `router-link` 中使用。
  - `redirect`: 重定向，例如 `path: '/'` 可以重定向到 `'/home'`。
  - `children`: 用于配置嵌套路由。
- **创建路由实例**: 使用 `createRouter` 函数创建路由实例，并将其提供给 Vue 应用。


示例 (`/router/index.js`):

```js
import { createRouter, createWebHistory } from 'vue-router';
import HomeView from '../views/HomeView.vue';
import AboutView from '../views/AboutView.vue';

// 1. 定义路由表 (routes)
const routes = [
  {
    path: '/',
    name: 'home',
    component: HomeView
  },
  {
    path: '/about',
    name: 'about',
    component: AboutView
  }
];

// 2. 创建 router 实例
const router = createRouter({
  // history 属性用于配置路由模式 (下面会详细讲)
  history: createWebHistory(), 
  routes, // `routes: routes` 的缩写
});

export default router;
```

在 `main.js` 中挂载:

```js
import { createApp } from 'vue';
import App from './App.vue';
import router from './router'; // 引入 router 实例

const app = createApp(App);

app.use(router); // 将 router 实例挂载到应用上

app.mount('#app');
```


#### 嵌套路由

用于构建复杂的页面布局，例如一个用户中心页面，包含“我的资料”、“我的订单”等子页面。

**配置**: 使用 **`children` 属性。**

```js
const routes = [
  {
    path: '/user',
    component: UserLayout, // 父组件，包含 <router-view>
    children: [
      {
        path: 'profile', // 注意这里没有前导 '/'
        component: UserProfile,
      },
      {
        path: 'orders',
        component: UserOrders,
      },
    ],
  },
]
```

- `path` 为 `/user` 的 `UserLayout` 组件**必须包含一个 `<router-view>`**，`UserProfile` 和 `UserOrders` 组件才会被渲染到这个出口中。
- 访问路径是父路径和子路径的拼接：`/user/profile` 和 `/user/orders`。


### 3.导航方式

#### 3.1声明式导航:`RouterLink`

这是用于声明式导航的组件，它会被渲染成一个 `<a>` 标签。

- **作用**: 替代传统的 `<a>` 标签，让我们在不重新加载页面的情况下更改 URL，处理 URL 的变化和组件的渲染。

- **常用属性**:

  - **`to`**: 必填，指定目标路由的路径或一个路由对象。
    - 字符串: `<RouterLink to="/about">About</RouterLink>`
    - 对象: `<RouterLink :to="{ name: 'user', params: { id: 123 } }">User</RouterLink>`

  - `active-class`: 当链接被激活时（即当前路由匹配该链接时）应用的 CSS 类名，默认为 `RouterLink-active`。


#### 3.2编程式导航：`router.push()`

在某些场景下，你需要通过 JavaScript 代码来跳转路由，比如用户登录成功后、提交表单后。



**在组合式 API (Composition API) 中:** 需要使用 `useRouter` hook。

```js
import { useRouter } from 'vue-router'

export default {
  setup() {
    const router = useRouter()

    function login() {
      // 登录逻辑...
      if (success) {
        router.push({ name: 'Dashboard' }); // 使用命名路由跳转，更推荐
      }
    }
    
    return { login }
  }
}
```

**`router.push` 的参数可以是：**

- 字符串路径: `router.push('/home')`
- 对象路径: `router.push({ path: '/home' })`
- **命名路由 (推荐)**: `router.push({ name: 'user', params: { userId: '123' }})`
- 带查询参数: `router.push({ path: '/register', query: { plan: 'private' }})`

**`router.push` vs `router.replace`**

- `router.push`: 会向 history 栈添加一条新记录，用户可以点击浏览器后退按钮返回。
- `router.replace`: 不会添加新记录，而是替换掉当前的 history 记录。例如，在登录页成功登录后，跳转到后台主页，这时应该用 `replace`，因为用户不应该能“后退”到登录页。


### 4.动态路由

 【重要知识点】

当我们需要用同一个组件来渲染不同 ID 的内容时（例如用户详情页 `/users/1`, `/users/2`），就需要动态路由。

#### 4.1定义动态路由

在 `path` 中使用冒号 `:` 来标记一个动态片段（称为 "params"）。

示例 (`/router/index.js`):

```js
// ...
const routes = [
  {
    // :id 就是一个动态片段，可以匹配 /user/alice, /user/123 等
    path: '/user/:id', 
    name: 'user',
    component: () => import('../views/UserView.vue') // 懒加载组件
  }
];
```


#### 4.2在组件中获取动态参数

当路由被匹配时，动态片段的值会暴露在 `$route.params` 对象上。

- **Composition API (推荐)**: 使用 `useRoute` hook。
- **Options API**: 使用 `this.$route`。

示例 (`/views/UserView.vue`):

```js
<template>
  <div>
    <h1>用户详情页</h1>
    <p>当前用户 ID 是: {{ userId }}</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';

// 1. 调用 useRoute() 获取当前路由信息对象
const route = useRoute();

// 2. 从 route.params 中获取动态参数 'id'
const userId = ref(route.params.id);

// 注意：如果从 /user/1 导航到 /user/2，组件会被复用
// onMounted 只会执行一次。需要监听路由变化来更新 userId
// 详见下面的 onBeforeRouteUpdate
</script>
```


#### 4.3响应路由参数的变化

一个常见的场景是，当用户从 `/user/1` 导航到 `/user/2` 时，`UserView.vue` 组件实例会被**复用**，而不是销毁再重建。这意味着 **`setup` 或 `created`/`onMounted` 等生命周期钩子不会再次触发。**

为了在这种情况下响应参数的变化，你需要使用 **`onBeforeRouteUpdate` 组件内守卫**或 `watch`。

##### 使用 `onBeforeRouteUpdate` (推荐):

```vue
<script setup>
import { ref } from 'vue';
import { useRoute, onBeforeRouteUpdate } from 'vue-router';

const route = useRoute();
const userId = ref(route.params.id);

// 添加组件内守卫
onBeforeRouteUpdate((to, from) => {
  // 仅在 id 参数发生变化时更新
  if (to.params.id !== from.params.id) {
    userId.value = to.params.id;
  }
});
</script>
```



### 5.导航守卫

【重要知识点】

导航守卫主要用来通过跳转或取消导航来**保护导航**。例如，检查用户是否登录，如果未登录则重定向到登录页。

所有守卫函数都接收三个参数：

- `to`: 即将要进入的目标路由对象。
- `from`: 当前导航正要离开的路由对象。
- `next`: **(在 Vue Router 4 中已不推荐使用)**。现在守卫的返回值决定了导航的行为。
  - **返回 `true` 或 `undefined`**: 验证通过，导航继续。
  - **返回 `false`**: 取消当前导航。
  - **返回一个路由地址**: `return '/login'` 或 `return { name: 'login' }`，重定向到新的地址。


#### 5.1全局前置守卫 (`router.beforeEach`)

这是最常用的守卫，在**每次路由跳转前都会被调用。**

**使用场景**: 登录验证、权限校验。

示例 (`/router/index.js`):

```js
// ...
const router = createRouter({ ... });

// 全局前置守卫
router.beforeEach((to, from) => {
  const isAuthenticated = localStorage.getItem('token'); // 假设 token 存在 localStorage

  // 如果目标路由需要认证，并且用户未认证
  if (to.meta.requiresAuth && !isAuthenticated) {
    // 重定向到登录页
    return { name: 'login' };
  }

  // 否则，放行
  return true; 
});

// 在路由表中可以这样定义 meta
const routes = [{
    path: '/profile',
    name: 'profile',
    component: Profile,
    meta: { requiresAuth: true } // 添加元信息
}];
```


#### 5.2全局后置钩子 (`router.afterEach`)

与守卫不同，后置钩子**不会**改变导航本身。它在导航**确认**之后被调用。

**使用场景**: 修改页面标题、发送页面分析数据。

示例 (`/router/index.js`):

```js
router.afterEach((to, from) => {
  document.title = to.meta.title || 'My Awesome App';
});
```


#### 5.3路由独享守卫 (`beforeEnter`)

直接在**路由配置上定义**的守卫，**只有在进入该路由时才会被触发。**

示例 (`/router/index.js`):

```js
const routes = [{
  path: '/admin',
  component: AdminPanel,
  beforeEnter: (to, from) => {
    // 检查用户是否有管理员权限
    if (!isAdmin()) {
      return '/'; // 没有权限则返回首页
    }
  }
}];
```


### 6.路由模式

【重要知识点】

Vue Router 提供两种模式来处理 URL，通过 `createRouter` 的 **`history`** 选项进行配置。

#### 6.1Hash 模式 (`createWebHashHistory`)

- **URL 形式**: `https://example.com/#/user/1`
- **原理**: 使用 URL 的 hash 部分 (即 `#` 后面的内容) 来模拟一个完整的 URL。当 hash 改变时，页面不会重新加载。
- 优点:
  - **兼容性好**，无需服务器端特殊配置。
  - 部署简单，可以直接将打包后的 `dist` 文件夹放到任何静态文件服务器上。
- 缺点:
  - URL 中带有 `#`，看起来不那么美观。
  - 对 SEO (搜索引擎优化) 不如 History 模式友好（虽然现代搜索引擎已能很好地处理）。

**配置:**

```js
import { createRouter, createWebHashHistory } from 'vue-router';
const router = createRouter({
  history: createWebHashHistory(),
  routes: [ ... ],
});
```


#### 6.2History 模式 (`createWebHistory`)

- **URL 形式**: `https://example.com/user/1`
- **原理**: 利用 HTML5 History API (`pushState`, `replaceState`) 来完成 URL 跳转而无需重新加载页面。
- 优点:
  - URL 更美观、更“正常”。
  - 对 SEO 更友好。
- 缺点:
  - **需要服务器端配置支持**。因为当用户在浏览器直接访问 `https://example.com/user/1` 或刷新页面时，浏览器会向服务器请求这个路径。如果服务器没有配置将这个请求指向你的 `index.html`，就会返回 404 错误。

**服务器配置 (重要!)**: 你必须配置你的服务器，对于所有未匹配到静态文件的请求，都返回应用的 `index.html` 文件。这样，Vue Router 才能接管路由，并显示正确的页面。

- 配置详情：[服务器配置示例](https://router.vuejs.org/zh/guide/essentials/history-mode.html#服务器配置示例)

**配置:**

```js
import { createRouter, createWebHistory } from 'vue-router';
const router = createRouter({
  history: createWebHistory(), // 使用 History 模式
  routes: [ ... ],
});
```



### 7.其他重要特性

#### 7.1路由元信息 (Route Meta)

在路由配置中定义 `meta` 字段，可以附加任意信息到路由上。

```js
{
  path: '/profile',
  component: Profile,
  meta: { requiresAuth: true, roles: ['admin'] }
}
```

然后在导航守卫中通过 `to.meta.requiresAuth` 来访问，这是一种非常灵活和常用的权限控制方案。


#### 7.2路由懒加载

将不同路由对应的组件分割成不同的代码块（Chunk），只有当用户**实际访问**某个路由时，才会去下载和解析对应组件的代码。

**重点：**将路由的 `component` 选项从一个组件本身，改成一个**返回 `import()` 调用的箭头函数**。

```js
// router/index.js

import { createRouter, createWebHistory } from 'vue-router';

const routes = [
  {
    path: '/',
    name: 'Home',
    // Home 组件通常是首屏，可以不使用懒加载，以获得最快的初始显示
    component: () => import('../views/Home.vue')
  },
  {
    path: '/about',
    name: 'About',
    // 当访问 /about 路由时，才会加载 About.vue 对应的 JS 文件
    component: () => import('../views/About.vue')
  },
  {
    path: '/user',
    name: 'User',
    // 当访问 /user 路由时，才会加载 User.vue 对应的 JS 文件
    component: () => import('../views/User.vue')
  }
];

const router = createRouter({
  history: createWebHistory(),
  routes
});

export default router;
```

