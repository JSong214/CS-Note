---
date: 2025-06-19
tags:
  - Vue
  - 动画
---
### `<Transition>`：单个元素的过渡

`<Transition>` 用于包裹**单个**需要实现进入和离开过渡的元素或组件。当其内部的元素或组件因 `v-if`、`v-show` 或动态组件切换而插入或移除时，它会自动应用 CSS 过渡类名。


#### 核心原理：6个CSS类名

这是 `<Transition>` 工作的基础。Vue 会在过渡的不同阶段，为目标元素动态地添加或移除以下6个类名：

1. `v-enter-from`：进入过渡的**起始状态**。元素被插入 DOM 的一瞬间，在下一帧被移除。
2. `v-enter-active`：进入过渡的**激活状态**。在整个进入阶段应用，定义过渡的持续时间、延迟和曲线。
3. `v-enter-to`：进入过渡的**结束状态**。元素被插入 DOM 后的下一帧，在过渡结束后被移除。
4. `v-leave-from`：离开过渡的**起始状态**。触发离开的一瞬间，在下一帧被移除。
5. `v-leave-active`：离开过渡的**激活状态**。在整个离开阶段应用，定义过渡的持续时间、延迟和曲线。
6. `v-leave-to`：离开过渡的**结束状态**。触发离开后，在过渡结束时添加，元素从 DOM 中移除。



**记忆技巧**：

- `-from`: 起始状态 (opacity: 0, transform: translateX(-20px))
- `-to`: 结束状态 (opacity: 1, transform: translateX(0))
- `-active`: 定义动画过程 (transition: all 0.5s ease;)

示例代码：

假设我们有一个由 `v-if` 控制的 `p` 标签：

```vue
<button @click="show = !show">Toggle</button>
<Transition name="fade">	<!-- 注意name，与CSS类名对应 -->
  <p v-if="show">Hello Vue!</p>
</Transition>
```

对应的 CSS：

```css
/* 进入过渡 */
.fade-enter-from {
  opacity: 0;
  transform: translateY(-20px);
}

.fade-enter-active {
  transition: all 0.5s ease;
}

.fade-enter-to {
  opacity: 1;
  transform: translateY(0);
}

/* 离开过渡 */
.fade-leave-from {
  opacity: 1;
  transform: translateY(0);
}

.fade-leave-active {
  transition: all 0.5s ease;
}

.fade-leave-to {
  opacity: 0;
  transform: translateY(20px);
}
```

**注意**：这里的类名前缀是 `fade-` 而不是 `v-`，因为我们通过 `name="fade"` 属性指定了过渡的名称。如果不提供 `name` 属性，则默认使用 `v-` 作为前缀。


#### 重要属性 (Props)

- **`name`**: `string`
  - 用于**自动生成 CSS 过渡类名。**例如 `name="slide"` 会生成 `.slide-enter-from` 等类名。非常适合用于定义可复用的过渡效果。

- **`appear`**: `boolean`
  - 使过渡在**初始渲染**时也生效。默认值为 `false`。这对于希望页面加载时就有一个动画效果的场景非常有用。它会添加对应的 `appear` 类名，如 `.fade-appear-from`。

- **`mode`**: `'in-out' | 'out-in'`
  - **【极其重要】** 用于控制**多个元素之间切换**时的时序。例如，使用 `v-if` / `v-else-if` / `v-else` 时。
  - **`out-in` (最常用)**: 当前元素先执行离开动画，动画结束后，新元素再执行进入动画。这可以防止两个元素在过渡期间同时存在，避免布局跳动。
  - **`in-out`**: 新元素先执行进入动画，动画结束后，旧元素再执行离开动画。这会导致两个元素在过渡期间有重叠。

  - **示例**:
    ```vue
    <Transition name="fade" mode="out-in">
      <button v-if="on" key="on" @click="on = false">ON</button>
      <button v-else key="off" @click="on = true">OFF</button>
    </Transition>
    ```

- **`duration`**: `{ enter: number, leave: number } | number`
  - **用于显式指定过渡的持续时间 (毫秒)。**Vue 通常会自动嗅探 CSS `transition-duration` 或 `animation-duration`。但在某些情况下（例如，CSS 和 JS 动画混合使用），手动指定可以确保时序的精确性。




### `<TransitionGroup>`：列表的过渡

`<TransitionGroup>` 是 `<Transition>` 的增强版，专门用于处理**列表**（例如由 `v-for` 渲染的多个元素）的进入、离开和**位置变化**。

#### 与 `<Transition>` 的关键区别

1. **包裹多个元素**: 它可以包含多个由 `v-for` 生成的元素。
2. **渲染真实元素**: 默认情况下，它会渲染一个 `<span>` 标签作为包裹容器。可以通过 `tag` 属性更改，例如 `tag="ul"` 会将其渲染为 `<ul>` 标签。如果不想渲染任何包裹元素，可以使用 `tag="template"` (Vue 3.1+)。
3. **`key` 属性是必须的**: 为了能正确追踪每个节点的身份以实现动画，`v-for` 中的每个元素都**必须**有一个唯一的 `:key` 属性。
4. **新增 `v-move` 类**: 这是 `<TransitionGroup>` 的独有特性，用于处理列表中元素位置改变时的动画。
5. **没有 `mode` 属性**: 因为它处理的是列表元素的各自进入和离开，而不是互斥元素的切换。



#### 核心原理：`v-move` 类

当列表中的项目顺序发生改变时，Vue 会应用 `v-move` 类，可以**平滑地将元素从旧位置移动到新位置。**

你只需要在 CSS 中为 `v-move` 类添加一个 `transition` 属性即可。

**示例代码：**

```html
<button @click="shuffle">Shuffle</button>
<TransitionGroup tag="ul" name="list" class="my-list">
  <li v-for="item in items" :key="item.id">
    {{ item.text }}
  </li>
</TransitionGroup>
```

```js
// in setup()
const items = ref([{id: 1, text: 'A'}, {id: 2, text: 'B'}, {id: 3, text: 'C'}]);

function shuffle() {
  // 打乱数组顺序
  items.value = items.value.sort(() => Math.random() - 0.5);
}
```

对应的 CSS:

```css
.my-list {
  position: relative; /* 父容器通常需要相对定位 */
}

/* 进入和离开动画 (与 <Transition> 相同) */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* 关键的 v-move 类，用于处理位置变化 */
.list-move {
  transition: transform 0.8s ease; /* 必须设置 transform 的 transition */
}

/* 确保离开的元素脱离文档流，让位给移动的元素 */
.list-leave-active {
  position: absolute;
}
```

**【重要解释】 `v-move` 和 `.list-leave-active { position: absolute; }`**

- **`.list-move`**: 当你点击 `shuffle` 按钮时，Vue 检测到 `items` 数组的顺序改变了。它会计算出每个元素应该移动到的新位置，并通过 CSS `transform` 属性来移动它们。你为 `.list-move` 添加的 `transition: transform 0.8s ease;` 使得这个 `transform` 变化过程是平滑的动画，而不是瞬间完成。
- **`.list-leave-active { position: absolute; }`**: 当一个元素要离开列表时，如果不将它设置为绝对定位，它会继续占据着原来的空间，直到离开动画结束。这会导致其他需要移动到它位置的元素无法平滑地“滑入”，而是会等待它消失后再跳过去。将其设置为 `absolute` 能让它立刻脱离标准文档流，后面的元素就能无缝地开始它们的 `move` 动画。





### 最佳实践与要点回顾

1. **`key` 是关键**：在 `<Transition>` 中切换同类型元素（如两个 `<button>`）或在 `<TransitionGroup>` 中渲染列表时，务必使用唯一的 `key`。
2. **`out-in` 模式**：在需要切换的两个元素之间，`mode="out-in"` 是最常用也是最能避免布局问题的模式。
3. **`position: absolute` for leaving items**：在 `<TransitionGroup>` 中，为离开中的元素（`.name-leave-active`）设置绝对定位，是实现流畅移动动画的关键。
4. **性能**：避免在大型列表上进行过于复杂的动画。对于非常长的列表，可以考虑结合虚拟滚动库来优化性能。