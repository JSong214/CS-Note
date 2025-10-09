---
date: 2025-06-27
tags:
  - React
---
### 1\. 性能优化 Hooks: `useCallback` & `useMemo`

这两个 Hooks 是 React 性能优化的关键，它们都利用了 **记忆化 (Memoization)** 的思想来避免不必要的计算和渲染。

#### `useCallback`

  * **核心思想**: 记忆化一个**函数**。它会返回该函数的一个记忆化版本，只有在依赖项数组中的某个值发生变化时，这个函数才会重新创建。

  * **使用场景**:

    1.  **最主要的场景**: 将一个函数作为 prop 传递给一个被 `React.memo` 包裹的子组件。如果不使用 `useCallback`，父组件每次重新渲染时都会创建一个新的函数实例，导致子组件即使 props 没有实质变化也会重新渲染。
    2.  在 `useEffect` 的依赖项中使用了某个函数时，为了防止 `useEffect` 在每次渲染时都执行，可以用 `useCallback` 包裹那个函数。

  * **语法**:

    ```javascript
    const memoizedCallback = useCallback(
      () => {
        // 你要记忆化的函数逻辑
        doSomething(a, b);
      },
      [a, b], // 依赖项数组
    );
    ```

  * **详细解释**:
    想象一个父组件 `Parent` 和一个子组件 `Child`。`Child` 组件用 `React.memo` 包裹，意味着只有 props 变化时它才重新渲染。

    ```jsx
    // 子组件
    const Child = React.memo(({ onButtonClick }) => {
      console.log('Child component re-rendered');
      return <button onClick={onButtonClick}>Click me</button>;
    });

    // 父组件
    function Parent() {
      const [count, setCount] = useState(0);

      // 每次 Parent 渲染，这个函数都是一个新的实例
      // const handleClick = () => {
      //   console.log('Button clicked');
      // };

      // 使用 useCallback 后，只有在依赖项（这里是空数组）变化时才创建新函数
      const handleClick = useCallback(() => {
        console.log('Button clicked');
      }, []); // 空数组意味着函数永不重新创建

      return (
        <div>
          <p>Count: {count}</p>
          <button onClick={() => setCount(count + 1)}>Increment Count</button>
          <Child onButtonClick={handleClick} />
        </div>
      );
    }
    ```

    在上面的例子中，如果你点击 "Increment Count" 按钮，`Parent` 组件会重新渲染。

      * **不使用 `useCallback`**: `handleClick` 函数会是一个全新的函数。`Child` 组件接收到的 `onButtonClick` prop 变化了（因为是新函数），所以 `Child` 会重新渲染，即使它的功能完全没变。
      * **使用 `useCallback`**: `handleClick` 函数被记忆化了，它的实例保持不变。`Child` 组件接收到的 `onButtonClick` prop 也没变，因此 `React.memo` 会阻止 `Child` 的不必要渲染。

#### `useMemo`

  * **核心思想**: 记忆化一个**值**。它会在组件渲染期间计算一个值，并将其结果缓存起来。只有在依赖项数组中的某个值发生变化时，它才会重新计算。

  * **使用场景**:

    1.  避免在每次渲染时都进行高开销的计算（例如，对一个大数组进行复杂的过滤、排序或计算）。
    2.  当一个对象或数组作为 prop 传递给子组件时，用 `useMemo` 来保证它的引用稳定性，防止子组件不必要的渲染（原理同 `useCallback`）。

  * **语法**:

    ```javascript
    const memoizedValue = useMemo(
      () => computeExpensiveValue(a, b), // 一个返回计算结果的函数
      [a, b], // 依赖项数组
    );
    ```

  * **详细解释**:

    ```jsx
    function DataDisplay({ data, filter }) {
      // 假设 filterData 是一个非常耗时的操作
      const filteredData = useMemo(() => {
        console.log('Calculating filtered data...');
        return data.filter(item => item.includes(filter));
      }, [data, filter]); // 只有 data 或 filter 变化时才重新计算

      return (
        <ul>
          {filteredData.map(item => <li key={item}>{item}</li>)}
        </ul>
      );
    }
    ```

    在这个例子中，如果组件因为其他状态（比如一个无关的 `count`）而重新渲染，`filteredData` 不会重新计算，而是直接使用上次缓存的结果，从而提升了性能。

  * **`useCallback` vs `useMemo`**:

      * `useCallback(fn, deps)` 等价于 `useMemo(() => fn, deps)`。
      * `useCallback` 是函数的记忆化，`useMemo` 是值的记忆化。记住这个核心区别。

-----

### 2\. 高级状态管理: `useReducer`

  * **核心思想**: `useState` 的替代方案，用于更复杂的状态逻辑管理。它借鉴了 Redux 的思想，将状态更新的逻辑从组件中抽离出来。

  * **使用场景**:

    1.  状态逻辑复杂，包含多个子值，或者下一个 state 依赖于前一个 state。
    2.  多个地方需要以可预测的方式更新同一个状态。
    3.  希望将状态更新逻辑与组件分离，便于测试和维护。

  * **语法与组成**:
    它接收一个 `reducer` 函数和 `initialState`，返回当前的 `state` 和一个 `dispatch` 函数。

    ```javascript
    const [state, dispatch] = useReducer(reducer, initialState);
    ```

    1.  **`state`**: 当前的状态值。
    2.  **`dispatch`**: 一个可以触发状态更新的函数。你通过 `dispatch({ type: 'ACTION_TYPE', payload: ... })` 来“派发”一个 action。
    3.  **`reducer`**: 一个纯函数 `(state, action) => newState`。它接收当前 state 和一个 action 对象，然后计算并返回一个新的 state。

  * **详细解释**:
    以一个复杂的计数器为例：

    ```jsx
    // 1. 定义 Reducer 函数
    function counterReducer(state, action) {
      switch (action.type) {
        case 'increment':
          return { count: state.count + 1 };
        case 'decrement':
          return { count: state.count - 1 };
        case 'reset':
          return { count: 0 };
        case 'set':
          return { count: action.payload };
        default:
          throw new Error('Unknown action type');
      }
    }

    // 2. 在组件中使用 useReducer
    function Counter() {
      const initialState = { count: 0 };
      const [state, dispatch] = useReducer(counterReducer, initialState);

      return (
        <div>
          <p>Count: {state.count}</p>
          <button onClick={() => dispatch({ type: 'increment' })}>+</button>
          <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
          <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
          <button onClick={() => dispatch({ type: 'set', payload: 10 })}>Set to 10</button>
        </div>
      );
    }
    ```

- **优点**: 
	- **逻辑分离**：将状态更新的逻辑（reducer）从组件中抽离出来，更易于测试和维护。
	- **更可预测**：所有状态的变更都通过 `dispatch` action 来触发，意图更明确。
	- **性能优化**：在某些场景下，可以给子组件传递 `dispatch` 函数，因为 `dispatch` 函数本身是稳定不变的，可以避免不必要的重渲染。



-----

### 3\. 渲染时序控制: `useLayoutEffect`(了解)

  * **核心思想**: 其函数签名与 `useEffect` 相同，但它在**所有 DOM 变更之后同步调用**。

  * **与 `useEffect` 的区别**:

      * **`useEffect`**: **异步**执行。在浏览器完成绘制**之后**运行，不会阻塞浏览器渲染。这是 99% 情况下你需要的。
      * **`useLayoutEffect`**: **同步**执行。在 React 完成 DOM 更新**之后**、浏览器绘制**之前**运行。它会阻塞浏览器的绘制过程。

  * **使用场景**:
    当你需要在 DOM 更新后立即**读取 DOM 布局**并同步触发重新渲染时。最典型的例子是：为了避免页面闪烁，需要测量 DOM 元素（如获取 tooltip 的位置和大小）并更新其样式。

  * **详细解释**:
    想象一个 Tooltip 组件，它需要在按钮旁边显示。

    1.  组件渲染，Tooltip 在默认位置（如 `top: 0`）。
    2.  `useLayoutEffect` 触发，它会读取按钮的位置和尺寸。
    3.  `useLayoutEffect` 内部立即调用 `setState` 来更新 Tooltip 的位置。
    4.  因为这一切都发生在浏览器绘制之前，用户看不到 Tooltip 从 `top: 0` 闪到正确位置的过程。

    如果用 `useEffect`，用户可能会先看到 Tooltip 在 `top: 0`，然后（在下一次绘制时）跳到正确位置，造成视觉上的闪烁。

    ```jsx
    function TooltipComponent() {
      const [position, setPosition] = useState({ top: 0, left: 0 });
      const buttonRef = useRef(null);

      useLayoutEffect(() => {
        if (buttonRef.current) {
          const { bottom } = buttonRef.current.getBoundingClientRect();
          setPosition({ top: bottom + 5, left: 0 });
        }
      }, []); // 依赖项

      return (
        <div>
          <button ref={buttonRef}>Hover me</button>
          <div style={{ position: 'absolute', ...position }}>
            This is a tooltip
          </div>
        </div>
      );
    }
    ```

  * **注意事项**: 优先使用 `useEffect`。`useLayoutEffect` 会阻塞渲染，滥用会影响性能。
