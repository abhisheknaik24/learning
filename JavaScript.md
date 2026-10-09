# JavaScript Interview Questions and Answers

> JavaScript behavior can differ between browser and server runtimes, and language features depend on the supported ECMAScript version. This guide focuses on modern JavaScript while explaining foundational semantics.

## Contents

1. [Language fundamentals and types](#1-language-fundamentals-and-types)
2. [Scope, functions, and objects](#2-scope-functions-and-objects)
3. [Arrays, collections, and modules](#3-arrays-collections-and-modules)
4. [Asynchronous JavaScript](#4-asynchronous-javascript)
5. [Browser APIs and security](#5-browser-apis-and-security)
6. [Testing and coding questions](#6-testing-and-coding-questions)

## 1. Language fundamentals and types

### 1. What is JavaScript?

JavaScript is a high-level, dynamically typed language standardized as ECMAScript. It is used in browsers and server runtimes such as Node.js. The language defines core syntax and semantics; host environments provide APIs such as the DOM, timers, filesystem, and networking.

### 2. What are JavaScript's primitive types?

The primitive types are `undefined`, `null`, `boolean`, `number`, `bigint`, `string`, and `symbol`. Objects, including arrays and functions, are reference values. `NaN` is a special numeric value, not a separate type.

### 3. What is the difference between `null` and `undefined`?

`undefined` commonly means a value has not been assigned or a property is absent. `null` is an explicit absence value. Historically, `typeof null` returns `"object"`; this is a language quirk. Use strict equality and establish consistent conventions for optional data.

### 4. What is the difference between `==` and `===`?

`===` compares without type coercion (except that `NaN !== NaN`; use `Number.isNaN`). `==` applies coercion rules that can produce surprising results, such as `0 == false`. Prefer strict equality unless a specific coercion is deliberately required.

### 5. What is truthy and falsy?

Falsy values include `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`. Most other values—including empty arrays and empty objects—are truthy. Truthiness is used by conditionals and logical operators, but it can conflate meaningful zero/empty values with absence.

### 6. What is `NaN`?

`NaN` means “not a number” and is a numeric value resulting from invalid numeric operations. It is unequal to every value, including itself. Use `Number.isNaN(value)` for a non-coercing check; global `isNaN` coerces its argument.

### 7. What is the difference between `var`, `let`, and `const`?

`var` is function-scoped (or global-scoped in scripts) and can be redeclared. `let` and `const` are block-scoped and have a temporal dead zone before declaration. `const` prevents reassignment of the binding, not mutation of an object it references. Prefer `const` by default, then `let` when reassignment is needed.

### 8. What is hoisting?

Hoisting describes declaration processing before code execution. Function declarations are generally callable before their textual location. `var` declarations are initialized to `undefined`; `let` and `const` bindings exist but cannot be accessed before initialization due to the temporal dead zone. Thinking of all declarations as simply “moved to the top” is misleading.

### 9. What is type coercion?

Type coercion converts a value to another type, either implicitly through operators or explicitly through functions such as `Number()` and `String()`. Implicit coercion can make expressions surprising; explicit conversion communicates intent. Validate inputs because conversions may accept forms different from a business rule.

### 10. What is the difference between `Number` and `BigInt`?

`Number` uses IEEE-754 double precision and cannot exactly represent every large integer. `BigInt` represents arbitrary-precision integers within implementation limits and uses an `n` suffix such as `123n`. Arithmetic generally cannot mix `BigInt` and `Number` without explicit conversion. JSON does not serialize BigInt by default.

## 2. Scope, functions, and objects

### 11. What is lexical scope?

Lexical scope means variable visibility is determined by where code is written. Inner functions can access variables in enclosing scopes. This enables closures and gives predictable name resolution independent of the function's eventual caller.

### 12. What is a closure?

A closure is a function together with access to variables from its lexical environment. The function can continue using those bindings after the outer function returns. Closures are useful for callbacks and encapsulation, but captured references can extend object lifetimes or retain more memory than intended.

### 13. What is the difference between a function declaration and function expression?

A function declaration defines a named function and is available according to declaration-hoisting rules. A function expression creates a function value assigned or passed as an expression; whether it is callable before the assignment depends on the binding declaration. Arrow functions are a concise expression form with lexical `this`.

### 14. How does `this` work?

For ordinary functions, `this` is determined by the call form (method call, constructor call, explicit binding, or other rules). Arrow functions capture `this` lexically from the surrounding context and do not have their own `this`. Avoid assuming `this` remains bound when extracting a method; use binding, a wrapper, or an arrow where suitable.

### 15. What are `call`, `apply`, and `bind`?

`call` invokes a function with a specified `this` and individual arguments; `apply` invokes with an argument array-like value; `bind` returns a new function with `this` and optionally initial arguments fixed. Arrow functions' lexical `this` cannot be changed by these methods.

### 16. What is prototypal inheritance?

Objects can inherit properties and methods through a prototype chain. When a property is not found on an object, JavaScript checks its prototype, continuing up the chain. `class` syntax provides a more familiar abstraction over prototype-based behavior; it does not change the underlying prototype model.

### 17. What is the difference between `Object.create` and a class?

`Object.create(proto)` creates an object with a specified prototype (or no prototype with `null`). A class defines a constructor and prototype methods with class-specific semantics. Choose class syntax for conventional instance types; use `Object.create` for explicit prototype construction or dictionaries only when its behavior is intended.

### 18. What are rest and spread syntax?

Rest syntax gathers remaining values into an array or object; spread syntax expands iterable or enumerable properties into another expression. Array/object spread creates a shallow copy, not a deep clone. Object spread copies enumerable own properties and does not preserve prototypes or property descriptors.

### 19. What are default parameters?

Default parameter expressions are used when an argument is `undefined` or omitted, but not when it is `null`. They are evaluated when the function is called. Defaults can simplify API setup; avoid surprising side effects in default expressions.

### 20. What is a pure function?

A pure function returns a result determined by its inputs and has no observable side effects. Pure functions are easier to test, cache, and reason about. They are a design property, not a special JavaScript syntax.

## 3. Arrays, collections, and modules

### 21. What is the difference between `map`, `filter`, and `reduce`?

`map` transforms every element into a new array; `filter` keeps elements meeting a predicate; `reduce` accumulates values into one result. These methods generally do not mutate the source array themselves, though callbacks can mutate external state. Use the clearest operator for the operation.

### 22. What is the difference between `forEach` and `map`?

`forEach` invokes a callback for side effects and returns `undefined`. `map` returns a new array of callback results. Do not use `map` only for side effects, and do not assume `forEach` can be stopped with `break`.

### 23. What is a shallow copy?

A shallow copy duplicates the outer container but retains references to nested objects. For example, `[...items]` creates a new array but nested objects remain shared. Deep copying requires a deliberate strategy based on data shape; `structuredClone` supports many values but not every object or host resource.

### 24. How do `Map` and `Set` differ from plain objects and arrays?

`Map` stores key/value pairs with keys of any value type and preserves insertion order. `Set` stores unique values. Plain objects are useful for records with string/symbol keys but inherit object behavior unless created with a null prototype. Arrays are ordered indexed sequences and can contain duplicates.

### 25. What are ES modules?

ES modules use `import` and `export` to declare dependencies and public exports. They have module scope and are loaded according to host and bundler rules. Static import declarations enable tooling and tree-shaking; dynamic `import()` loads modules asynchronously at runtime.

### 26. What is the difference between named and default exports?

A module can have multiple named exports and at most one default export. Named imports identify the exported binding by name; default imports choose a local name. Follow consistent project conventions because default exports can make symbol renaming and search less predictable.

### 27. What is JSON serialization?

`JSON.stringify` converts supported JavaScript values to JSON text; `JSON.parse` parses JSON text into data. Functions, symbols, and `undefined` are omitted or transformed in context-dependent ways; cyclic references throw. JSON is not a general-purpose deep clone or secure validation mechanism.

## 4. Asynchronous JavaScript

### 28. What is the event loop?

The event loop coordinates JavaScript execution with queued work. In browsers, tasks and microtasks are processed under host scheduling rules; promise reactions are microtasks and usually run before the next task. JavaScript execution in a given agent is run-to-completion, but browser APIs and workers provide asynchronous or parallel capabilities.

### 29. What is a Promise?

A Promise represents the eventual fulfillment or rejection of an asynchronous operation. It transitions from pending to fulfilled or rejected once. `.then`, `.catch`, and `.finally` return chains; errors thrown in callbacks generally reject the next promise.

### 30. What do `async` and `await` do?

An `async` function always returns a Promise. `await` pauses that function's continuation until a value or promise settles, without blocking the JavaScript thread. Use `try/catch` for rejection handling. `await` does not itself make synchronous CPU work non-blocking.

### 31. What is the difference between `Promise.all` and `Promise.allSettled`?

`Promise.all` fulfills when all inputs fulfill and rejects as soon as one rejects; other operations are not automatically canceled. `Promise.allSettled` waits for all inputs and returns each fulfillment/rejection outcome. Use `Promise.race` for the first settled promise and `Promise.any` for the first fulfilled one, understanding their distinct failure behavior.

### 32. How do you run asynchronous work sequentially versus concurrently?

Sequential work awaits each operation inside a loop. Concurrent work starts operations before awaiting their combined results, often using `Promise.all`. Concurrency can overload a service or exhaust resources; use bounded concurrency for large inputs and preserve required ordering or failure semantics.

### 33. What is a callback?

A callback is a function passed to another operation to be invoked later, such as on completion or an event. Deeply nested callbacks can make error and sequencing logic difficult; promises and async/await provide compositional alternatives.

### 34. How do you handle errors in asynchronous code?

Attach `.catch` to promise chains or use `try/catch` around awaited operations. Ensure rejected promises are observed; unhandled rejections can cause runtime warnings or termination depending on host policy. Add context and recover only when recovery is valid.

### 35. What is a race condition in JavaScript?

A race occurs when asynchronous operations complete in an order different from the order they started, causing stale or conflicting results. For example, an older search request may overwrite newer results. Use cancellation, request identifiers, sequencing, or state checks to ensure only relevant completions update state.

### 36. What is `fetch`?

`fetch` is a promise-based API for HTTP requests in browsers and some server runtimes. It rejects for certain network failures, but an HTTP error status such as 404 generally still fulfills with a `Response`; check `response.ok` or `status`. Parse the body asynchronously and handle cancellation with `AbortController` where supported.

## 5. Browser APIs and security

### 37. What is the DOM?

The Document Object Model is a host API representing an HTML/XML document as nodes. JavaScript can query and modify it. DOM APIs are browser-provided, not part of the ECMAScript language specification itself.

### 38. What is event bubbling and delegation?

Many DOM events propagate from the target up through ancestors (bubbling). Event delegation attaches one handler to a common ancestor and checks the event target, which can handle many descendants efficiently and also work for dynamically added children. Some events do not bubble or have special behavior.

### 39. What is the difference between `localStorage`, `sessionStorage`, and cookies?

Web storage APIs store string data accessible to same-origin scripts; `localStorage` generally persists beyond a tab session, while `sessionStorage` is scoped to a page session/tab context. Cookies are sent with matching HTTP requests and can have security attributes such as `HttpOnly`, `Secure`, and `SameSite`. Do not store sensitive credentials in script-accessible storage without a carefully reviewed threat model.

### 40. What is a cross-site scripting (XSS) vulnerability?

XSS occurs when attacker-controlled content is executed as script in another user's browser. Avoid unsafe HTML insertion, encode output for its context, sanitize rich content with a trusted sanitizer, and use a Content Security Policy as defense in depth. Client-side validation alone does not prevent XSS.

### 41. What is same-origin policy?

The same-origin policy restricts how documents and scripts from one origin interact with resources from another. An origin is based on scheme, host, and port. CORS allows servers to selectively relax some cross-origin read restrictions; it is not an authentication mechanism.

### 42. What is event debouncing versus throttling?

Debouncing delays execution until events stop arriving for a specified interval, useful for search input. Throttling limits execution to at most a chosen frequency, useful for scroll or resize work. Both need careful handling of trailing/leading calls and cleanup.

## 6. Testing and coding questions

### 43. How do `slice` and `splice` differ?

`slice` returns a shallow copy of a portion without modifying the original array. `splice` modifies the original by removing, replacing, or inserting elements. The similar names are easy to confuse.

### 44. What is the output?

```javascript
console.log(typeof null);
```

**Answer:** `"object"`. This is a historical JavaScript behavior; test null with `value === null`.

### 45. What is the output?

```javascript
console.log(0 == false, 0 === false);
```

**Answer:** `true false`. Loose equality coerces operands; strict equality does not.

### 46. What is the output?

```javascript
const obj = { count: 1 };
obj.count++;
console.log(obj.count);
```

**Answer:** `2`. `const` prevents rebinding `obj`, not mutation of the referenced object. For immutable state, create a new object instead.

### 47. What is the output?

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

**Answer:** Usually `3` printed three times after the synchronous loop, because `var` is function-scoped and each callback closes over the same binding. Using `let i` creates a per-iteration binding.

### 48. How do you remove duplicates from an array of primitives?

For values with `Set` equality semantics:

```javascript
const unique = [...new Set(values)];
```

For objects, this deduplicates by object identity, not by matching properties; use a deliberate key or comparison strategy for structural uniqueness.

### 49. How do you safely access a nested optional property?

Use optional chaining and a fallback if appropriate:

```javascript
const city = user?.address?.city ?? "Unknown";
```

Optional chaining avoids a nullish access failure but should not hide a violated invariant where the value is required.

### 50. How should JavaScript code be tested?

Use unit tests for deterministic functions, component/integration tests for interactions, and end-to-end tests for critical user flows. Control time, network, and other nondeterministic dependencies. Test observable behavior and meaningful edge cases, not only implementation details.
