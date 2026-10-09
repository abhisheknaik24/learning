# React.js Interview Questions and Answers

> This guide focuses on modern React function components and Hooks. React APIs and recommended patterns evolve; confirm behavior against the version and rendering framework used by the project.

## Contents

1. [React fundamentals](#1-react-fundamentals)
2. [Components, props, and state](#2-components-props-and-state)
3. [Hooks and rendering](#3-hooks-and-rendering)
4. [Data flow and performance](#4-data-flow-and-performance)
5. [Routing, forms, and testing](#5-routing-forms-and-testing)
6. [Elements, JSX, and component design](#6-elements-jsx-and-component-design)
7. [State, identity, and reconciliation](#7-state-identity-and-reconciliation)
8. [Hooks in depth](#8-hooks-in-depth)
9. [Concurrent rendering and Suspense](#9-concurrent-rendering-and-suspense)
10. [Effects, context, and external stores](#10-effects-context-and-external-stores)
11. [Forms, actions, and mutations](#11-forms-actions-and-mutations)
12. [Routing, data fetching, and caching](#12-routing-data-fetching-and-caching)
13. [Server rendering, streaming, and hydration](#13-server-rendering-streaming-and-hydration)
14. [Accessibility and security](#14-accessibility-and-security)
15. [Testing and performance](#15-testing-and-performance)
16. [Architecture, tooling, and version caveats](#16-architecture-tooling-and-version-caveats)

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

## 6. Elements, JSX, and component design

### 49. What is a React element?

A React element is an immutable description of a UI node, not the DOM node itself.

### 50. How do elements differ from components?

A component is callable UI logic; an element is the value describing a component or host node in a tree.

### 51. What does JSX compile to?

Modern transforms compile JSX to runtime functions such as `jsx`/`jsxs`; exact output depends on compiler settings.

### 52. Must JSX return one root element?

An expression must return one value, but a Fragment can group multiple siblings without extra DOM.

### 53. How do you embed JavaScript in JSX?

Use `{expression}` in child content or supported attribute values; statements require surrounding JavaScript logic.

### 54. How are boolean JSX attributes represented?

`<Button disabled />` passes `true`; pass an expression for dynamic values and omit false boolean attributes where appropriate.

### 55. How do HTML attributes differ in JSX?

Most follow DOM property names, such as `className` and `htmlFor`; custom data attributes retain their lowercase spelling.

### 56. What does `children` represent?

`children` is the nested content passed between a component's opening and closing tags; it may be text, elements, or other renderable values.

### 57. Can a component return `null`?

Yes. Returning `null` renders no host content while the component can still participate in React's tree.

### 58. What does `false` render as a child?

Booleans, `null`, and `undefined` are omitted; beware that numeric zero is rendered as text.

### 59. Can a component return an array?

Yes. An array of renderable nodes is valid, and sibling elements in that array need stable keys.

### 60. What are render props?

A render prop is a function passed to customize rendered output; Hooks often offer a simpler composition alternative.

### 61. What is a higher-order component?

A higher-order component accepts a component and returns an enhanced component; it remains useful in some libraries but can add wrapper complexity.

### 62. What is component composition?

Composition builds complex UI by nesting focused components and passing slots or children instead of relying on inheritance.

### 63. What are slots in React?

Slots are an API pattern for caller-provided content, commonly implemented with `children` or named element props.

### 64. What is a polymorphic component?

A polymorphic component can render different host or component types; its API must preserve correct attributes, refs, semantics, and types.

### 65. Why should render functions be pure?

Pure rendering makes output predictable and lets React restart or repeat render work without duplicated external effects.

### 66. Why avoid calling a component as a regular function?

Render it as `<Child />`; direct calls bypass React's component boundaries and can violate Hook ordering or debugging assumptions.

### 67. What is a component's display name?

`displayName` is an optional debugging label used by tools and warnings, especially for wrapped or anonymous components.

### 68. How should component props be designed?

Prefer a small, explicit API with meaningful names, sensible defaults, and types that prevent invalid combinations.

### 69. When should a component be extracted?

Extract when a boundary clarifies responsibility, reuse, state ownership, or testing—not solely because a function is long.

### 70. What is a compound component?

A compound component family shares implicit coordination, often through context, while giving callers a flexible declarative composition API.

### 71. What is a controlled component API?

The parent supplies the value and change callback, retaining authority over the component's state.

### 72. What is an uncontrolled component API?

The component or DOM owns current state; callers may initialize it and inspect or reset it through an imperative interface.

### 73. How do React event handlers differ from native listeners?

JSX handlers are managed by React's event system; modern React delegates many events, but details vary by event and version.

### 74. What is event propagation in React?

Events generally bubble through the React tree; capture handlers run first, and `stopPropagation()` prevents further propagation.

### 75. How do you pass arguments to an event handler?

Use a closure such as `onClick={() => remove(id)}`; avoid invoking the handler during render accidentally.

### 76. What does `preventDefault()` do?

It cancels a browser default action when the event is cancelable; it does not stop propagation.

### 77. What is the purpose of `key` in JSX?

`key` helps React match sibling elements across renders and is consumed by React rather than passed as an ordinary prop.

### 78. How do you pass a key to component logic?

Pass a separate named prop, such as `itemId`; `key` itself is not available inside the component.

## 7. State, identity, and reconciliation

### 79. What does it mean that state is a render snapshot?

Each render and its handlers observe the state values from that render, even after scheduling an update.

### 80. Why might a state setter not update a variable immediately?

It schedules a later render; the current function continues using its existing closure and snapshot.

### 81. When should a state update use a function?

Use an updater when the next value depends on prior state, especially for multiple queued updates.

### 82. How are object state updates performed?

Create a replacement object, usually spreading unchanged fields; the setter does not merge objects automatically.

### 83. How are array state updates performed?

Use non-mutating operations such as `map`, `filter`, and spread to produce a new array.

### 84. Why avoid redundant state?

Duplicated facts can diverge; derive values during render whenever they can be computed from existing props or state.

### 85. What is derived state?

Derived state is calculated from other inputs; storing it separately often creates synchronization bugs unless it represents a deliberate snapshot.

### 86. How do you reset a component's state?

Render it with a new key when a full remount is appropriate, or explicitly reset the relevant state values.

### 87. What determines whether state is preserved?

React associates state with a component's position, type, and key in the rendered tree.

### 88. What happens when a component type changes at one position?

React typically unmounts the old subtree and mounts the new type, resetting its local state.

### 89. Why can nested component definitions cause remounts?

Defining a component inside another creates a new component function identity per render, which can reset child state.

### 90. Are keys globally unique?

No. Keys need to be unique among siblings in the same list, not across the entire application.

### 91. Can keys be generated randomly during render?

Avoid it: changing keys remounts elements and discards their local state on every render.

### 92. What is a keyed reset pattern?

Changing a component's key deliberately creates a fresh instance, useful when switching between distinct records or workflows.

### 93. How does React compare dependencies?

Hook dependency entries are compared with `Object.is`; a new object reference therefore counts as changed.

### 94. What is structural sharing?

Structural sharing reuses unchanged nested references while replacing changed paths, supporting efficient identity-based comparisons.

### 95. What is a stale closure?

A stale closure reads values captured by an earlier render when later work executes; correct dependencies or functional updates address many cases.

### 96. How can state be normalized?

Store entities by stable ID and represent relationships with IDs to reduce duplicated nested data and inconsistent updates.

### 97. What is an optimistic state update?

It displays an expected result before the server confirms it, then reconciles success or failure with authoritative data.

### 98. What does React batching change?

Batching can combine queued updates into fewer renders; it does not change the rule that each handler sees its render snapshot.

### 99. What is a transition update?

A transition marks non-urgent rendering work so urgent interactions can remain responsive; it is not a network transaction.

### 100. How should independent state variables be grouped?

Group values that transition together; separate unrelated values when that makes updates and ownership clearer.

### 101. When is a reducer preferable to multiple setters?

Use a reducer when transitions involve coordinated fields, explicit actions, or logic that benefits from centralized testing.

### 102. What makes a reducer safe?

It should be pure, deterministic, and return new state rather than mutating existing state.

### 103. What is lazy state initialization?

Passing an initializer function to `useState` calculates initial state only when React initializes that component instance.

### 104. Does a new `useState` initial value reset existing state?

No. The initial value is used on mount; later prop changes do not automatically replace state.

### 105. What is state colocation?

State colocation keeps each state value near the components that own and use it, limiting unrelated rerenders and coupling.

### 106. What is an external state store?

It is a state container outside component-local Hooks that can notify subscribed components of updates.

### 107. What is tearing in concurrent rendering?

Tearing is inconsistent UI reading different versions of shared mutable data; external-store integration APIs help prevent it.

### 108. How should state updates handle asynchronous results?

Associate results with the active request or entity and ignore or cancel responses that have become obsolete.

### 109. Should a ref be used for displayed state?

No. Ref mutations do not trigger rendering; use state for values whose changes should update visible output.

### 110. Can state be shared between Hook callers automatically?

No. Each component has its own Hook state unless the Hook uses a shared provider or external store.

## 8. Hooks in depth

### 111. What are the Rules of Hooks?

Call Hooks unconditionally at the top level of function components or custom Hooks, preserving call order across renders.

### 112. Why can Hooks not be called conditionally?

React associates Hook state with call order; conditional calls can shift that order and associate values incorrectly.

### 113. What is a custom Hook's purpose?

It packages reusable stateful behavior and effect logic behind a function whose name begins with `use`.

### 114. Does a custom Hook share its local state?

No. Each call creates an independent Hook state unless it connects to shared context or a store.

### 115. What does `useRef` return?

It returns a stable ref object whose mutable `.current` property persists across renders without scheduling updates.

### 116. When does a DOM ref become available?

React assigns it during commit; it is generally available after the node is mounted, not during render.

### 117. What does `useImperativeHandle` do?

It customizes the value exposed through a ref, allowing a component to provide a narrow imperative API.

### 118. Is `forwardRef` required in every React version?

No. React 19 supports `ref` as a prop for function components; earlier versions commonly use `forwardRef`.

### 119. What does `useId` provide?

It generates stable IDs suitable for associating related accessibility attributes across server rendering and hydration.

### 120. Should `useId` generate list keys?

No. Keys should come from data identity; `useId` is intended for accessibility relationships and similar stable IDs.

### 121. What does `useTransition` return?

It returns a pending flag and a function to mark state updates as non-urgent transitions.

### 122. What does the `startTransition` function do?

It marks synchronously scheduled state updates in its callback as non-urgent; it does not delay arbitrary side effects.

### 123. Can an urgent update interrupt a transition?

Yes. React can prioritize urgent updates such as typing and continue or restart interruptible transition rendering.

### 124. What does `useDeferredValue` do?

It lets a non-urgent part of the UI lag behind a rapidly changing value while React prepares updated rendering.

### 125. Does `useDeferredValue` debounce network requests?

No. It defers rendering priority, not a fixed interval; implement request debouncing separately when required.

### 126. What does `useSyncExternalStore` solve?

It subscribes safely to an external store and reads snapshots consistently, including under concurrent rendering.

### 127. What does `useSyncExternalStore` need?

It needs a subscription function, a snapshot reader, and for server rendering an appropriate server snapshot reader.

### 128. What is the `use` API?

In supported React versions, `use` can read a supported resource such as a Promise or context during render.

### 129. Can `use` be called conditionally?

Unlike ordinary Hooks, `use` may be called in conditions and loops in supported React versions, but not outside components or Hooks.

### 130. How does `use` handle a pending Promise?

It suspends the component until the Promise settles, allowing a nearby Suspense boundary to show fallback UI.

### 131. What happens when a Promise read with `use` rejects?

The rejection propagates to an error boundary or framework error handling, rather than becoming a fulfilled value.

### 132. What is `useEffectEvent`?

In React versions that provide it, it extracts non-reactive event logic from an Effect while reading the latest props and state.

### 133. Should an Effect Event be used as a child callback?

No. It is for logic called from Effects, not a general mechanism for stable event handlers passed through the UI.

### 134. What is `useActionState`?

React 19's `useActionState` manages the result and pending state of an Action, including form-related submissions.

### 135. What arguments does a `useActionState` action receive?

It receives the previous state and the action's payload, commonly `FormData` when used with a form.

### 136. What does `useFormStatus` report?

In React DOM, it exposes status for the nearest parent form submission, such as whether it is pending.

### 137. Why must `useFormStatus` be inside a form descendant?

It reads the nearest parent form's submission context, so a Hook in the form component itself does not observe that form.

### 138. What does `useOptimistic` do?

It derives temporary optimistic UI state while an Action is pending, then returns to the authoritative state when the Action completes.

### 139. Does `useOptimistic` persist server data?

No. It only represents temporary UI state; the mutation and durable data update must be handled separately.

### 140. What does `useDebugValue` do?

It labels custom Hook values in React DevTools and has no effect on application rendering semantics.

### 141. What is the difference between `useEffect` and `useLayoutEffect`?

`useEffect` generally runs after paint; `useLayoutEffect` runs before paint after DOM mutation and may block visual updates.

### 142. When should `useInsertionEffect` be used?

Libraries that inject styles may use it before layout effects; application code should generally prefer other Effects.

### 143. What are the limitations of `useInsertionEffect`?

It is intended for CSS-in-JS integration, cannot access refs reliably, and should not be used to schedule state updates.

### 144. When is `useMemo` appropriate?

Use it to avoid measured expensive recalculation or preserve identity needed by a memoized boundary, not by default.

### 145. Is `useMemo` a correctness guarantee?

No. React may discard cached values in some circumstances; code must remain correct without the cache.

### 146. When is `useCallback` appropriate?

Use it when function identity measurably affects a memoized child or a Hook dependency, not simply to wrap every handler.

### 147. How do missing dependencies affect a callback?

The callback may close over outdated values; include all reactive dependencies or redesign the logic.

### 148. What does the `exhaustive-deps` lint rule detect?

It identifies reactive values used by Effects and memoization callbacks but missing from their dependency lists.

### 149. Should dependencies be removed to stop an Effect rerun?

No. Restructure the Effect or stabilize its inputs; omitting required dependencies can create stale behavior.

### 150. What is a reactive value in an Effect?

Props, state, and values or functions declared in the component body can change between renders and may be dependencies.

### 151. What is an Effect's setup function?

It runs after a commit to synchronize with an external system and may return cleanup for that synchronization.

### 152. When does React run Effect cleanup?

Before rerunning setup with changed dependencies and when the component is removed from the tree.

### 153. Why can Strict Mode repeat Effect setup in development?

The extra setup-cleanup cycle checks whether cleanup correctly reverses setup; production behavior is not duplicated for this check.

### 154. Is an Effect the right way to derive filtered data?

Usually not. Compute derived data during render, optionally memoizing only if measurement shows a cost.

### 155. What is an Effect synchronization loop?

An Effect updates a dependency it observes on every run, causing repeated renders; make synchronization idempotent or correct the dependency design.

### 156. How do you subscribe to a browser event in an Effect?

Add the listener in setup and remove the same listener in cleanup, accounting for changing handler dependencies.

### 157. How can an Effect avoid outdated event values?

Use proper dependencies, functional updates, or a supported Effect Event where the callback is genuinely non-reactive.

### 158. Should Effects run during server rendering?

No. Effects run only on the client after commit; server output must not depend on their results.

### 159. How can an Effect avoid a flash of incorrect layout?

Use `useLayoutEffect` only when a pre-paint measurement correction is necessary; otherwise design stable initial markup.

### 160. What is an asynchronous Effect cleanup limitation?

An Effect callback cannot itself return a Promise; start async work inside it and abort or disregard stale work in cleanup.

### 161. How can an Effect cancel a fetch?

Create an `AbortController`, pass its signal, and abort it from cleanup; still handle rejection and stale results.

### 162. What is context propagation?

Context consumers read the nearest matching provider, and a changed provider value schedules updates for its consumers.

### 163. Why can a context provider trigger many renders?

All consumers of a changed value may update; splitting values or using selector-capable stores can narrow subscriptions.

### 164. Does `React.memo` block context updates?

No. A component consuming a changed context can rerender even when its props are memoized.

### 165. What is a context default value?

It is used only when no matching provider exists above the consumer, not when a provider explicitly supplies `undefined`.

### 166. How can a context Hook detect a missing provider?

Use a distinctive undefined default and throw a clear error from a custom consumer Hook when absent.

### 167. Should frequently changing data always use context?

Not necessarily; high-frequency broad context updates may be better served by localized state or a subscription-based store.

### 168. What is context value stabilization?

Memoizing an object value can avoid changes caused only by new object identity, but consumers still update when actual fields change.

### 169. Can context be used to inject a service?

Yes. Context can provide stable dependencies such as clients or configuration, with clear provider ownership and test overrides.

### 170. What is an external-store snapshot?

It is an immutable or stable representation read by React to determine whether subscribed external data changed.

### 171. Why should a store snapshot be cached?

Returning a newly allocated snapshot every read can appear perpetually changed and cause repeated rendering.

### 172. What does a store subscription cleanup do?

It unsubscribes the component when it no longer needs updates, preventing leaks and unnecessary notifications.

### 173. How should an external store support SSR?

Provide a server snapshot consistent with the HTML and initial client snapshot to avoid hydration inconsistency.

### 174. What is state-machine modeling?

It represents UI as explicit states and transitions, preventing impossible combinations such as loading and success simultaneously.

### 175. When is a state machine useful in React?

Use it for workflows with many states, guarded transitions, retries, or concurrent events that are hard to manage with booleans.

### 176. What is an actor model for UI state?

An actor encapsulates state and message handling, often coordinating asynchronous work independently of component rendering.

### 177. What is a selector in state management?

A selector derives a focused value from a store so components can subscribe to only the data they need.

### 178. What is a selector equality function?

It determines whether selected values are meaningfully different and therefore whether a subscriber should update.

### 179. How should global state be limited?

Keep state local by default; promote it only when multiple routes or distant components genuinely share ownership.

### 180. What are the risks of putting everything in a global store?

It expands coupling, makes ownership unclear, and can increase update fan-out and testing complexity.

### 181. What is a controlled context boundary?

It is a provider placed around the smallest subtree that needs shared data, limiting scope and rerender impact.

### 182. How can custom Hooks hide implementation details?

Expose a small semantic API while keeping subscription, effect, and state-transition mechanics internal.

### 183. Should custom Hooks return tuples or objects?

Choose based on use: tuples suit fixed positional values; objects offer named fields and easier selective destructuring.

### 184. Why should custom Hook names start with `use`?

The convention enables Hook lint rules and signals that the function follows React's Hook call constraints.

### 185. Can a Hook be called in a class component?

No. Hooks are for function components and custom Hooks; class components use lifecycle methods or wrappers.

### 186. What is a Hook dependency identity issue?

Inline objects and functions are recreated each render, making them appear changed; restructure first and memoize only when useful.

### 187. How should timer IDs be stored?

A ref is suitable when the timer ID is bookkeeping that should persist without affecting rendered output.

### 188. How should a component coordinate focus after rendering?

Use a ref and a narrowly scoped layout or passive Effect depending on whether focus must occur before paint.

### 189. Why should render-phase state updates be avoided?

They make rendering harder to reason about and can create loops; derive values or update in event handlers and Effects as appropriate.

### 190. What is the compiler's relationship to memoization?

React Compiler can automatically optimize supported components; availability and guarantees depend on toolchain configuration and version.

## 9. Concurrent rendering and Suspense

### 191. What does concurrent rendering mean?

React can prepare rendering work interruptibly and prioritize updates, while only committed work changes the UI.

### 192. Is concurrent rendering a separate mode users toggle?

Modern React roots use concurrent-capable rendering, but interruptible behavior is applied according to React features and update priority.

### 193. Can a render be interrupted?

Yes. React may pause, restart, or abandon render work before commit; render logic must therefore remain pure.

### 194. What is the render phase?

React calls components and calculates the next tree; this phase should not perform external side effects.

### 195. What is the commit phase?

React applies host changes and runs commit-related work for the selected render; committed UI is what users observe.

### 196. What is an urgent update?

It is work users expect immediately, such as reflecting typed input or direct interaction feedback.

### 197. What is a non-urgent update?

It is work that can be delayed, such as rendering a large filtered result after an input remains responsive.

### 198. What does Suspense coordinate?

It lets a boundary show fallback content while supported descendant rendering work is waiting for a resource.

### 199. Can any Promise automatically trigger Suspense?

No. The Promise must be read through a supported React or framework integration; arbitrary Effect fetches do not suspend automatically.

### 200. What is a Suspense boundary?

It defines the part of the tree that can be replaced by fallback UI while its supported work is pending.

### 201. How should Suspense boundaries be placed?

Place them around meaningful independently loadable regions so one pending part need not blank the entire page.

### 202. What happens if a child suspends before its first mount?

React can discard its incomplete render and retry it when the resource becomes available; no uncommitted state is preserved.

### 203. Can already visible content be replaced by a fallback?

It can, depending on update priority and boundary behavior; transitions can help preserve existing content during non-urgent updates.

### 204. What is a SuspenseList?

It coordinates reveal order for multiple Suspense boundaries where supported; support and behavior should be checked for the target React version.

### 205. What is a fallback UI?

Fallback UI is a lightweight accessible placeholder rendered by a Suspense boundary while its child work is pending.

### 206. How do error boundaries relate to Suspense?

Suspense handles pending supported work; error boundaries handle rendering errors and rejected resources through their respective mechanisms.

### 207. What is selective hydration?

In supported SSR setups, React can hydrate parts of a page as their code or user interaction makes them ready.

### 208. What does event replay mean during hydration?

Some user events can be queued and replayed after the relevant server-rendered region becomes interactive, depending on the framework.

### 209. What is a transition's pending state?

It indicates that work marked as a transition has not yet finished, allowing the UI to communicate progress.

### 210. Can a transition control text input state?

The state that directly controls an input should remain urgent; transition expensive dependent rendering instead.

### 211. Do transitions guarantee a render will never block?

No. They provide scheduling priority, but expensive synchronous JavaScript and unsupported work can still block the main thread.

### 212. What is interruption safety?

It is the property that restarted render work cannot duplicate irreversible effects because such effects are deferred until commit.

### 213. Can concurrent rendering expose mutation bugs?

Yes. Mutating shared objects during render can make interrupted or repeated work observe inconsistent data.

### 214. What is tearing prevention for stores?

React's external-store API coordinates snapshots so concurrent renders do not commit a mixture of store versions.

### 215. What is a render lane?

Internally, React uses priority categories to track update work; applications should use public scheduling APIs, not depend on lane internals.

### 216. What is `flushSync`?

`flushSync` forces pending React updates to flush synchronously in limited integration cases; it can hurt performance and should be rare.

### 217. When might `flushSync` be needed?

A third-party imperative API may require the DOM to reflect an update before the next line executes.

### 218. Why can excessive `flushSync` be harmful?

It defeats scheduling flexibility, can force extra work, and may cause responsiveness regressions.

### 219. What does a Suspense-enabled router improve?

It can coordinate navigation, loading boundaries, and transitions so pending route content does not unnecessarily replace the whole screen.

### 220. How should pending work be communicated accessibly?

Use clear status text or progress semantics when appropriate, preserve useful existing content, and avoid announcing every trivial render.

## 10. Effects, context, and external stores

### 221. What does “synchronize with an external system” mean?

It means reflecting React state in an imperative resource or subscribing to something React does not own.

### 222. What is an example of synchronization that needs an Effect?

Connecting to a websocket, controlling a media player, or subscribing to a browser event are typical examples.

### 223. Is copying props to state usually necessary?

No. It duplicates a source of truth; derive the value from props unless preserving a deliberate editable draft is required.

### 224. How should an editable draft respond to a new record?

Give the editor a record-specific key or explicitly reset its draft when the identity changes.

### 225. What is an Effect race condition?

An older asynchronous operation completes after a newer one and overwrites state; abort or validate request identity.

### 226. What is idempotent Effect setup?

Repeated setup produces the same intended external state without accumulating duplicate subscriptions or resources.

### 227. Why should cleanup mirror setup?

It releases exactly the resource that setup acquired, making reruns and unmounts safe.

### 228. Can an Effect depend on a mutable ref's `.current`?

Changing `.current` does not rerender React, so it is not a reliable reactive dependency.

### 229. How can a callback dependency be removed safely?

Move non-reactive logic outside the component, use a functional state update, or restructure the synchronization.

### 230. What is an Effect Event useful for?

Where supported, it lets an Effect invoke logic that must observe the latest values without resubscribing for those values.

### 231. Why avoid using Effects for event-specific work?

An interaction's side effect belongs in its event handler; an Effect can rerun for unrelated renders and obscure causality.

### 232. Should an Effect set state after every render?

Only when synchronizing an external result that truly requires it; otherwise derive the value and avoid a second render.

### 233. What does `useLayoutEffect` block?

It blocks browser painting while its synchronous work runs, so keep it brief and limited to layout-sensitive tasks.

### 234. How do you make browser-only code SSR-safe?

Access browser globals in client-only code or guarded Effects, not unconditionally during server rendering.

### 235. What is a context provider boundary's lifetime?

It lasts while that provider instance remains at the same reconciled position and identity in the tree.

### 236. Does changing a provider recreate all consumer state?

Changing its value updates consumers; remounting or changing provider identity can instead reset descendant state.

### 237. How should nested providers of the same context behave?

A consumer reads the nearest provider above it, enabling scoped overrides such as themes or locale.

### 238. Is context a dependency injection mechanism?

It can provide dependencies per subtree, though explicit props may be simpler for local dependencies.

### 239. How can context avoid unstable object values?

Separate independent values or memoize the provider object where measured rerender fan-out justifies it.

### 240. What is context selector behavior?

Core `useContext` subscribes to the context value as a whole; selector behavior generally comes from a library or store abstraction.

### 241. How should provider values be typed?

Model the real API explicitly and avoid overly broad `any`; a nullable context plus guarded Hook is a common pattern.

### 242. What is a provider composition component?

It groups related providers behind a readable boundary, reducing nesting at application entry points.

### 243. Can context replace every state library?

No. It transports values but does not inherently provide selectors, persistence, server caching, or complex update tooling.

### 244. What is a subscription snapshot consistency problem?

A render can read data that changes between render and subscription; `useSyncExternalStore` is designed to coordinate this boundary.

### 245. What should a subscription function return?

It should return a cleanup function that unsubscribes the exact listener registered.

### 246. Why must external-store subscribe functions be stable?

Changing the subscribe function identity can force React to unsubscribe and resubscribe; define it outside render or stabilize it.

### 247. What is a derived store selector?

It computes a component-specific projection from store state, reducing the data a consumer must observe.

### 248. How should an external store notify subscribers?

Notify after a coherent state change, and ensure snapshot reads return the corresponding current state.

### 249. Can a store mutate its data internally?

It can, but snapshots must still have reliable change identity and avoid exposing inconsistent mutable state to concurrent renders.

### 250. What is a good boundary between React and imperative widgets?

Let React own the container and lifecycle; initialize, update, and dispose the widget through a narrowly scoped Effect.

## 11. Forms, actions, and mutations

### 251. What is a controlled text input?

Its `value` comes from React state and `onChange` updates that state synchronously.

### 252. What causes a controlled input warning?

An input switches between controlled and uncontrolled states, commonly because its value changes from `undefined` to a string.

### 253. How should a controlled input value be initialized?

Use a consistent string or boolean initial value, such as `''` for text, throughout its controlled lifetime.

### 254. What is an uncontrolled form field?

The browser owns its current value, which can be read from `FormData` or a ref at submission.

### 255. How do you read form values with `FormData`?

Construct `new FormData(formElement)` and retrieve fields by their `name`, accounting for repeated names and missing values.

### 256. Why do form controls need `name` attributes?

Names identify submitted fields in native form data and are also used by browser and server form workflows.

### 257. What does a checkbox's `checked` prop control?

It controls the checkbox's boolean selection state; its `value` is the submitted string when checked.

### 258. How is a multiple select controlled?

Supply an array of selected values and update it from the selected options in the change event.

### 259. What is the difference between `value` and `defaultValue`?

`value` controls the current value; `defaultValue` initializes an uncontrolled field.

### 260. How are file inputs controlled?

Browsers restrict setting their file value; typically leave the input uncontrolled and read selected files through its event or ref.

### 261. Why validate on both client and server?

Client validation improves feedback, but only server validation can enforce trust across tampered or direct requests.

### 262. What is constraint validation?

It is native browser validation based on attributes such as `required`, `type`, and `minLength`.

### 263. How should field errors be associated accessibly?

Provide a visible label and connect explanatory or error text using `aria-describedby`; expose invalid state when appropriate.

### 264. How should a submission error be announced?

Render a clear message near the form and use appropriate live-region or focus behavior without overwhelming users.

### 265. What is a form Action in React 19?

A function supplied to a form's `action` can handle submission, including progressive-enhancement and pending-state workflows in supported environments.

### 266. Can a form Action run on the server?

Server Actions are framework/build-system features, not a generic React-only deployment feature; follow the framework's security model.

### 267. What is `useActionState` useful for in a form?

It returns action state and pending information, useful for validation results or messages after a submission.

### 268. What does `useFormStatus` help implement?

It lets a nested submit button or status component reflect the parent form's pending submission state.

### 269. Why disable or guard duplicate submissions?

It prevents accidental repeated requests, but server-side idempotency or deduplication is still needed for correctness.

### 270. What is an idempotency key?

It is a request identifier that lets a server recognize retries of the same operation and avoid duplicate effects.

### 271. What is optimistic form feedback?

The interface shows an expected mutation result immediately, then confirms or corrects it when the server responds.

### 272. How should optimistic updates handle failure?

Revert or reconcile temporary state and show a useful error while retaining any user input that can be safely retried.

### 273. What is progressive enhancement for forms?

A form works with standard browser submission first and gains richer client behavior when JavaScript is available.

### 274. Why use a real `<form>` element?

It provides built-in keyboard submission, semantics, browser behavior, and integration with native or framework form handling.

### 275. How should Enter-key submission work?

Preserve native form semantics and use a submit button; custom key handlers are usually unnecessary.

### 276. What does `requestSubmit()` do?

It requests form submission through normal submit-button and validation behavior, unlike directly invoking the low-level `submit()` method.

### 277. How should async validation be handled?

Debounce where useful, associate results with current input, expose pending feedback, and validate again authoritatively on the server.

### 278. How can a form preserve user input after an error?

Keep the submitted values or draft intact and report recoverable errors without resetting the form prematurely.

### 279. What is a controlled-to-uncontrolled migration?

It changes who owns field values; avoid switching an individual input's mode during its lifetime.

### 280. How do form libraries relate to React?

They provide abstractions for registration, validation, and state; choose based on control needs, accessibility, and bundle/runtime constraints.

## 12. Routing, data fetching, and caching

### 281. What is client-side routing?

A client router maps URL state to rendered UI and can navigate without a full document reload.

### 282. What is a route?

A route associates a URL pattern with UI and often related data-loading, mutation, and error-boundary behavior.

### 283. What is a nested route?

A nested route renders within a parent route's outlet or layout, sharing surrounding UI and route context.

### 284. What is a route outlet?

It is a router-provided insertion point where the matching child route is rendered.

### 285. Why should URL state be shareable?

Encoding navigational state in the URL enables bookmarking, refresh recovery, and browser back/forward behavior.

### 286. What state belongs in a URL?

Search terms, filters, pagination, and selected resources often belong there when users should share or revisit them.

### 287. What is a route loader?

A loader obtains route data before or during navigation according to a router's data APIs.

### 288. What is a route action?

A route action handles a mutation associated with a route, often followed by invalidation or revalidation of loader data.

### 289. What is a navigation pending state?

It indicates a route transition is in progress so the UI can communicate navigation without losing useful content.

### 290. What is a route error boundary?

It presents localized recovery UI for errors associated with a route subtree rather than failing the entire application.

### 291. How should route parameters be validated?

Parse and validate them at the route or data boundary; URL input is untrusted and may be absent or malformed.

### 292. What is a protected route?

It gates access based on authorization, but real security must be enforced by the server for protected data and actions.

### 293. How should redirects be handled?

Use the router's navigation or redirect APIs and validate externally supplied redirect destinations to avoid open redirects.

### 294. What is a not-found route?

It renders an explicit response for unmatched paths and should set appropriate HTTP status during server rendering when possible.

### 295. What is data prefetching?

It loads likely-needed route data before navigation to reduce perceived latency, balanced against bandwidth and cache freshness.

### 296. What is a query cache?

It stores fetched server data with identity, freshness, and lifecycle policies to deduplicate requests and coordinate updates.

### 297. How is server state different from client UI state?

Server state is remotely authoritative and needs fetching, caching, and invalidation; UI state is local interaction state.

### 298. What is cache invalidation?

It marks cached data stale or removes it so later reads obtain data consistent with a mutation or policy.

### 299. What is stale-while-revalidate?

It returns cached data immediately while refreshing it in the background, then updates the view when fresher data arrives.

### 300. What is request deduplication?

Equivalent concurrent requests share work or a cached result, reducing duplicate network and server load.

### 301. What is a cache key?

It identifies a resource and all relevant query inputs; omitting an input can return incorrect cached data.

### 302. How should cache keys handle user identity?

Include appropriate account or authorization scope and clear sensitive cached data when identity changes.

### 303. What is optimistic cache mutation?

It updates cached data before server confirmation and later commits, reconciles, or rolls back the change.

### 304. How should a cache handle overlapping mutations?

Use mutation identity and reconciliation rules so an earlier failure does not erase a later successful update.

### 305. What is pagination?

It divides a result set into addressable portions; cursor-based pagination often handles changing data more robustly than offsets.

### 306. What is infinite scrolling?

It loads additional pages as users approach the end; retain accessible controls and manage memory for long lists.

### 307. What is request cancellation useful for?

It stops obsolete work after navigation or input changes, reducing resource use while still guarding against late responses.

### 308. Should every fetch be started in an Effect?

No. Framework loaders or query libraries can provide caching, deduplication, and navigation integration more reliably.

### 309. What is waterfall fetching?

It is sequential dependent fetching that delays later requests; parallelize independent requests or use server data orchestration.

### 310. What is a data-fetching waterfall in components?

A parent waits for data before rendering a child that starts another request, increasing total latency.

### 311. How can fetch waterfalls be reduced?

Start independent requests in parallel, preload likely data, or move orchestration to a route or server boundary.

### 312. What is an API query library's role?

It centralizes remote-data caching, retries, invalidation, subscriptions, and loading states while leaving UI composition to React.

### 313. What is a retry policy?

It defines whether and when failed requests are repeated, with limits and backoff to avoid amplifying outages.

### 314. Why use exponential backoff?

It spaces retries increasingly to reduce load during transient failure; add jitter to avoid synchronized retry bursts.

### 315. What is a mutation invalidation tag?

It associates cached queries with data affected by a mutation so relevant entries can be refreshed.

### 316. How should stale data be presented?

Keep it clearly distinguishable when necessary, indicate refresh activity, and avoid implying stale results are guaranteed current.

### 317. What is cache time-to-live?

It is a freshness or retention policy that determines how long cached entries remain usable or stored.

### 318. What is cache hydration?

It transfers preloaded server cache data to the client so the initial client view can reuse it.

### 319. What can go wrong with a shared server cache?

User-specific or request-specific data can leak across requests unless cache scope and keys are carefully isolated.

### 320. What is a service worker cache?

A browser-managed cache controlled by a service worker, distinct from React state and application query caches.

### 321. How do router prefetch and query-cache prefetch relate?

A router may preload route resources, while a query cache may reuse data; coordinate their keys and lifecycles to avoid duplication.

### 322. What is a route-level code split?

It loads route code only when needed, reducing initial JavaScript at the cost of possible navigation latency.

### 323. How should navigation preserve scroll?

Use router scroll-restoration behavior and define exceptions for in-page anchors or interactions that should retain position.

### 324. What is a client-side navigation accessibility concern?

Update document title and manage focus or announce route changes so assistive technology users perceive navigation.

### 325. What is a route transition blocker?

A router feature that delays navigation until the user confirms or resolves unsaved changes.

### 326. How should unsaved changes be protected?

Warn on navigation where appropriate, but also save drafts or use durable server state when data loss would be serious.

### 327. What is a backend-for-frontend?

A server layer tailored to the UI that aggregates services and can centralize authentication and data shaping.

### 328. Why should API responses be validated?

Network data is untrusted and may not match compile-time types; validate critical shapes at runtime boundaries.

### 329. What is a typed API client?

It provides compile-time request and response contracts, but those types do not validate untrusted runtime payloads by themselves.

### 330. How do you choose between route loaders and client queries?

Use loaders for navigation-critical route data and query caches for shared, frequently updated client data; many apps combine them.

## 13. Server rendering, streaming, and hydration

### 331. What is client-side rendering?

The browser runs JavaScript to render the interface, typically starting from a minimal HTML document.

### 332. What is static site generation?

It produces HTML ahead of requests, with freshness and personalization handled through build, revalidation, or client updates.

### 333. What is request-time server rendering?

The server generates HTML for a particular request, often using request data, cookies, or current backend responses.

### 334. What is a hybrid rendering strategy?

It combines static, server-rendered, and client-rendered regions or routes according to their data and interaction needs.

### 335. What does React DOM's server renderer produce?

It serializes a React tree to HTML or a stream; framework integrations typically manage routing, data, and document concerns.

### 336. What is streaming SSR?

The server sends HTML progressively as parts of the rendered tree become ready instead of waiting for the entire page.

### 337. How does Suspense support streaming?

A server renderer can emit fallback HTML for pending boundaries and stream completed content later, when supported by the setup.

### 338. What is the benefit of streaming HTML?

It can deliver shell and ready content earlier, improving perceived progress while slower regions resolve.

### 339. What is a streaming shell?

It is the initial page structure sent before all deferred regions have completed rendering.

### 340. What is a Suspense reveal boundary on the server?

It divides streamed content into regions that can become available independently with their own fallback behavior.

### 341. What is hydration in React DOM?

Hydration attaches React's client behavior to compatible HTML previously rendered on the server.

### 342. Why must initial server and client output match?

React expects the same initial tree for hydration; mismatches can cause warnings, incorrect content, or client recovery work.

### 343. What commonly causes hydration mismatch?

Different data, locale/time output, random values, invalid HTML nesting, or browser-only conditions can make markup diverge.

### 344. How should time-dependent content be rendered?

Use a consistent server/client value or render a stable placeholder and update after hydration.

### 345. What does `suppressHydrationWarning` do?

It suppresses a narrow mismatch warning for an intentional difference; it does not repair divergent trees or data.

### 346. What is selective hydration?

React can prioritize hydration of particular server-rendered regions rather than requiring all regions to become interactive together.

### 347. What is partial hydration?

It is a broad architecture in which only selected page regions receive client JavaScript; exact support is framework-specific.

### 348. What is resumability?

It is an approach where serialized execution state lets a client resume without replaying a full component render; it is not synonymous with React hydration.

### 349. What is a React Server Component?

A Server Component runs on the server and can access server-side resources without shipping its component implementation to the browser.

### 350. Do Server Components replace client components?

No. Interactive state, event handlers, and browser APIs require client components; applications commonly compose both.

### 351. What marks a client module in React frameworks?

Frameworks commonly use a directive such as `"use client"` to mark a client module boundary; exact semantics are framework-defined.

### 352. What does `"use server"` mean?

In frameworks that implement Server Functions, it marks functions callable across a server boundary; React alone does not provide deployment semantics.

### 353. Can a Server Component use `useState`?

No. Server Components cannot use client-only interactive Hooks such as `useState` or Effects.

### 354. Can a Client Component import a Server Component directly?

Framework rules often require passing Server Component output through a supported composition boundary rather than importing server-only implementation into client code.

### 355. What is the RSC payload?

It is a framework transport representation of Server Component output and references used to compose server and client UI.

### 356. Are Server Components the same as SSR?

No. Server Components describe where components execute; SSR describes generating HTML, and frameworks may combine both.

### 357. What is a server-only module?

It contains code that must not enter a client bundle, such as secrets or privileged database access.

### 358. How should secrets be handled in React applications?

Keep them on trusted servers and never embed secret credentials in client bundles or rendered public data.

### 359. What is hydration cost?

It is the client CPU and JavaScript work required to make server-rendered regions interactive.

### 360. How can hydration work be reduced?

Reduce client component surface and shipped JavaScript, defer noncritical code, and prefer server-rendered static UI where suitable.

### 361. What is an island architecture?

It renders mostly static HTML with isolated interactive regions; React may be used for islands depending on the framework.

### 362. What is the role of `hydrateRoot`?

It hydrates server-generated DOM with a React tree and returns a root for client updates.

### 363. When should `createRoot` be used instead?

Use it for a client-rendered root without existing server HTML to hydrate.

### 364. What is the role of `renderToString`?

It creates an HTML string synchronously for supported server-rendering use cases but does not provide streaming.

### 365. What is `renderToPipeableStream`?

It is a Node.js streaming server API for React DOM that writes rendered output to a pipeable stream.

### 366. What is `renderToReadableStream`?

It is a Web Streams-based React DOM server API available in supported runtimes and React versions.

### 367. How should server rendering handle errors?

Use framework request and route error handling; streamed output may already be partially sent, limiting HTTP status changes.

### 368. Why avoid module-level request data on a server?

Server modules may be shared across requests; storing user-specific mutable data globally risks cross-request leaks.

### 369. What is a server component cache scope?

It determines which requests may reuse computed data; scope personalization and invalidation carefully to prevent cross-user exposure.

### 370. What framework caveat applies to SSR and RSC?

React APIs and framework conventions evolve independently; verify supported features, caching, and deployment behavior for the chosen versions.

## 14. Accessibility and security

### 371. Why prefer semantic HTML?

Native elements provide built-in meaning, keyboard behavior, and accessibility support that custom elements must otherwise recreate.

### 372. How should a form control get an accessible name?

Associate a visible `<label>` or provide a suitable accessible naming mechanism when visible text is not possible.

### 373. What is an accessible name?

It is the computed label assistive technology uses to identify a control or other named element.

### 374. How should buttons be chosen?

Use `<button>` for actions and `<a>` for navigation, preserving native semantics and keyboard behavior.

### 375. Why set a button's `type` in a form?

A button inside a form defaults to submit in HTML; `type="button"` prevents accidental submission for other actions.

### 376. What is focus management?

It ensures keyboard focus moves predictably after interactions such as opening a dialog, changing routes, or showing errors.

### 377. What should a modal dialog do with focus?

Move focus inside, keep interaction appropriately contained, support Escape where suitable, and restore focus to the opener.

### 378. What is a live region?

An element with suitable ARIA live semantics announces dynamic updates without requiring focus movement.

### 379. When should `aria-live` be used?

Use it for meaningful asynchronous status changes, not as a substitute for semantic markup or on every changing value.

### 380. What is keyboard operability?

All interactive functionality should be usable through keyboard input with visible, logical focus.

### 381. What is the purpose of heading hierarchy?

Headings provide a navigable document outline; choose levels based on structure, not visual size alone.

### 382. How should images receive alternative text?

Describe informative image purpose concisely; use empty alt text for purely decorative images.

### 383. What is a React XSS risk?

Rendering untrusted HTML through `dangerouslySetInnerHTML` can execute attacker-controlled markup or script.

### 384. How should raw HTML be rendered safely?

Avoid it when possible; otherwise sanitize with a maintained allowlist-based sanitizer and validate the trust boundary.

### 385. Does JSX escaping prevent every XSS attack?

It escapes ordinary text and attribute values, but unsafe URL schemes, raw HTML, and other injection contexts still need care.

### 386. How should URL props be validated?

Reject dangerous schemes and validate destinations before using untrusted URLs in links, redirects, or resource attributes.

### 387. What is a Content Security Policy?

A browser policy that restricts resource and script execution, helping reduce impact of injection vulnerabilities.

### 388. What is CSRF?

Cross-site request forgery tricks an authenticated browser into issuing an unwanted request; use appropriate server protections.

### 389. Does client-side route protection secure an API?

No. Authorization must be enforced by trusted server endpoints for every protected operation.

### 390. Why is client-side validation not security enforcement?

Users can alter client code or requests; the server must validate input and authorization independently.

### 391. How should authentication tokens be stored?

Choose storage based on threat model; HttpOnly secure cookies reduce direct script access, while CSRF and session controls remain necessary.

### 392. What is a dependency supply-chain risk?

A vulnerable or compromised package can run code in builds or users' browsers; minimize dependencies and maintain review/update practices.

### 393. Why avoid putting sensitive data in client state?

Client data can be inspected by users and extensions; treat every shipped value as public to that client.

### 394. How should error messages avoid leaking data?

Show actionable user-facing details while keeping secrets, stack traces, and internal infrastructure information in protected logs.

### 395. What is a clickjacking defense?

Server response policies such as CSP `frame-ancestors` can restrict which origins embed sensitive pages.

### 396. How should external links opened in a new tab be handled?

Use appropriate `rel` protections such as `noopener`; browsers mitigate some risks by default, but explicit intent is clearer.

### 397. What is an accessibility regression test?

It checks rendered behavior or semantics, ideally supplemented by keyboard and assistive-technology testing.

### 398. Can automated accessibility tools prove a page is accessible?

No. They catch some detectable issues but cannot replace manual keyboard, screen-reader, and usability evaluation.

### 399. Why should custom widgets follow ARIA patterns?

Correct roles, states, keyboard behavior, and focus management are necessary for assistive technology to interpret them.

### 400. What is the first choice for a custom control?

Prefer a native HTML control when it meets the need; it usually reduces accessibility and browser-compatibility risk.

## 15. Testing and performance

### 401. What is a unit test for a React component?

It checks a focused component behavior under selected inputs, generally through observable output or interaction.

### 402. What is an integration test?

It verifies several components and dependencies work together across a meaningful user flow.

### 403. What should a UI test assert?

Assert user-visible behavior, such as accessible content and interactions, rather than implementation details.

### 404. Why prefer role-based queries?

They reflect how users and assistive technologies perceive controls and encourage accessible markup.

### 405. What is a user-event testing utility?

It simulates realistic browser-like interactions, including focus and input behavior, more fully than dispatching isolated events.

### 406. What should be mocked in component tests?

Mock unstable external boundaries such as network or browser services, not the internal component behavior under test.

### 407. What is a test double?

A stub, fake, or mock stands in for a dependency to control or observe its behavior in a test.

### 408. Why use an MSW-style network mock?

It intercepts requests at a network boundary, allowing tests to exercise real application request logic.

### 409. What is an end-to-end test?

It exercises an application through a real or browser-automated environment across integrated UI and backend boundaries.

### 410. What should an end-to-end test avoid?

Avoid brittle timing sleeps and overly broad dependence on shared external systems; wait on meaningful user-visible conditions.

### 411. How should asynchronous UI be tested?

Wait for the expected accessible result and test both pending and settled states when those states matter.

### 412. How do you test an error boundary?

Render a descendant that throws during rendering and assert the boundary's fallback and recovery behavior.

### 413. How should Suspense be tested?

Use a supported suspending resource or lazy component and assert fallback, resolution, and relevant error behavior.

### 414. What is a fake timer useful for?

It makes timer-dependent behavior such as debounce deterministic without waiting for real elapsed time.

### 415. Why should tests clean up after themselves?

Unmounting components and restoring mocks, timers, and global state prevents leaks and order-dependent failures.

### 416. What is snapshot testing?

It compares serialized output to a stored baseline; keep snapshots focused and do not treat them as a substitute for behavior tests.

### 417. What is a flaky test?

A test whose result varies without relevant code changes, often due to timing, shared state, or uncontrolled dependencies.

### 418. How can flaky UI tests be reduced?

Use deterministic test data, wait on observable conditions, isolate state, and avoid arbitrary delays.

### 419. What is a visual regression test?

It compares rendered screenshots to detect unintended visual changes, with controlled fonts, viewport, and rendering environment.

### 420. How should React DevTools Profiler be used?

Record a representative interaction to identify which components render and where time is spent before optimizing.

### 421. What does a React Profiler render duration represent?

It estimates render work for a component subtree in a profiling build; it is not the entire user-perceived page cost.

### 422. What is the browser Performance panel useful for?

It reveals main-thread tasks, layout, paint, scripting, and network timing beyond React component rendering alone.

### 423. What is an unnecessary rerender?

A render that does work without changing useful output; profile to determine whether it materially affects responsiveness.

### 424. What is list virtualization?

It renders only visible or nearby rows of a large list, reducing DOM nodes and rendering work.

### 425. What tradeoffs does virtualization introduce?

It complicates accessibility, dynamic heights, focus, scrolling, and measurement, and may require careful overscan.

### 426. What is code splitting's performance tradeoff?

It reduces initial bundle transfer but can add chunk requests and delay a feature when first opened.

### 427. What is lazy component loading?

`React.lazy` loads a component module on demand and typically relies on a surrounding Suspense boundary.

### 428. How should bundle size be diagnosed?

Inspect production bundle analysis and identify large dependencies, duplicated modules, and code that ships unnecessarily.

### 429. What is tree shaking?

It is build-time removal of unused exports when module format and side-effect metadata permit safe elimination.

### 430. What is the largest-contentful-paint metric?

LCP measures when the largest visible content element is rendered; server delivery, image strategy, and blocking code can affect it.

### 431. What is interaction to next paint?

INP measures responsiveness across interactions by observing delays until the next visual update.

### 432. How can long React tasks be reduced?

Break up expensive work, virtualize large lists, avoid unnecessary synchronous computation, and schedule non-urgent updates appropriately.

### 433. Does memoization always improve performance?

No. Comparisons and cache maintenance cost work, and invalidation can make memoization ineffective.

### 434. What is a stable prop identity?

It is a reference that remains equal between renders when its semantic value is unchanged, relevant to shallow memo comparisons.

### 435. How can state locality improve performance?

Keeping state near the UI it affects prevents unrelated parts of the tree from rerendering on each update.

### 436. How can context update cost be reduced?

Split contexts by update frequency or use a store with focused subscriptions where profiling shows broad fan-out.

### 437. What is React Compiler?

It is a compiler that can automatically optimize supported React code; adoption, diagnostics, and behavior depend on version and configuration.

### 438. Should manual memoization be removed automatically?

Only after validating compiler support, lint guidance, and measured behavior; preserve memoization that has required semantic intent.

### 439. What is a production profiling build?

It is a profiling-enabled production-like build that records React timing with more realistic optimization behavior than development mode.

### 440. Why can development performance differ from production?

Development checks, Strict Mode, source maps, and unoptimized builds add overhead not representative of production.

### 441. What is a web worker useful for?

It moves CPU-intensive calculations off the main thread; React UI updates still need to be sent back to the browser thread.

### 442. How should expensive derived data be computed?

Measure it, cache only if worthwhile, or move heavy work to a worker or server when appropriate.

### 443. What is image loading performance?

It concerns choosing dimensions, formats, responsive sources, priority, and lazy loading according to image visibility and importance.

### 444. Why not lazy-load the above-the-fold hero image?

It may delay the largest content and worsen LCP; prioritize critical visible assets instead.

### 445. What is a resource hint?

Hints such as preload or preconnect tell the browser about likely-needed resources, but excessive hints compete for bandwidth.

### 446. How does CSS affect React performance?

Large style recalculation, layout thrashing, and animated layout properties can dominate costs independently of React reconciliation.

### 447. What is layout thrashing?

Alternating layout reads and writes can force repeated synchronous layout; batch measurements and DOM updates when using imperative code.

### 448. What is a performance budget?

It is a defined limit for metrics such as JavaScript size or interaction latency that teams monitor over time.

### 449. Why test production builds?

They expose bundling, environment, hydration, and optimization behavior that development servers can hide.

### 450. How do you prioritize performance work?

Use user impact and field or representative measurements, then optimize the measured bottleneck and verify the improvement.

## 16. Architecture, tooling, and version caveats

### 451. What is feature-based React architecture?

It groups components, state, and data logic around product capabilities rather than placing all files into generic technical folders.

### 452. What is a design system?

A shared set of accessible components, visual tokens, and interaction conventions that promotes consistency across an application.

### 453. What belongs in a shared component library?

Stable reusable primitives and documented patterns belong there; product-specific orchestration should remain near its feature.

### 454. What is a container-presentational split?

It separates data or orchestration from visual rendering, a useful option rather than a mandatory rule for every component.

### 455. What is a dependency boundary?

It defines which modules may depend on others, keeping features decoupled and limiting accidental cycles.

### 456. How should shared state ownership be chosen?

Put state at the narrowest owner that can coordinate all consumers, using URL, server cache, context, or a store as appropriate.

### 457. What is a domain model in a React app?

It represents product concepts and rules independently enough that UI components need not duplicate business logic.

### 458. Why separate data access from presentation?

It allows data policy and UI rendering to evolve and be tested independently, without requiring excessive abstraction.

### 459. What is a component contract?

It documents supported props, events, state ownership, accessibility behavior, and the component's guarantees.

### 460. How should a monorepo share React packages?

Align compatible React peer dependencies and bundler resolution to avoid duplicate React instances.

### 461. What causes the invalid Hook call warning?

Common causes include violating Hook rules, mismatched React and renderer versions, or multiple React copies in the bundle.

### 462. Why must React and its renderer versions be compatible?

The renderer integrates with React internals; unsupported version mismatches can break rendering or Hook behavior.

### 463. Why is having two React copies problematic?

Hooks rely on a shared React dispatcher, so components using a different copy from the renderer may fail.

### 464. What is the automatic JSX runtime?

It lets compilers transform JSX without requiring a `React` identifier in each module, when tooling is configured for it.

### 465. Does JSX itself require a browser?

No. JSX is compiled syntax; it can describe UI for browser, server, test, or other supported renderers.

### 466. What is React DOM?

React DOM is the renderer that integrates React with browser DOM nodes and provides server-rendering APIs.

### 467. What is React Native?

It is a React renderer and platform for native mobile UI; DOM elements and browser APIs do not apply directly.

### 468. Does React guarantee backward compatibility for every API?

Stable public APIs aim for compatibility, but experimental, canary, or framework-specific APIs can change and require version checks.

### 469. What is a React release channel?

Stable, canary, or experimental channels expose different maturity levels; production applications should use supported versions intentionally.

### 470. What is an experimental React API?

It is an API not yet guaranteed stable; verify its channel requirements and avoid assuming production support.

### 471. How does React 18 differ from React 19 for applications?

React 19 adds APIs and behaviors such as Actions and `use`; exact migration needs depend on renderer, framework, and installed versions.

### 472. What is the React 19 `ref` change?

Function components can receive `ref` as a prop in React 19; `forwardRef` remains relevant for older versions and compatibility.

### 473. What are Actions in React 19?

Actions are async functions integrated with transition and form workflows; framework support determines how server execution is wired.

### 474. What is the React Compiler version caveat?

Compiler support is toolchain-dependent and evolves; follow its matching version and diagnostics rather than assuming automatic optimization.

### 475. What is a framework's role in a React app?

A framework may supply routing, bundling, SSR, data APIs, server components, and deployment conventions beyond React core.

### 476. Why should framework documentation accompany React documentation?

Framework behavior such as caching, Server Functions, and route rendering is not specified by React alone.

### 477. What is a bundler's role?

A bundler resolves modules, transforms JSX and assets, splits code, and emits deployable client or server artifacts.

### 478. What is a JSX transform configuration?

It tells the compiler how to translate JSX and which runtime or import source to target.

### 479. What is a lint rule for React Hooks?

It statically checks Hook call order and dependency completeness, catching patterns that are difficult to diagnose at runtime.

### 480. What is TypeScript's role in React?

It checks component contracts and data flow at compile time but cannot validate untrusted runtime inputs automatically.

### 481. What is a discriminated union for component props?

It models mutually exclusive prop variants through a shared tag, allowing TypeScript to reject invalid combinations.

### 482. How should component types be defined in TypeScript?

Prefer explicit prop interfaces or types and infer JSX return types where practical instead of over-constraining every component.

### 483. What is `ReactNode`?

It is a type for values React can render as children; exact type definitions vary with React TypeScript packages.

### 484. How should event types be selected in TypeScript?

Use the event type matching the element and handler, such as a change event for the relevant form control.

### 485. What is a peer dependency for React?

A library declares React as a peer so the application supplies a compatible shared runtime rather than bundling a separate copy.

### 486. How should React upgrades be approached?

Review release and framework migration notes, upgrade compatible packages, run type checks and tests, then validate production rendering.

### 487. What is a codemod?

A codemod automates mechanical source transformations, but its output still needs review and tests.

### 488. What is a feature flag in a React application?

A flag controls feature availability; evaluate authorization and sensitive decisions on the server rather than trusting client flags.

### 489. How can architecture support testing?

Expose clear boundaries for network, time, navigation, and storage dependencies so tests can substitute controlled implementations.

### 490. What is dependency inversion in UI architecture?

High-level UI logic depends on stable interfaces rather than concrete infrastructure, allowing implementations to be replaced or tested.

### 491. What is an anti-corruption boundary for API data?

It converts external payloads into application domain shapes, isolating inconsistent backend contracts from UI components.

### 492. Why keep business rules out of JSX expressions?

Named functions or domain modules make complex rules easier to test, review, and reuse than dense inline conditions.

### 493. What is a feature boundary's error policy?

It defines how failures are surfaced and recovered locally, including retry, fallback, and reporting behavior.

### 494. What is observability in a React application?

It combines user-impact metrics, error reports, and traces to diagnose production behavior while respecting privacy.

### 495. How should client errors be reported?

Capture useful context and source maps securely, redact sensitive values, and avoid collecting unnecessary personal data.

### 496. What is a deployment compatibility concern for SSR?

The server runtime, client bundle, and cached HTML must use compatible builds and data contracts during rollout.

### 497. How can rolling deployments affect hydration?

Old cached HTML may load new client code or vice versa; versioned assets and compatible deployment strategies reduce mismatches.

### 498. What is the safest way to evaluate an evolving React feature?

Check official React and framework documentation for the exact release channel and test it in the actual target runtime.

### 499. How should React knowledge be applied across versions?

Separate stable concepts from release-specific APIs, and verify current behavior against the project's installed React and renderer.

### 500. What is the core principle behind maintainable React systems?

Keep rendering pure, state ownership explicit, boundaries accessible, and platform-specific behavior verified against measured needs and supported APIs.
