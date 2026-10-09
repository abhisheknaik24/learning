# React.js Interview Questions and Answers

> This guide focuses on modern React function components and Hooks. React APIs and recommended patterns evolve; confirm behavior against the version and rendering framework used by the project.

## Contents

1. [React fundamentals](#1-react-fundamentals)
2. [Components, props, and state](#2-components-props-and-state)
3. [Hooks and rendering](#3-hooks-and-rendering)
4. [Data flow and performance](#4-data-flow-and-performance)
5. [Routing, forms, and testing](#5-routing-forms-and-testing)

## 1. React fundamentals

### 1. What is React?

React is a JavaScript library for building user interfaces from components. It describes UI as a function of props and state and updates the rendered output when relevant data changes. React itself is not a full application framework; routing, data fetching, and server rendering are provided by React APIs or surrounding frameworks/libraries.

### 2. What is a component?

A component is a reusable unit of UI. A function component accepts props and returns React elements (often written with JSX). Components should generally be focused, composable, and predictable for the same props/state.

### 3. What is JSX?

JSX is a syntax extension that lets code describe element trees using markup-like expressions. Build tooling transforms JSX into calls to the React element APIs. JSX expressions use JavaScript braces, attributes follow React conventions such as `className`, and values should be escaped safely by default.

### 4. What is the Virtual DOM?

“Virtual DOM” is an informal term for React's in-memory description of UI elements. React compares render output and coordinates updates to the host environment. It does not guarantee every update is faster than direct DOM work; architecture and rendering work still matter.

### 5. What is reconciliation?

Reconciliation is React's process of determining how a new element tree corresponds to the previous one and what updates are required. Element type, position, and keys affect whether component state is preserved or reset. A changed key can intentionally remount a component.

### 6. What is declarative UI?

Declarative UI describes what the interface should look like for current state, rather than issuing imperative DOM commands for every transition. React applies the updates. This makes state transitions and rendering easier to reason about when state is kept coherent.

## 2. Components, props, and state

### 7. What are props?

Props are inputs passed from a parent component to a child. A child should treat props as read-only and request changes through callbacks or shared state mechanisms. Props can include values, objects, and functions.

### 8. What is state?

State is data owned by a component (or an external state system) that may change over time and influence rendering. Updating React state schedules a render; directly mutating state does not notify React and can cause stale or inconsistent UI.

### 9. What is the difference between props and state?

Props are supplied by a parent and are read-only to the receiving component. State is managed by a component or state owner and updated through the relevant API. Both can trigger rendering when their values change.

### 10. Why should state be immutable?

React and optimization mechanisms often rely on identity comparisons. Mutating an object in place can make changes difficult to detect and can affect other code sharing the same reference. Create updated objects/arrays rather than mutate existing state:

```jsx
setItems(current => current.map(item =>
  item.id === id ? { ...item, done: true } : item
));
```

### 11. What is lifting state up?

Lifting state up moves shared state to the closest common ancestor of components that need it. The ancestor passes values and callbacks down. This creates a single source of truth, though excessive lifting can make components tightly coupled; context or a state library may suit broader sharing.

### 12. What is prop drilling?

Prop drilling is passing props through intermediate components that do not otherwise need them. For a small tree, explicit props can be clear. For widely shared data, context or a state-management solution may reduce plumbing, but broad context updates can cause unnecessary rerenders if designed poorly.

### 13. What are keys in lists?

Keys help React identify siblings between renders so it can preserve the correct component instances and state. Use stable, unique identifiers from data where possible. Array indexes can cause incorrect state association when items are inserted, removed, or reordered; they are acceptable only for truly static lists.

### 14. What are controlled and uncontrolled form components?

A controlled input has its displayed value driven by React state and updates that state through event handlers. An uncontrolled input keeps its current value in the DOM and is typically read through a ref or form submission. Controlled inputs simplify validation and synchronization; uncontrolled inputs can be convenient for simple forms or specialized integrations.

### 15. What is a React ref?

A ref stores a mutable value that persists across renders without causing a render when changed. Refs are commonly used to access a DOM node or retain imperative values such as timer IDs. Do not use refs as a substitute for state when a value must affect rendered output.

## 3. Hooks and rendering

### 16. What are Hooks?

Hooks are functions that let function components use React features such as state, context, refs, and effects. Hook calls must follow the Rules of Hooks: call them at the top level of React function components or custom Hooks, not conditionally or inside loops.

### 17. What does `useState` do?

`useState` declares a state value and setter. Calling the setter schedules an update; React may batch updates. When new state depends on previous state, use the functional form:

```jsx
setCount(current => current + 1);
```

This avoids relying on a potentially stale captured value.

### 18. What does `useEffect` do?

`useEffect` synchronizes a component with external systems after a render, such as subscriptions, timers, or browser APIs. It is not a general-purpose mechanism for deriving values that can be computed during rendering. Effects can return cleanup functions and should declare reactive dependencies correctly.

### 19. How does the effect dependency array work?

With no dependency array, an effect runs after each committed render. With `[]`, it runs for the component's mounted lifetime according to the rendering mode and development checks. With dependencies, it reruns when a dependency changes by React's comparison rules. Missing dependencies can cause stale closures; suppressing lint warnings without understanding them is risky.

### 20. Why does an effect cleanup matter?

Cleanup removes or cancels external work established by an effect, such as event listeners, subscriptions, or timers. React runs cleanup before re-running an effect whose dependencies changed and when the component unmounts. Cleanup prevents leaks and stale callbacks.

### 21. What is `useLayoutEffect`?

`useLayoutEffect` runs synchronously after DOM mutations but before the browser paints, allowing layout measurement and immediate visual correction. It can block painting, so use it only when pre-paint behavior is needed. In server-rendered contexts it has limitations because there is no browser layout during server rendering.

### 22. What is `useMemo`?

`useMemo` caches a calculated value between renders while its dependencies remain unchanged. It is a performance optimization, not a semantic guarantee or a substitute for correct data flow. Use it when calculation cost or referential identity matters and measurement supports it.

### 23. What is `useCallback`?

`useCallback` caches a function identity while its dependencies remain unchanged. It can reduce unnecessary work when passed to memoized children or used as an effect dependency. It does not make the function itself execute faster, and incorrect dependencies can preserve stale values.

### 24. What is `React.memo`?

`React.memo` can skip rendering a function component when its props compare equal under the configured comparison. It is an optimization, not a guarantee that rendering will never occur. New object or function references can defeat shallow comparison; profile before adding memoization broadly.

### 25. What are custom Hooks?

A custom Hook is a function whose name conventionally starts with `use` and composes built-in or other Hooks to share stateful logic. It shares logic, not a single state instance: each component calling the Hook gets its own Hook state unless it delegates to shared context or an external store.

### 26. What does `useReducer` do?

`useReducer` manages state through a reducer function that maps current state and an action to next state. It is useful for complex transitions or when updates are naturally represented as actions. Reducers should be pure; side effects belong outside the reducer.

### 27. What is Context?

Context provides a way to pass values through a component tree without manually threading props at every level. Consumers rerender when the provider value changes according to React's context behavior. Split contexts or stabilize values when broad updates become costly; context is not automatically a complete state-management system.

### 28. What is Strict Mode?

Strict Mode enables additional development checks to surface unsafe patterns. In some React versions and development configurations, it may invoke render-related logic or effect setup/cleanup more than once to detect bugs. This behavior is development-only and should not be “fixed” by removing correct cleanup.

## 4. Data flow and performance

### 29. How do you fetch data in React?

Data can be loaded through framework loaders, server components where supported, a query/cache library, or effects for simple client-side synchronization. Handle loading, error, cancellation/stale responses, and caching. Avoid creating duplicate fetch logic in many components when a shared data layer is appropriate.

### 30. How do you avoid stale network responses?

Cancel obsolete requests with `AbortController` when supported, or associate each request with an identifier and ignore results that no longer match the current input. Ensure cleanup handles component lifecycle and that loading/error state corresponds to the active request.

### 31. What causes a React component to rerender?

A component may rerender when its state changes, its parent renders, a consumed context value changes, or an external store signals an update. Rerendering does not necessarily mean the DOM is fully recreated; reconciliation determines necessary host changes.

### 32. How do you optimize a slow React interface?

Profile first using React and browser profiling tools. Identify expensive rendering, large lists, repeated calculations, or excessive network work. Possible remedies include better state locality, list virtualization, memoization where justified, code splitting, and reducing unnecessary effects. Do not optimize based only on intuition.

### 33. What is batching?

Batching groups multiple state updates so React can process them together and avoid some intermediate renders. The exact batching behavior has expanded across React versions and rendering contexts. Use functional state updates when sequential updates depend on previous values.

### 34. What is an error boundary?

An error boundary catches certain errors thrown during rendering of descendant components and displays fallback UI. Traditional error boundaries are class components or framework-provided abstractions. They do not catch every error, such as ordinary event-handler errors or all asynchronous failures; handle those at the appropriate boundary.

### 35. What is code splitting?

Code splitting divides application code into chunks that can be loaded when needed, often using dynamic imports and `React.lazy`/`Suspense` or framework features. It can reduce initial download size but introduces loading boundaries and potential latency. Split around meaningful routes or features and monitor chunk behavior.

### 36. What is server-side rendering?

Server-side rendering produces HTML on the server for an initial page, which can improve initial content availability and SEO depending on the framework. The client may hydrate the markup to attach interactivity. Server rendering introduces requirements around deterministic markup, data loading, caching, and avoiding browser-only APIs during server execution.

### 37. What is hydration?

Hydration attaches client-side React behavior to server-rendered HTML. If the server and client render different initial markup, hydration mismatches can occur. Use consistent data and avoid rendering decisions based on browser-only state until the client can safely access it.

## 5. Routing, forms, and testing

### 38. Is routing built into React?

React core does not prescribe a full router. Applications commonly use a routing library or framework that maps URLs to views, supports navigation, and may provide loaders or server integration. Use the routing solution supported by the application's framework.

### 39. How should forms be validated?

Validate at the user interface for immediate feedback, and validate again on the server because client-side checks can be bypassed. Keep validation rules reusable where appropriate, expose accessible error messages, and distinguish field errors from submission/network failures.

### 40. What is accessibility in React applications?

Use semantic HTML, labels, keyboard-operable controls, correct focus management, and accessible names and status announcements. React does not automatically make a custom widget accessible. Prefer native elements over recreating browser behavior where possible.

### 41. How do you test React components?

Test behavior through user-visible interactions and accessible queries using a component testing library. Mock external boundaries rather than internal implementation details. Add integration and end-to-end tests for important workflows; avoid tests that fail whenever harmless markup or internal structure changes.

### 42. What is a portal?

A portal renders children into a DOM node outside the parent component's DOM hierarchy while preserving React context and event relationships. Portals are commonly used for dialogs and overlays. Manage focus, accessibility, and cleanup for the resulting UI.

### 43. What are fragments?

Fragments group multiple elements without adding an extra DOM node. The long form can accept a key when rendering a list; the short `<>...</>` form cannot take keys or other props.

### 44. What is the output?

```jsx
function Counter() {
  const [count, setCount] = React.useState(0);
  function incrementThreeTimes() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }
  return <button onClick={incrementThreeTimes}>{count}</button>;
}
```

**Answer:** In the usual batched event behavior, one click changes the displayed value from `0` to `1`, because each update uses the same render's captured `count`. Use `setCount(value => value + 1)` three times to increment by three.

### 45. Why should array indexes often not be used as keys?

When items reorder or are removed, an index key can make React associate existing component state with a different item. Use a stable ID from the data. If the list never changes order or membership, an index may be acceptable, but stable identity is generally safer.

### 46. What is the difference between state and a ref?

State updates request a render and should be used for values that affect UI. A ref persists a mutable value without triggering a render and is suited to imperative handles or bookkeeping. Mutating a ref does not update displayed output automatically.

### 47. What is Suspense?

`Suspense` displays fallback UI while supported child work is pending, such as lazy-loaded code or data handled through a compatible framework/library. It does not automatically detect every arbitrary promise started in an effect. Suspense data behavior depends on the React version and surrounding framework.

### 48. What makes a good React component?

A good component has clear inputs and responsibilities, keeps state near its consumers, uses semantic accessible markup, and avoids unnecessary effects or hidden side effects during rendering. Extract components or Hooks when they improve reuse, separation, or comprehension—not merely to reduce line count.
