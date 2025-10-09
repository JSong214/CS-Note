---
date: 2025-06-27
tags:
  - React
  - Router
---
## React Router v6 核心知识点梳理

React Router 是 React 生态中最主流的路由管理库。它通过管理 URL，让你的单页面应用（SPA）能够模拟多页应用的用户体验，实现不同页面（组件）之间的切换，而无需刷新整个网页。v6 版本相比之前的版本，API 更加精简和强大，全面拥抱了 Hooks。

### 第一部分：核心概念与基础组件

这是使用 React Router 必须掌握的基础。

#### 1\. 什么是前端路由？

在传统的网站（多页应用 MPA）中，你点击一个链接，浏览器会向服务器发送一个新的请求，服务器返回一个新的 HTML 文档。

而在单页面应用（SPA）中，整个应用只有一个主 HTML 文件。当 URL 发生变化时，并不会向服务器请求新的页面。而是由前端的 JavaScript（也就是 React Router）监听到 URL 的变化，然后根据不同的 URL，动态地卸载和挂载不同的 React 组件，从而模拟出页面切换的效果。

  * **优点**：用户体验好，页面切换快，没有白屏等待，服务器压力小。
  * **React Router 的角色**：就是这个“监听和管理”URL 与组件对应关系的核心工具。

#### 2\. 主要组件 (The Main Components)

##### **a. `<BrowserRouter>`**

这是路由的根组件。你需要用它包裹你的整个应用或者应用中需要路由管理的部分。

  * **作用**：它使用 HTML5 History API (`pushState`, `replaceState`, `popstate` 事件) 来保持你的 UI 和 URL 的同步。在浏览器中，它会让你拥有像 `example.com/users/123` 这样清晰的 URL。
  * **用法**：通常在你的应用的入口文件（如 `index.js` 或 `App.js`）中，包裹住根组件 `<App />`。

```jsx
// index.js
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
);
```

##### **b. `<Routes>` 和 `<Route>`**

这两个组件是定义路由映射关系的核心。
  * **`<Routes>`**: 它的作用是包裹一组 `<Route>`。当 URL 变化时，它会查找其下所有子 `<Route>`，并渲染第一个与当前 URL 路径匹配的那个。可以把它想象成一个 `if-else if-else` 语句块。
  * **`<Route>`**: 定义了“当 URL 是这个路径时，渲染这个组件”的规则。
      * **`path` prop**: 字符串，定义了路由的路径。
      * **`element` prop**: React 元素（使用 JSX），定义了当路径匹配时要渲染的组件。这是 v6 的标准写法。

**详细解释：**
在 v5 中，我们使用 `<Switch>` 组件，并且 `component` prop 接收一个组件引用。在 v6 中，`<Switch>` 被 `<Routes>` 替代，`component` prop 被 `element` prop 替代，并且 `element` prop 要求传入一个**JSX 元素**（例如 `<Home />` 而不是 `Home`）。这个改变使得传递 props 给路由组件变得非常直接。

```jsx
// App.js
import { Routes, Route } from 'react-router-dom';
import Home from './pages/Home';
import About from './pages/About';
import Dashboard from './pages/Dashboard';

function App() {
  return (
    <div>
      {/* 其他导航组件等 */}
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </div>
  );
}
```

##### **c. `<Link>` 和 `<NavLink>`**

这两个组件用于生成导航链接。

  * **`<Link>`**: 这是最常用的导航组件。它最终会渲染成一个 `<a>` 标签，但它会阻止浏览器的默认跳转行为。当用户点击它时，它会更新 URL，并由 React Router 处理后续的组件渲染，而不会引起页面刷新。
      * **`to` prop**: 指定了要跳转的目标路径。

  * **`<NavLink>`**: 一个特殊版本的 `<Link>`。它知道自己是否处于“激活”状态。当它指向的 `to` 路径与当前 URL 完全匹配时，它会自动添加一个 `active` class。这对于设置导航菜单中当前选中项的样式非常有用。

**详细解释：**
你可以通过给 `<NavLink>` 的 `className` 或 `style` prop 传入一个函数来定制激活状态的样式。这个函数会接收一个 `{ isActive, isPending }` 对象作为参数。

```jsx
import { Link, NavLink } from 'react-router-dom';

function Navigation() {
  const activeStyle = {
    textDecoration: 'underline',
    color: 'red',
  };

  return (
    <nav>
      <NavLink to="/" style={({ isActive }) => (isActive ? activeStyle : undefined)}>
        Home
      </NavLink>
      <NavLink to="/about" style={({ isActive }) => (isActive ? activeStyle : undefined)}>
        About
      </NavLink>
      <Link to="/dashboard">Dashboard</Link>
    </nav>
  );
}
```

-----

### 第二部分：常用 Hooks

Hooks 是在函数组件中使用 React Router 功能的主要方式。

#### 1\. `useNavigate()`

  * **作用**：提供一个函数，让你可以在代码中进行**编程式导航**。
  * **使用场景**：例如，在用户成功登录后跳转到仪表盘页面，或者提交表单后返回上一页。
  * **用法**：
    ```jsx
    import { useNavigate } from 'react-router-dom';
    
    function LoginButton() {
      const navigate = useNavigate();
    
      function handleLogin() {
        // 假设登录成功...
        navigate('/dashboard'); // 跳转到 /dashboard
        // 或者跳转到上一页
        // navigate(-1);
      }
    
      return <button onClick={handleLogin}>Login</button>;
    }
    ```

#### 2\. `useParams()`

  * **作用**：获取 URL 中的**动态参数**。
  * **使用场景**：当你的路由路径中包含动态部分时，例如 `/users/:userId`，你可以用它来获取 `userId` 的具体值。
  * **用法**：
    ```jsx
    // 1. 定义带参数的路由
    // <Route path="/users/:userId" element={<UserProfile />} />
    
    // 2. 在组件中获取参数
    import { useParams } from 'react-router-dom';
    
    function UserProfile() {
      const { userId } = useParams(); // userId 必须和路由中 :后的名字一致
      return <div>User Profile for ID: {userId}</div>;
    }
    ```

#### 3\. `useLocation()`

  * **作用**：获取当前 URL 的详细信息，返回一个 `location` 对象。
  * **`location` 对象包含**：
      * `pathname`: 当前的路径名 (e.g., `/users/123`)
      * `search`: 查询字符串 (e.g., `?name=leo&age=30`)
      * `hash`: URL 中的 hash 值 (e.g., `#section-one`)
      * `state`: 通过 `<Link>` 或 `Maps` 传递过来的 state。
  * **使用场景**：在 Google Analytics 中上报页面浏览，或者根据当前路径执行某些逻辑。

#### 4\. `useSearchParams()`

  * **作用**：专门用来读取和修改 URL 中的查询字符串（query string）。
  * **用法**：它的行为非常像 `useState`。它返回一个数组，第一个值是当前的 `URLSearchParams` 对象，第二个值是更新它的函数。
    ```jsx
    import { useSearchParams } from 'react-router-dom';
    
    function ProductList() {
      const [searchParams, setSearchParams] = useSearchParams();
      const sortBy = searchParams.get('sort'); // 读取 sort 参数
    
      function handleSortChange(e) {
        setSearchParams({ sort: e.target.value }); // 更新 sort 参数
      }
    
      return (
        <div>
          <p>Current sort: {sortBy || 'default'}</p>
          <select onChange={handleSortChange}>
            <option value="price">Price</option>
            <option value="name">Name</option>
          </select>
        </div>
      );
    }
    ```

-----

### 第三部分：进阶与实用技巧

#### 1\. 嵌套路由 (Nested Routes) 与 `<Outlet />`

**详细解释：**
嵌套路由是 React Router 中一个非常强大且常用的功能。它允许你的路由结构和你的组件结构保持一致。例如，一个用户仪表盘页面，它有一个固定的侧边栏，但右侧的内容区域根据子路径（如 `/dashboard/profile` 或 `/dashboard/settings`）变化。

  * **实现方式**：
    1.  在 `<Route>` 组件内部嵌套其他的 `<Route>`。
    2.  在父路由的组件中，使用 `<Outlet />` 组件作为占位符，告诉 React Router 在哪里渲染子路由匹配的组件。

<!-- end list -->

```jsx
// 1. 定义嵌套路由
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<DashboardHome />} /> {/* 索引路由，见下文 */}
  <Route path="profile" element={<UserProfile />} />
  <Route path="settings" element={<Settings />} />
</Route>

// 2. 在父路由组件中使用 <Outlet />
import { Outlet, Link } from 'react-router-dom';

function DashboardLayout() {
  return (
    <div className="dashboard">
      <nav className="sidebar">
        <Link to="profile">Profile</Link>
        <Link to="settings">Settings</Link>
      </nav>
      <main className="content">
        {/* 子路由组件将在这里被渲染 */}
        <Outlet />
      </main>
    </div>
  );
}
```

#### 2\. 索引路由 (Index Routes)

  * **作用**：用于渲染父路由下的默认子组件。当用户访问父路由的路径（例如 `/dashboard`），但没有匹配任何子路由时，索引路由就会被渲染。

  * **用法**：在嵌套路由中，创建一个没有 `path` prop，但有 `index` prop 的 `<Route>`。

    ```jsx
    <Route path="/dashboard" element={<DashboardLayout />}>
      {/* 当URL是/dashboard时，渲染DashboardHome组件 */}
      <Route index element={<DashboardHome />} />
      <Route path="profile" element={<UserProfile />} />
    </Route>
    ```

#### 3\. 404 / 未匹配路由 (Not Found Route)

  * **作用**：当用户访问的 URL 无法匹配你定义的任何路由时，显示一个“404 Not Found”页面。

  * **用法**：在 `<Routes>` 的最后，添加一个 `path` 为 `"*"` 的 `<Route>`。这个通配符会匹配所有未被前面路由捕获的路径。

    ```jsx
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      {/* ... 其他路由 */}
      <Route path="*" element={<NotFoundPage />} />
    </Routes>
    ```

#### 4\. 受保护的路由 (Protected Routes)

**详细解释：**
这是实际项目中必不可少的功能，用于限制未登录用户访问某些页面。

  * **实现思路**：创建一个包装组件（例如 `<ProtectedRoute>`）。这个组件内部检查用户的认证状态。
      * 如果用户已认证，就渲染子组件（通过 `<Outlet />`）。
      * 如果用户未认证，就重定向到登录页面。
  * **v6 中的实现**：使用 `<Navigate>` 组件进行重定向。


```jsx
import React from 'react';
import { Navigate, Outlet } from 'react-router-dom';

// 假设你有一个检查登录状态的 hook 或函数
const useAuth = () => {
  // 实际项目中，这里会检查 token, cookie,或全局 state
  return { isAuthenticated: localStorage.getItem('token') };
};

const ProtectedRoute = () => {
  const { isAuthenticated } = useAuth();
  // replace prop 可以防止用户通过点击后退按钮回到受保护的页面
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" replace />;
};

// 在路由定义中使用
<Route element={<ProtectedRoute />}>
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/settings" element={<Settings />} />
</Route>
```

#### 5\. 懒加载路由 (Lazy Loading)

  * **作用**：代码分割（Code Splitting）。应用初始加载时，只下载必要的代码。当用户访问某个路由时，才去下载那个路由对应的组件代码。这能极大地优化应用的初始加载性能。

  * **用法**：结合 `React.lazy()` 和 `React.Suspense`。

    ```jsx
    import React, { Suspense, lazy } from 'react';
    import { Routes, Route } from 'react-router-dom';
    
    // 使用 React.lazy 动态导入组件
    const Home = lazy(() => import('./pages/Home'));
    const About = lazy(() => import('./pages/About'));
    
    function App() {
      return (
        <Routes>
          {/* 使用 Suspense 包裹，并提供 fallback UI */}
          <Route
            path="/"
            element={
              <Suspense fallback={<div>Loading...</div>}>
                <Home />
              </Suspense>
            }
          />
          <Route
            path="/about"
            element={
              <Suspense fallback={<div>Loading...</div>}>
                <About />
              </Suspense>
            }
          />
        </Routes>
      );
    }
    ```

-----

### 总结

  * **声明式**：v6 的 API 设计非常直观，你的路由结构就是你的组件树结构。
  * **Hooks 优先**：几乎所有功能都通过 Hooks（`useNavigate`, `useParams` 等）暴露出来，与现代 React 开发范式完美融合。
  * **核心组件**：掌握 `<BrowserRouter>`, `<Routes>`, `<Route>`, `<Link>` 是基础。
  * **实用模式**：嵌套路由 (`<Outlet />`)、保护路由和懒加载是构建复杂应用的基石。
