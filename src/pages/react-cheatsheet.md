# React Cheatsheet

A concise but comprehensive cheat sheet covering React concepts, component patterns, and all built-in Hooks with examples. Includes TypeScript tips and common edge cases.

---

## Quick Overview

- React is a JavaScript library for building user interfaces using components and declarative rendering.
- Core concepts: JSX, components (function + class), props, state, and lifecycle.
- Hooks let you use state and other React features inside function components.

---

## Table of contents

- Overview
- JSX
- Components (Function vs Class)
- Props and State
- Lifecycle (with Hooks)
- Rules of Hooks
- Built-in Hooks (with examples)
  - useState
  - useEffect
  - useRef
  - useMemo
  - useCallback
  - useContext
  - useReducer
  - useLayoutEffect
  - useImperativeHandle
  - useDebugValue
- Custom Hooks
- Performance tips
- Testing & TypeScript
- Common pitfalls and FAQ

---

## JSX

- JSX is syntactic sugar for React.createElement.
- Use parentheses for multi-line JSX.
- Expressions inside JSX use `{}`. Attributes use camelCase (e.g., `onClick`).

Example:

```jsx
function Hello({ name }) {
  return <h1>Hello, {name}</h1>;
}
```

---

## Components

- Function components are the modern standard. They can use Hooks.
- Class components are still supported but Hooks replace most use cases.

Function component:

```jsx
function Counter() {
  const [count, setCount] = React.useState(0);
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  );
}
```

Class component (legacy):

```jsx
class Counter extends React.Component {
  state = { count: 0 };
  render() {
    return (
      <div>
        <p>{this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>+1</button>
      </div>
    );
  }
}
```

---

## Props and State

- Props are read-only inputs passed from parent to child.
- State is local to the component and should be updated via setters (e.g., `setState`, `setCount`).
- Lift state up when multiple components need the same state.

---

## Lifecycle with Hooks

- useEffect(() => { ... }, []) runs after mount (componentDidMount).
- useEffect cleanup runs on unmount or before re-running the effect (componentWillUnmount / componentDidUpdate cleanup).
- For layout-type effects (synchronous after DOM mutations), use `useLayoutEffect`.

---

## Rules of Hooks

1. Only call Hooks at the top level (no loops, conditions, nested functions).
2. Only call Hooks from React function components or custom Hooks.

These rules ensure consistent hook ordering between renders.

---

## Built-in Hooks

Each hook below includes a short explanation, return type, common usage pattern, and an example.

### useState

- Purpose: Add local state to function components.
- Signature: `const [state, setState] = useState(initial)`
- Re-render is scheduled when `setState` is called.

Example:

```jsx
import React from 'react';

function NameInput() {
  const [name, setName] = React.useState('');
  return (
    <input value={name} onChange={e => setName(e.target.value)} placeholder="Your name" />
  );
}
```

Tip: For expensive initial state computations, pass a lazy initializer: `useState(() => expensiveInit())`.

Edge cases:
- Updating state with previous value: `setCount(c => c + 1)` to avoid stale closures.


### useEffect

- Purpose: Run side effects after render (data fetching, subscriptions, DOM changes).
- Signature: `useEffect(() => { effect }, [deps])`
- Cleanup: return a function from the effect to clean up.

Examples:

Data fetch on mount:

```jsx
React.useEffect(() => {
  let mounted = true;
  fetch('/api/data')
    .then(r => r.json())
    .then(data => {
      if (mounted) setData(data);
    });
  return () => {
    mounted = false;
  };
}, []);
```

Effect with dependencies:

```jsx
React.useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);
```

Common pitfalls:
- Missing dependencies (use exhaustive-deps lint rule).
- Putting non-serializable values (functions, objects) into deps without memoization.


### useRef

- Purpose: Hold a mutable value that persists across renders without causing re-renders. Commonly used for DOM refs.
- Signature: `const ref = useRef(initial)`

Examples:

DOM ref:

```jsx
function FocusInput() {
  const inputRef = React.useRef(null);
  React.useEffect(() => {
    inputRef.current?.focus();
  }, []);
  return <input ref={inputRef} />;
}
```

Mutable container:

```jsx
const renderCountRef = React.useRef(0);
renderCountRef.current++;
```

Important: Changing `ref.current` does not trigger a re-render.


### useMemo

- Purpose: Memoize a computed value to avoid expensive recalculation unless dependencies change.
- Signature: `const memoValue = useMemo(() => compute(a, b), [a, b])`

Example:

```jsx
const expensive = React.useMemo(() => heavyComputation(items), [items]);
```

Don't overuse it—only for expensive computations. Remember it affects memory.


### useCallback

- Purpose: Memoize a function so it has stable identity across renders (useful when passing callbacks to optimized children or deps).
- Signature: `const memoFn = useCallback(() => { doSomething(); }, [deps])`

Example:

```jsx
const onClick = React.useCallback(() => setCount(c => c + 1), []);
```

Note: `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.


### useContext

- Purpose: Access a context value created with `React.createContext`.
- Signature: `const value = useContext(MyContext)`

Example:

```jsx
const ThemeContext = React.createContext('light');
function Toolbar() {
  const theme = React.useContext(ThemeContext);
  return <div className={theme}>Toolbar</div>;
}
```

Note: Context consumers re-render when the context value identity changes.


### useReducer

- Purpose: Alternative to `useState` for more complex state logic.
- Signature: `const [state, dispatch] = useReducer(reducer, initialArg, init?)`

Example:

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    default: return state;
  }
}

function Counter() {
  const [state, dispatch] = React.useReducer(reducer, { count: 0 });
  return (
    <>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+1</button>
    </>
  );
}
```

Great for complex updates and when you want to centralize update logic.


### useLayoutEffect

- Purpose: Runs synchronously after all DOM mutations but before the browser paints. Use for measuring DOM layout and synchronously re-rendering.
- Signature: same as `useEffect`.

Example:

```jsx
React.useLayoutEffect(() => {
  const height = ref.current.getBoundingClientRect().height;
  setHeight(height);
}, [deps]);
```

Caution: can cause layout thrashing if misused.


### useImperativeHandle

- Purpose: Customize the instance value that is exposed to parent refs when using `forwardRef`.
- Signature: `useImperativeHandle(ref, () => ({ ... }), [deps])`

Example:

```jsx
const FancyInput = React.forwardRef((props, ref) => {
  const inputRef = React.useRef();
  React.useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
  }));
  return <input ref={inputRef} />;
});
```


### useDebugValue

- Purpose: Display debug values for custom hooks in React DevTools.
- Signature: `useDebugValue(value, formatter?)`

Example:

```jsx
function useFriendStatus(friendId) {
  const status = /* ... */;
  React.useDebugValue(status ? 'online' : 'offline');
  return status;
}
```


## Custom Hooks

- Create reusable logic by extracting into functions that use hooks: `function useSomething(...) { const [s, setS] = useState(); useEffect(...); return s }`.
- Name must start with `use` to follow the rules of hooks.

Example:

```jsx
function useWindowSize() {
  const [size, setSize] = React.useState([0, 0]);
  React.useEffect(() => {
    function onResize() {
      setSize([window.innerWidth, window.innerHeight]);
    }
    window.addEventListener('resize', onResize);
    onResize();
    return () => window.removeEventListener('resize', onResize);
  }, []);
  return size;
}
```


## Performance tips

- Memoize expensive computations with `useMemo`.
- Memoize callbacks with `useCallback` when passing to memoized children.
- Use `React.memo` for pure functional components to avoid unnecessary re-renders.
- Avoid creating new objects/arrays inline as props unless memoized.
- Virtualize long lists (react-window, react-virtualized).


## Testing & TypeScript notes

- Test components with React Testing Library and Jest (or Vitest).
- For TypeScript, prefer typing props and hooks. Example:

```tsx
import React from 'react';

type Props = { initial?: number };
function Counter({ initial = 0 }: Props) {
  const [count, setCount] = React.useState<number>(initial);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

Hooks typing:

```ts
function useToggle(initial = false) {
  const [on, setOn] = React.useState<boolean>(initial);
  const toggle = React.useCallback(() => setOn(o => !o), []);
  return { on, toggle };
}
```


## Common pitfalls & FAQ

- "Why does my effect run twice in dev?" React Strict Mode intentionally mounts, unmounts, and mounts again for components to help detect side effects. Effects should be written to handle mount/unmount cleanly.
- "Stale closures" — prefer updater form `setState(prev => newVal(prev))` when working with previous state in callbacks.
- "Missing effect deps" — include everything referenced in the effect (or use refs/memoization where appropriate).
- Avoid heavy work in render; use memoization and effects.

---

## Further reading

- Official docs: https://reactjs.org
- Hooks RFC and blog posts in the React blog

---

If you'd like, I can also:

- Add a TypeScript-focused version of this cheatsheet.
- Split the cheatsheet into multiple pages (Overview, Hooks, Patterns).
- Add small runnable examples inside the Astro site (CodeSandbox embeds or local examples).

Tell me which of these you'd like next.
