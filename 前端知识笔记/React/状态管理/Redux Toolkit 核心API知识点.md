---
date: 2025-06-27
tags:
  - React
  - Redux
---
### 核心 API 知识点

#### 1\. `configureStore`：简化 Store 配置

这是你使用 RTK 的第一步。`configureStore` 封装了 `createStore`，但提供了更简洁的配置和强大的默认功能。

**详细解释：**

`configureStore` 接收一个配置对象，其中最重要的属性是 `reducer`。

与 `createStore` 不同，它：

  * **自动合并 Reducers:** 你可以直接传入一个包含了多个 "slice reducer" 的对象，`configureStore` 会自动调用 `combineReducers` 将它们合并成一个根 reducer。
  * **内置常用中间件 (Middleware):** 它默认集成了一些最有用的中间件，最核心的是 **`redux-thunk`** 用于处理异步逻辑，以及在开发模式下的 **`redux-immutable-state-invariant`** 和 **`serializable-state-invariant-middleware`** 用于检测 state 的意外突变和不可序列化的值，帮助你避免常见错误。
  * **自动开启 Redux DevTools Extension:** 无需额外配置，只要浏览器安装了 Redux DevTools 插件，就能直接使用。

**代码示例：**

```javascript
// store.js
import { configureStore } from '@reduxjs/toolkit';
import postsReducer from '../features/posts/postsSlice';
import usersReducer from '../features/users/usersSlice';

export const store = configureStore({
  // 将所有的 slice reducer 在这里汇集
  // key 的名称会成为 state 对象中的 key
  reducer: {
    posts: postsReducer,
    users: usersReducer,
  },
  // middleware 和 devTools 都是自动配置好的，通常无需手动设置
});
```

#### 2\. `createSlice`：核心中的核心 (★★★★★)

`createSlice` 是 RTK 最具革命性的功能。它让你能够将 action types、action creators 和 reducer 的逻辑集中在一个地方，极大地减少了样板代码。

**详细解释：**

`createSlice` 接收一个配置对象，包含三个主要参数：

  * `name` (string): 用作 action type 的前缀，例如 `name: 'posts'`，生成的 action type 就会是 `'posts/increment'`。这能有效避免不同 slice 之间的 action type 命名冲突。
  * `initialState`: 这个 slice 的初始状态。
  * `reducers` (object): 一个对象，**key 是 action 的名称，value 是一个函数 (reducer)**，用于处理对应的 action 和更新 state。

**`createSlice` 的魔法之处：**

1.  **自动生成 Action Creators:** 它会根据 `reducers` 对象中的每个函数名，自动创建同名的 action creator。例如，你定义了 `reducers: { increment: (state) => { ... } }`，RTK 就会自动生成一个 `postsSlice.actions.increment()` 的 action creator。调用它会返回一个带有正确 `type` 和 `payload` 的 action 对象。

2.  **内置 Immer.js，简化 State 更新:** 这是最重要的特性！在 `createSlice` 的 reducer 函数中，你**可以“直接修改” state**。

    ```javascript
    // 看起来像直接修改，但背后是 Immer 在工作
    state.value += 1;
    state.items.push(newItem);
    ```

    RTK 在底层使用了 [Immer.js](https://immerjs.github.io/immer/) 库。Immer 会记录你的“修改”操作，并在背后生成一个正确的、不可变的 state 新副本。这让你摆脱了繁琐的扩展运算符 (`...`)，代码变得极其直观和简洁。

**代码示例：**

```javascript
// features/counter/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  value: 0,
  status: 'idle',
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
  // 定义 reducers，这里的函数就是 state 的更新逻辑
  reducers: {
    // action: increment, reducer: (state) => { ... }
    increment: (state) => {
      // Immer.js 让你能够“直接修改” state
      state.value += 1;
    },
    // action: decrement
    decrement: (state) => {
      state.value -= 1;
    },
    // action: incrementByAmount, 接收 payload
    incrementByAmount: (state, action) => {
      // action.payload 是 action creator 调用时传入的参数
      state.value += action.payload;
    },
  },
});

// 导出自动生成的 action creators
export const { increment, decrement, incrementByAmount } = counterSlice.actions;

// 导出 reducer，给 store 使用
export default counterSlice.reducer;
```

#### 3\. `createAsyncThunk`：官方异步解决方案 (★★★★☆)

在 Redux 中处理 API 请求等异步操作是常见需求。`createAsyncThunk` 是 RTK 提供的用于处理这些异步逻辑的标准模式。

**详细解释：**

`createAsyncThunk` 是一个函数，它会为你创建一个封装了异步逻辑的 "thunk action creator"。当你 dispatch 这个 thunk 时，它会根据你的异步操作（通常是一个 Promise）的状态，自动 dispatch 三种不同生命周期的 action：

  * `pending`: 在异步操作开始时 dispatch。
  * `fulfilled`: 在 Promise 成功解决时 dispatch，`payload` 是 Promise 返回的结果。
  * `rejected`: 在 Promise 被拒绝时 dispatch，`payload` 是错误信息。

**如何使用：**

1.  **创建 Thunk:** 调用 `createAsyncThunk`。

      * **第一个参数 (type prefix):** 一个字符串，用作生成 action types 的前缀，例如 `'posts/fetchPosts'`。
      * **第二个参数 (payloadCreator):** 一个异步函数，它接收 `arg` (调用 thunk 时传入的参数) 和 `thunkAPI` (一个包含 `dispatch`, `getState` 等方法的对象)，并返回一个 Promise。

2.  **在 Slice 中处理 Action:** 你不能在 `createSlice` 的 `reducers` 字段中处理这些 action。必须在 `extraReducers` 字段中处理。`extraReducers` 专门用于处理由 `createSlice` 外部定义的 action，包括其他 slice 的 action 或 `createAsyncThunk` 生成的 action。

**代码示例：**

```javascript
// features/posts/postsSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { fetchPostsAPI } from './postsAPI'; // 假设这是一个请求 API 的函数

// 1. 创建 async thunk
export const fetchPosts = createAsyncThunk(
  'posts/fetchPosts', // Action type 前缀
  async () => { // payloadCreator 异步函数
    const response = await fetchPostsAPI();
    return response.data; // 这个返回值会成为 fulfilled action 的 payload
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
    // 这里放同步的 reducers
  },
  // 2. 在 extraReducers 中处理 async thunk 的生命周期
  extraReducers: (builder) => {
    builder
      .addCase(fetchPosts.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload; // payload 是 API 返回的数据
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  },
});

export default postsSlice.reducer;
```

-----
