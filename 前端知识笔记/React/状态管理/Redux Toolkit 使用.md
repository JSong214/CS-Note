### Redux Toolkit 配置与使用指南

这个流程可以分为 **6 个核心步骤**：

1. **安装依赖**
2. **创建 Redux Store**
3. **创建 State "Slice" (状态切片)**
4. **将 Store 提供给 React 应用**
5. **在组件中使用 State 和 Actions**
6. **(进阶) 处理异步逻辑**

------

#### 步骤 1: 安装依赖

首先，你需要安装 Redux Toolkit 和 React-Redux 这两个核心库。

```bash
# 使用 npm
npm install @reduxjs/toolkit react-redux

# 或者使用 yarn
yarn add @reduxjs/toolkit react-redux
```

------


#### 步骤 2: 创建 Redux Store

Store 是你应用中所有状态的唯一存放地。使用 RTK 的 `configureStore` 来创建它。

在你的 `src` 目录下，创建一个 `app` 或 `store` 文件夹，并在其中创建 `store.js`。

**`src/app/store.js`**
```JavaScript
import { configureStore } from '@reduxjs/toolkit';
// 稍后我们会在这里导入 reducer
// import counterReducer from '../features/counter/counterSlice';

export const store = configureStore({
  // reducer 是一个对象，键名将成为 state 对象中的键名
  reducer: {
    // counter: counterReducer,
  },
});
```

**关键点:**

- `configureStore` 接收一个包含 `reducer` 属性的对象。
- 它会自动配置好 Redux DevTools 插件和一些有用的中间件 (比如 `redux-thunk` 用于处理异步)。

------


#### 步骤 3: 创建 State "Slice" (状态切片)

"Slice" 是应用中单个功能的 state、reducers 和 actions 的集合。这是 RTK 的核心。

在 `src` 目录下，创建一个 `features` 文件夹来存放不同功能的 slice。我们以一个计数器为例。

**`src/features/counter/counterSlice.js`**
```JavaScript
import { createSlice } from '@reduxjs/toolkit';

// 1. 定义初始状态
const initialState = {
  value: 0,
};

// 2. 创建一个 Slice
export const counterSlice = createSlice({
  name: 'counter', // slice 的名称，会用作 action type 的前缀
  initialState,
  // reducers 字段定义了 state 的更新逻辑
  reducers: {
    // 每个 reducer 函数接收 state 和 action 两个参数
    increment: (state) => {
      // RTK 底层使用 Immer，允许你“直接修改” state
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    // 使用 action.payload 来接收外部传入的数据
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

// 3. 导出自动生成的 Action Creators
export const { increment, decrement, incrementByAmount } = counterSlice.actions;

// 4. 导出 reducer，给 store 使用
export default counterSlice.reducer;
```

**关键点:**

- `createSlice` 会根据 `reducers` 里的函数名，自动生成对应的 Action Creators。
- 你写的看似“直接修改” state 的代码是安全的，Immer 会在背后生成不可变的新状态。

现在，回到 `store.js`，把这个 reducer 添加进去。

**`src/app/store.js` (更新后)**
```JavaScript
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from '../features/counter/counterSlice'; // 导入 reducer

export const store = configureStore({
  reducer: {
    counter: counterReducer, // 添加 slice reducer
  },
});
```

------


#### 步骤 4: 将 Store 提供给 React 应用

为了让你的 React 组件能够访问到 Redux store，你需要使用 `react-redux` 提供的 `<Provider>` 组件，在应用的根部把它包裹起来。

**`src/index.js` (或 `main.jsx`)**
```JavaScript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { store } from './app/store'; // 导入 store
import { Provider } from 'react-redux'; // 导入 Provider

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    {/* 使用 Provider 包裹 App，并将 store 作为 prop 传入 */}
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

------


#### 步骤 5: 在组件中使用 State 和 Actions

现在，你可以在任何组件中通过 Hooks 来与 Redux store 交互了。

- **`useSelector`: 读取 store 中的 state。**
- **`useDispatch`: 获取 `dispatch` 函数来派发 actions。**

**`src/features/counter/Counter.jsx` (示例组件)**

```JavaScript
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import {
  decrement,
  increment,
  incrementByAmount,
} from './counterSlice'; // 导入 actions

export function Counter() {
  // 1. 使用 useSelector 从 store 中读取 state
  const count = useSelector((state) => state.counter.value);
  
  // 2. 使用 useDispatch 获取 dispatch 函数
  const dispatch = useDispatch();

  return (
    <div>
      <div>
        <button
          aria-label="Increment value"
          onClick={() => dispatch(increment())} // 派发 action
        >
          Increment
        </button>
        <span>{count}</span>
        <button
          aria-label="Decrement value"
          onClick={() => dispatch(decrement())} // 派发 action
        >
          Decrement
        </button>
        <button
          onClick={() => dispatch(incrementByAmount(5))} // 派发带 payload 的 action
        >
          Add 5
        </button>
      </div>
    </div>
  );
}
```

------


#### 步骤 6: (进阶) 处理异步逻辑

真实应用中，你经常需要请求 API。RTK 提供了 `createAsyncThunk` 来优雅地处理异步操作。

我们创建一个新的 slice 来获取文章列表。

**`src/features/posts/postsSlice.js`**
```JavaScript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// 1. 创建一个 async thunk 来执行异步操作
export const fetchPosts = createAsyncThunk(
  'posts/fetchPosts', // action type 的前缀
  async () => {
    // 这里的逻辑可以是任何异步操作，比如 API 请求
    const response = await fetch('https://jsonplaceholder.typicode.com/posts?_limit=5');
    const data = await response.json();
    return data; // 这个返回值会成为 fulfilled action 的 payload
  }
);

const postsSlice = createSlice({
  name: 'posts',
  initialState: {
    items: [],
    status: 'idle', // 'idle' | 'loading' | 'succeeded' | 'failed'
    error: null,
  },
  reducers: {
    // 同步 reducers 放在这里
  },
  // extraReducers 用于处理 slice 外部的 actions (比如 async thunk 的生命周期)
  extraReducers: (builder) => {
    builder
      .addCase(fetchPosts.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload;
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  },
});

export default postsSlice.reducer;
```

**在组件中使用异步 Thunk:**

```JavaScript
// PostsListComponent.jsx
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { fetchPosts } from './postsSlice';

function PostsList() {
  const dispatch = useDispatch();
  const posts = useSelector((state) => state.posts.items);
  const postStatus = useSelector((state) => state.posts.status);

  useEffect(() => {
    if (postStatus === 'idle') {
      dispatch(fetchPosts()); // 在组件加载时派发 async thunk
    }
  }, [postStatus, dispatch]);

  if (postStatus === 'loading') return <div>Loading...</div>;

  return (
    <ul>
      {posts.map(post => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}
```

别忘了把 `postsSlice.reducer` 添加到 `store.js` 中！


### 总结

| 步骤  | 文件        | 核心 API/组件                | 目的                       |
| ----- | ----------- | ---------------------------- | -------------------------- |
| **1** | -           | `npm` / `yarn`               | 安装依赖库                 |
| **2** | `store.js`  | `configureStore`             | 创建并配置 Redux Store     |
| **3** | `*Slice.js` | `createSlice`                | 封装状态、Reducer和Action  |
| **4** | `index.js`  | `<Provider>`                 | 将 Store 注入到 React 应用 |
| **5** | `*.jsx`     | `useSelector`, `useDispatch` | 在组件中读写状态           |
| **6** | `*Slice.js` | `createAsyncThunk`           | 处理 API 请求等异步逻辑    |
