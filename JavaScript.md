# JavaScript Interview Questions and Answers

> JavaScript behavior can differ between browser and server runtimes, and language features depend on the supported ECMAScript version. This guide focuses on modern JavaScript while explaining foundational semantics.

## Contents

1. [Language fundamentals and types](#1-language-fundamentals-and-types)
2. [Scope, functions, and objects](#2-scope-functions-and-objects)
3. [Arrays, collections, and modules](#3-arrays-collections-and-modules)
4. [Asynchronous JavaScript](#4-asynchronous-javascript)
5. [Browser APIs and security](#5-browser-apis-and-security)
6. [Testing and coding questions](#6-testing-and-coding-questions)
7. [Execution and language semantics](#7-execution-and-language-semantics)
8. [Conversions, equality, and operators](#8-conversions-equality-and-operators)
9. [Bindings, scope, and modules](#9-bindings-scope-and-modules)
10. [Functions and call behavior](#10-functions-and-call-behavior)
11. [Objects, properties, and reflection](#11-objects-properties-and-reflection)
12. [Classes and inheritance](#12-classes-and-inheritance)
13. [Arrays, iterators, and generators](#13-arrays-iterators-and-generators)
14. [Collections and data structures](#14-collections-and-data-structures)
15. [Modules and packaging](#15-modules-and-packaging)
16. [Promises and asynchronous patterns](#16-promises-and-asynchronous-patterns)
17. [Event loop and concurrency](#17-event-loop-and-concurrency)
18. [Browser APIs and platform behavior](#18-browser-apis-and-platform-behavior)
19. [Node.js and runtime differences](#19-nodejs-and-runtime-differences)
20. [Security and robustness](#20-security-and-robustness)
21. [Performance and memory](#21-performance-and-memory)
22. [Testing and developer tooling](#22-testing-and-developer-tooling)
23. [Modern ECMAScript features](#23-modern-ecmascript-features)
24. [Advanced coding scenarios](#24-advanced-coding-scenarios)

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

## 7. Execution and language semantics

### 51. What is an execution context?
An execution context is the specification-defined state used while evaluating code, including its lexical environment, variable environment, and `this` binding. Calls create function contexts; scripts and modules have their own top-level contexts.

### 52. What is an environment record?
An environment record stores identifier bindings for a lexical scope. Identifier lookup searches the current record and then its outer environment, implementing lexical name resolution.

### 53. What is the difference between a script and a module?
A script runs in a script context and may expose top-level `var` declarations on the global object. A module is strict by default, has module-scoped top-level bindings, and supports static imports and exports.

### 54. What does strict mode change?
Strict mode disallows or changes certain error-prone behaviors, such as accidental global assignments and some silent failures. Modules and class bodies are strict automatically.

### 55. What is an ECMAScript specification?
It is the normative language specification defining JavaScript syntax, types, algorithms, and observable behavior. Hosts such as browsers and Node.js supply additional APIs.

### 56. What is the difference between a language feature and a host API?
A language feature is standardized by ECMAScript, such as `Promise` or `Map`; a host API is provided by the environment, such as `document`, `fetch`, or Node's filesystem module.

### 57. What is a completion record?
It is an abstract specification value describing how an operation finishes: normally, by returning, by throwing, or through control flow such as `break`. It explains how abrupt completion propagates.

### 58. What does `return` do inside `try...finally`?
`finally` runs before control leaves the `try`. A `return` or `throw` from `finally` overrides a pending return or exception, so avoid control-flow statements there.

### 59. What is the difference between an expression and a statement?
An expression evaluates to a value; a statement performs an action or controls execution. Syntax such as function declarations and expressions differs by syntactic context.

### 60. How does automatic semicolon insertion work?
The parser inserts semicolons in certain restricted line-terminator cases, but not wherever one might expect. Keep statement starts unambiguous and avoid a line break after `return` when returning an expression.

### 61. What is an operator's precedence?
Precedence determines how operators group in an expression; associativity resolves grouping among equal-precedence operators. Parentheses clarify non-obvious intent.

### 62. What is short-circuit evaluation?
`&&`, `||`, and `??` evaluate the right operand only when needed and return operand values, not necessarily booleans. This supports conditional evaluation but can obscure intent if overused.

### 63. How do `&&` and `||` differ from `??`?
`&&` and `||` branch on truthiness; `??` branches only when the left side is `null` or `undefined`. Use `??` when valid values such as `0`, `false`, or `""` must be retained.

### 64. What is the conditional operator?
`condition ? a : b` evaluates the condition and then exactly one branch, returning that branch's value. Nested ternaries usually reduce readability.

### 65. What do `break` and `continue` do?
`break` exits the innermost loop or switch, while `continue` skips to the next loop iteration. Labels can target enclosing statements, but are rarely needed.

### 66. How does `switch` compare cases?
Case expressions are compared using strict equality semantics, without coercion. A matching case falls through until `break` or the end, so add breaks when fallthrough is unintended.

### 67. What does the comma operator do?
It evaluates expressions from left to right and yields the final expression's value. It is distinct from commas used as separators and is uncommon in clear application code.

### 68. What is the `void` operator?
`void expression` evaluates its operand and returns `undefined`. It has specialized uses; ordinary code should usually use an explicit `undefined` value or ignore the result.

### 69. What is `typeof` useful for?
It returns a string identifying a value's broad primitive category or `"function"` for callable objects. It reports `"object"` for `null` and many objects, so it is not a general classifier.

### 70. How can an object's class tag be misleading?
`Object.prototype.toString` can be affected by `Symbol.toStringTag` and proxies, so it is not secure proof of an object's true type. Prefer explicit validation or robust brand checks.

### 71. What does `instanceof` test?
Ordinarily it checks whether a constructor's `prototype` occurs in the object's prototype chain. It can be customized with `Symbol.hasInstance` and can fail across realms.

### 72. What is an intrinsic object?
An intrinsic is a built-in object or function specified by ECMAScript, such as the current realm's `%Array.prototype%`. Each realm has its own intrinsics, affecting identity checks.

### 73. What is a realm?
A realm is an execution environment with its own global object and built-ins. Browser frames commonly have separate realms; cross-realm objects may fail local `instanceof` checks.

### 74. What is a property key?
A property key is either a string or a symbol. Most other values used as keys are converted to strings, so object keys can collide after coercion.

### 75. What is the difference between own and inherited properties?
An own property belongs directly to the object; an inherited property is found on its prototype chain. `Object.hasOwn(obj, key)` checks own presence without consulting the chain.

## 8. Conversions, equality, and operators

### 76. How does string conversion differ from JSON serialization?
`String(value)` applies string conversion, while `JSON.stringify` serializes supported data structures and omits or transforms certain values. They serve different purposes and neither is a general deep-copy method.

### 77. How does numeric conversion of an empty string behave?
`Number("")` and `Number("   ")` produce `0`, while `parseInt("")` produces `NaN`. Validate the input format separately when blank input should be rejected.

### 78. What is the difference between `parseInt` and `Number`?
`parseInt` reads an integer prefix and stops at invalid trailing text; `Number` requires the whole trimmed string to represent a number. Pass a radix to `parseInt` when parsing textual integers.

### 79. Why is `parseInt("15px")` different from `Number("15px")`?
`parseInt` accepts the leading integer and returns `15`; `Number` rejects the trailing characters and returns `NaN`. Neither behavior replaces domain-specific validation.

### 80. What are `Number.isFinite` and global `isFinite` differences?
`Number.isFinite` accepts only finite values already of type `number`; global `isFinite` first coerces its argument. The non-coercing form is usually safer for validation.

### 81. What is negative zero?
`-0` is a distinct IEEE-754 value that compares equal to `0` with `===`, but affects operations such as reciprocal (`1 / -0` is `-Infinity`). Use `Object.is` to distinguish it.

### 82. How does `Object.is` differ from `===`?
`Object.is` treats `NaN` as equal to itself and distinguishes `+0` from `-0`; otherwise its comparison is similar to strict equality. It does not perform deep comparison.

### 83. What is SameValueZero?
SameValueZero equates `NaN` with itself and considers positive and negative zero equal. It is used by `Set`, `Map` keys, and `Array.prototype.includes`.

### 84. Why can `0.1 + 0.2` be imprecise?
Binary floating-point cannot exactly represent many decimal fractions, so arithmetic rounds. Use integer minor units for money or an appropriate decimal arithmetic library.

### 85. What is the remainder operator's sign behavior?
`%` returns a remainder whose sign follows the dividend, unlike modulo conventions in some languages. Normalize explicitly when a non-negative modulo is required.

### 86. What does the unary `+` operator do?
It converts its operand to a number using numeric coercion. It is concise but can accept surprising inputs; validate acceptable input forms separately.

### 87. What does the `in` operator check?
`key in object` tests whether the property exists on the object or anywhere on its prototype chain. Use `Object.hasOwn` if inherited properties should not count.

### 88. What does `delete` do?
It attempts to remove an object's configurable own property and returns a boolean for the operation. It does not free memory directly, and deleting an array element leaves a hole.

### 89. What is the difference between `++x` and `x++`?
Both increment `x`; prefix form evaluates to the updated value and postfix form to the previous value. Prefer standalone increments when the expression value is irrelevant.

### 90. How does bitwise arithmetic treat numbers?
Most bitwise operators convert `Number` operands to signed 32-bit integers, discarding higher bits and fractions. They are unsuitable for arbitrary-size integers without `BigInt`.

### 91. What is the nullish coalescing assignment operator?
`x ??= value` assigns only if `x` is `null` or `undefined`, evaluating the target once. It preserves other falsy values.

### 92. What do logical assignment operators do?
`&&=`, `||=`, and `??=` conditionally assign based on truthiness or nullishness. They combine short-circuit evaluation with assignment.

### 93. What is destructuring assignment?
It extracts values from arrays or properties from objects into bindings or assignment targets. Defaults apply to `undefined`, and destructuring can throw if its source is nullish.

### 94. How do object property keys convert?
Computed property names are converted to strings or symbols. An object used as a key is usually stringified, often becoming `"[object Object]"`; use `Map` for identity keys.

### 95. What is the difference between `in` and `Object.hasOwn`?
`in` includes inherited properties; `Object.hasOwn` checks only direct properties. This matters when processing untrusted objects or objects with custom prototypes.

### 96. What is a sparse array?
A sparse array has missing indexed properties (holes), distinct from present properties whose value is `undefined`. Many array methods skip holes, while others visit them as `undefined`.

### 97. How does `Array.prototype.includes` handle `NaN`?
It finds `NaN` because it uses SameValueZero comparison. `indexOf(NaN)` does not find it because it uses strict equality semantics.

### 98. Why can addition concatenate instead of add?
`+` converts operands to primitives, then concatenates if either primitive is a string; otherwise it performs numeric or BigInt addition. Normalize operands when numeric intent matters.

### 99. How are relational string comparisons performed?
When both operands become strings, comparison uses UTF-16 code-unit ordering, not locale-aware alphabetical order. Use `Intl.Collator` for human-language sorting.

### 100. What is explicit conversion?
It is deliberate conversion such as `Number(text)`, `String(value)`, or `Boolean(value)`. It makes intent visible but must still be paired with validation.

## 9. Bindings, scope, and modules

### 101. What is the temporal dead zone?
It is the interval from entering a lexical scope until a `let`, `const`, or class binding is initialized. Reading the binding in that interval throws a `ReferenceError`.

### 102. Can a `const` binding be reassigned?
No. Its binding cannot be reassigned, though the referenced object's mutable properties may change. `const` does not make values deeply immutable.

### 103. What happens when a `let` declaration is redeclared in one scope?
The program is a syntax error. A nested block may declare a new binding that shadows the outer one.

### 104. What is variable shadowing?
An inner binding with the same name hides an outer binding within its scope. Shadowing can help local clarity but may obscure which value a reference denotes.

### 105. What is a global lexical binding?
It is a top-level `let`, `const`, or class binding in a script's global environment. Unlike top-level `var` in a classic script, it is not necessarily a global-object property.

### 106. How does top-level `this` differ in scripts and modules?
In a classic script, top-level `this` is generally the global object; at module top level it is `undefined`. Function-level `this` follows its own call rules.

### 107. What does `globalThis` provide?
It is a standard way to access the current realm's global object across environments. It does not make browser-only or Node-only APIs portable.

### 108. What is an immediately invoked function expression?
An IIFE is a function expression called immediately, creating a local scope. Modules and block scopes make many historical IIFE uses unnecessary.

### 109. How can closures support private state?
A function can close over local bindings and expose operations that use them without exposing the bindings. Private class fields offer a syntax-level alternative.

### 110. What is closure-based state encapsulation's trade-off?
Each factory invocation may create new functions, while closed-over values remain private. Prototype methods share function objects but require a different privacy strategy.

### 111. What is a module's live binding?
An imported binding reflects updates to the exporting module's binding; it is not a copied value. Importers cannot assign to that binding.

### 112. Why are imports restricted to module top level?
Static imports are analyzed before module evaluation, enabling dependency linking and optimization. Use `import()` for conditional or deferred loading.

### 113. What is a cyclic module dependency?
It is a cycle in the module dependency graph. Modules are linked before evaluation, but early access can encounter an uninitialized lexical binding.

### 114. What is name shadowing in a module?
A local declaration can shadow an imported or outer name in its scope. Imports are immutable bindings; aliases make dependencies explicit and avoid conflicts.

### 115. What does `eval` do to scope?
Direct `eval` can evaluate code in the caller's lexical environment in non-strict contexts, complicating optimization and security. Indirect `eval` runs in global scope; avoid both for dynamic data.

### 116. Why is the `Function` constructor risky?
It compiles a string as code in global scope, can enable code injection, and may violate Content Security Policy. Prefer functions and structured data over generated source.

### 117. What is lexical environment capture in an arrow function?
An arrow function captures surrounding `this`, `arguments`, `super`, and `new.target` where applicable. It has no own `this` or `arguments` binding.

### 118. Does a block always create a new scope?
A block creates a lexical scope for `let`, `const`, and class declarations, but `var` is not block-scoped. Some statement forms have additional declaration rules.

### 119. What is the scope of a `catch` parameter?
The exception parameter is scoped to the catch clause. It can have a name independent of outer bindings, though shadowing remains a readability concern.

### 120. What is a module namespace object?
It is an immutable, module-branded object exposing a module's exports as read-only live properties. It is returned by `import()` and can be created with `import * as`.

### 121. What does `arguments` contain?
In ordinary functions it is an array-like object containing supplied arguments. Arrow functions inherit any surrounding `arguments`; use rest parameters for a real array.

### 122. Why prefer rest parameters over `arguments`?
Rest parameters are an actual array, clearly declared in the signature, and work in arrow functions. They avoid legacy argument aliasing behavior in some non-strict functions.

### 123. Can a closure observe later changes to an outer variable?
Yes. A closure retains access to the binding, not a snapshot of its value, so later assignments are visible.

### 124. What is accidental global state?
It is state stored on a shared global object or leaked by undeclared assignment. It creates collisions and hidden dependencies; strict mode makes many undeclared assignments throw.

### 125. How should module-level mutable state be handled?
Keep it private where possible, expose intentional operations, and document lifecycle assumptions. Shared mutable exports can make tests order-dependent.

## 10. Functions and call behavior

### 126. What is a higher-order function?
A function that accepts functions as arguments, returns functions, or both. Callbacks, decorators, and functional composition are common examples.

### 127. What is a callback?
A function passed to another operation to be invoked later or at a defined point. Document whether it may run synchronously, multiple times, or with a particular `this`.

### 128. What is an error-first callback?
It is a Node-style convention where the first callback argument is an error or null, followed by result values. Promises are a composable alternative for many asynchronous APIs.

### 129. What does `new` do?
It creates an object linked to the constructor's `prototype`, calls the constructor with that object as `this`, and returns it unless the constructor returns another object.

### 130. What is a constructor function?
A function intended to be invoked with `new`, typically initializing instance state. Classes enforce construction and cannot be called without `new`.

### 131. What is a bound function's `length`?
Binding can reduce a function's reported `length` by the number of preset arguments, down to zero. `length` and `name` are metadata, not reliable API validation.

### 132. What happens when a bound function is used with `new`?
The bound `this` is ignored for construction, while preset arguments are still applied. A bound constructor can inherit constructability from its target.

### 133. What is currying?
Currying transforms a function of multiple arguments into a chain of functions each taking fewer arguments. It can support composition but should suit the API's readability.

### 134. What is partial application?
It creates a function with some arguments pre-filled while leaving others to be supplied later. `bind` provides a built-in form for ordinary functions.

### 135. What is function composition?
It combines functions so the output of one becomes the input of another. Define direction and error behavior clearly, especially for asynchronous functions.

### 136. What does `Function.prototype.toString()` guarantee?
For ECMAScript function objects it returns source text representation subject to specification rules, but it is not a reliable way to inspect behavior or generate portable code.

### 137. What is a generator function?
A `function*` call returns an iterator whose execution can pause at `yield` and resume on subsequent calls. It can model lazy sequences and cooperative workflows.

### 138. What does `yield*` do?
It delegates iteration to another iterable or generator and forwards yielded values and the completion value. It can simplify composition of generator sequences.

### 139. What is an async function?
It always returns a promise. A returned value fulfills that promise; a thrown error rejects it, and `await` pauses the function until a value settles.

### 140. What is an async generator?
An `async function*` produces an asynchronous iterator, where each `next()` returns a promise for an iterator result. It suits streamed or paginated asynchronous data.

### 141. What is function arity?
It commonly refers to the number of declared parameters, reflected approximately by `function.length`. Defaults and rest parameters affect it; it does not restrict supplied arguments.

### 142. Are JavaScript functions first-class values?
Yes. Functions can be assigned, stored, passed, and returned like other values, though their call behavior and properties make them callable objects.

### 143. What is a method?
A method is a function used as an object's property, typically invoked through that object. Method syntax also enables `super` in object literals and classes.

### 144. Why does extracting a method sometimes break it?
Calling `const f = obj.method; f()` loses the property-reference call form that supplied `obj` as `this`. Bind the method or call it through the object.

### 145. What is a variadic function?
A function that accepts a variable number of arguments. Use rest parameters to gather them and validate supported argument counts explicitly.

### 146. How does an async function handle a synchronous throw?
The throw becomes a rejection of the promise returned by the async function. Callers should await it within `try...catch` or attach a rejection handler.

### 147. What is tail-call optimization in ECMAScript?
The specification defines proper tail calls in certain strict-mode contexts, but implementation support is limited and varies. Do not rely on it for recursion depth.

### 148. What is a decorator conceptually?
A decorator transforms or annotates a class or class element at definition time. Exact syntax and semantics depend on standardized decorator support in the target runtime.

### 149. How should a callback API preserve `this`?
Specify the invocation receiver or pass required state explicitly. Consumers can bind the callback or wrap it in an arrow function.

### 150. How can a function be made idempotent?
Design repeated calls with the same input/state to produce the same effect, or guard completed work with stored state. Idempotence is especially useful for retries.

## 11. Objects, properties, and reflection

### 151. What is a property descriptor?
A descriptor defines a property's value or accessor behavior and attributes such as writable, enumerable, and configurable. Inspect it with `Object.getOwnPropertyDescriptor`.

### 152. What is a data property?
A property storing a value with `writable`, `enumerable`, and `configurable` attributes. Ordinary assignment typically creates a writable, enumerable, configurable own data property.

### 153. What is an accessor property?
A property defined by getter and/or setter functions rather than a stored value. Access may execute code, so property reads are not always side-effect-free.

### 154. What does `Object.defineProperty` do?
It creates or reconfigures an own property using a descriptor. Unspecified descriptor flags default to false for a new property, unlike ordinary assignment.

### 155. What is property enumerability?
It controls visibility to selected enumeration operations such as `Object.keys` and `for...in`. `Reflect.ownKeys` includes own enumerable and non-enumerable string and symbol keys.

### 156. What is the difference between `Object.keys` and `Reflect.ownKeys`?
`Object.keys` returns enumerable own string keys. `Reflect.ownKeys` returns all own string and symbol keys, including non-enumerable ones.

### 157. What does `Object.freeze` guarantee?
It prevents extensions and makes own data properties non-writable and properties non-configurable. It is shallow: nested objects may remain mutable.

### 158. What is the difference between `seal` and `freeze`?
Both prevent adding properties and configurable changes; `freeze` additionally prevents changing existing data-property values. Both are shallow operations.

### 159. How can an object be made non-extensible?
`Object.preventExtensions(obj)` prevents new own properties but permits changes to existing properties where descriptors allow. It does not freeze nested values.

### 160. What is a null-prototype object?
An object created with `Object.create(null)` has no inherited object methods. It can suit dictionary use, but `Map` is often a clearer choice.

### 161. How does `Object.assign` behave?
It copies enumerable own string and symbol properties from sources to a target using get and set operations. It is shallow and can invoke getters or setters.

### 162. How does object spread differ from `Object.assign`?
Both copy enumerable own properties shallowly, but object spread defines properties on a new object instead of assigning through target setters. Neither preserves source descriptors or prototype.

### 163. What does `Reflect` provide?
It exposes methods corresponding to many internal object operations, with consistent return values and argument handling. It is often used by proxies and metaprogramming code.

### 164. What is a proxy?
A `Proxy` intercepts selected operations on a target through traps. Traps must obey invariants, and proxies can affect performance, identity, and debugging.

### 165. What is a proxy invariant?
It is a constraint a proxy trap must respect relative to target descriptors and extensibility. Violations cause a `TypeError`.

### 166. What is a revocable proxy?
`Proxy.revocable` returns a proxy and a `revoke` function. After revocation, operations on the proxy throw, allowing access to be deliberately disabled.

### 167. How can you check whether an object owns a property?
Use `Object.hasOwn(object, key)`. It works with null-prototype objects and avoids shadowing pitfalls of calling `hasOwnProperty` directly.

### 168. What order do object keys use?
Own keys are ordered with array-index string keys first in ascending numeric order, then other string keys in insertion order, then symbols in insertion order.

### 169. What is a symbol property useful for?
It provides a unique property key that avoids accidental string-key collisions. Symbols are not secret and can still be discovered through reflection.

### 170. Are symbols included in JSON?
Symbol-keyed properties are ignored by `JSON.stringify`, and symbol values are not serialized as ordinary JSON values. Define an explicit serialization policy when symbols carry meaning.

### 171. What does `Symbol.for` do?
It looks up or creates a symbol in a shared global symbol registry for the realm. `Symbol.keyFor` retrieves a registered key; ordinary symbols are not registered.

### 172. What is `Symbol.iterator`?
It is the well-known symbol whose method makes an object iterable. `for...of`, spread, and many built-ins use the iterable protocol.

### 173. What is `Symbol.toPrimitive`?
It lets an object customize conversion to a primitive, receiving a hint such as `"number"`, `"string"`, or `"default"`. The result must be primitive.

### 174. How does property lookup interact with prototypes?
JavaScript first checks own properties, then follows prototypes until finding a match or reaching `null`. An own property can shadow an inherited one.

### 175. Why should untrusted keys be handled carefully?
Keys such as `__proto__` and inherited names can interact unexpectedly with prototype-bearing dictionaries. Prefer `Map`, null-prototype dictionaries, and explicit own-property validation.

## 12. Classes and inheritance

### 176. What does class syntax provide?
It defines constructor behavior, prototype methods, static members, inheritance, and private fields with class-specific rules. Instances still use ordinary prototype chains.

### 177. Are class declarations hoisted like function declarations?
The class binding is subject to a temporal dead zone and cannot be used before its declaration is evaluated. Its methods are installed when the class is evaluated.

### 178. What is a class constructor's rule about `this`?
In a base class, `this` is initialized when construction begins; in a derived class, `super()` must run before using `this`. Constructors cannot be called without `new`.

### 179. What does `extends` do?
It establishes prototype relationships for both instance methods and static inheritance. The derived constructor must call `super()` before accessing `this`.

### 180. What does `super` mean in a method?
It accesses the superclass method through the method's home object, while the call receiver remains the current `this`. `super` is not a standalone object reference.

### 181. What is a static class method?
A method stored on the class constructor rather than its instance prototype. It is called on the class and is inherited through constructor inheritance.

### 182. What are private class fields?
Names prefixed with `#` declare lexically private fields or methods, accessible only within the declaring class body. They are not ordinary string or symbol properties.

### 183. Are private fields inherited as accessible names?
A subclass does not gain source-level access to a superclass's private names. The declaring class's methods can still access their private fields on valid instances.

### 184. What is a static initialization block?
It is a `static {}` block in a class that runs once during class evaluation and can initialize static state with class-local private access.

### 185. What are public class fields?
They declare own properties on each instance (or static properties on the class) during initialization. Field initializers run in a defined order relative to superclass construction.

### 186. What is a mixin?
A mixin is a composition pattern that adds behavior from reusable functions or objects rather than relying solely on a deep class hierarchy. Define conflict and initialization rules explicitly.

### 187. Why favor composition over deep inheritance?
Composition makes dependencies and behavior combinations more explicit and avoids fragile superclass coupling. Inheritance remains useful for genuine substitutable type relationships.

### 188. What is polymorphism?
It is the ability to use different objects through a shared operation or interface, with each providing suitable behavior. JavaScript commonly uses structural conventions rather than nominal interfaces.

### 189. Can class methods be enumerated with `Object.keys(instance)`?
Usually not: methods are non-enumerable properties on the prototype, while instance fields are own enumerable properties by default. Inspect the prototype to find methods.

### 190. How does `super()` affect construction?
It invokes the superclass constructor and initializes the derived instance's `this`. In a derived constructor it must happen before any access to `this`.

### 191. What is `new.target`?
It identifies the constructor invoked by `new`, or is `undefined` when not constructed. It can distinguish direct construction from subclass construction.

### 192. Can a class constructor return a different object?
Yes, explicitly returning an object from a base constructor replaces the constructed result; primitive returns are ignored. Derived constructors have additional restrictions before `super()`.

### 193. What does `instanceof` mean for class instances?
It ordinarily checks whether the class's current `prototype` appears in the object's chain. Changing the prototype or crossing realms can change the result.

### 194. How can a class customize `instanceof`?
Define a static method keyed by `Symbol.hasInstance` that returns a boolean. Keep custom checks predictable and distinguish identity from structural validation.

### 195. What is a class's `prototype` property?
It is the object used as the prototype for instances created by that class. Static methods live on the constructor itself, not this instance prototype.

### 196. What happens if a subclass overrides a method?
Method lookup finds the subclass's own prototype method first. The override may call the superclass implementation with `super.method()`.

### 197. What is a private brand check?
The `#field in object` syntax tests whether an object carries the declaring class's private brand. It returns false for unbranded objects and throws if the right operand is not an object.

### 198. Are class methods automatically bound?
No. Extracting an instance method loses its receiver just like extracting an ordinary method. Bind it or use an arrow-function field for an instance-bound callback.

### 199. What is the trade-off of arrow-function class fields?
They capture the instance as `this` and are convenient callbacks, but each instance gets its own function property rather than sharing a prototype method.

### 200. How should inheritance constructors initialize state?
Call `super()` first in a derived constructor, then establish subclass invariants. Keep constructor work predictable and avoid virtual method calls before subclass initialization completes.

## 13. Arrays, iterators, and generators

### 201. What is an iterable?
An object is iterable when it implements a method at `Symbol.iterator` that returns an iterator. Arrays, strings, maps, and sets are built-in iterables.

### 202. What is an iterator?
An iterator has a `next()` method returning `{ value, done }` results. Once `done` is true, standard iterator consumers treat the sequence as complete.

### 203. What is the iterator protocol's `return()` method for?
It optionally signals early termination, allowing iterators to release resources. Constructs such as `for...of` call it when they exit early if it is present.

### 204. What is an iterator result?
It is an object with a `value` field and a boolean `done` field. When done is true, `value` may contain a generator's final return value.

### 205. How does `for...of` differ from `for...in`?
`for...of` consumes iterable values; `for...in` enumerates enumerable string property names, including inherited ones. Use `Object.keys` for own enumerable string keys.

### 206. How does spreading an iterable work?
Spread obtains values through the iterable protocol and expands them into an array, call arguments, or object literal as supported. It consumes the iterable and may invoke user code.

### 207. How does array destructuring consume an iterable?
It calls the iterator and reads successive values; a rest element consumes the remainder. Destructuring may close the iterator early if the pattern finishes before the sequence does.

### 208. What is an array hole?
A hole is an absent indexed property, not an explicit `undefined`. Methods such as `map` skip holes, while `Array.from` produces explicit `undefined` entries for them.

### 209. What does `Array.from` do?
It creates an array from an iterable or array-like object and can map each value during creation. Its mapping callback receives value and index.

### 210. How does `Array.of` differ from the `Array` constructor?
`Array.of(3)` creates `[3]`; `Array(3)` creates a length-three sparse array. `Array.of` avoids the constructor's single-number special case.

### 211. What is the difference between `find` and `filter`?
`find` returns the first matching element or `undefined`; `filter` returns a new array containing every match. Use `findIndex` when the first match's index is needed.

### 212. What is the difference between `some` and `every`?
`some` tests whether any visited element passes; `every` tests whether all visited elements pass. Both short-circuit and return false/true respectively for empty arrays.

### 213. What does `flatMap` do?
It maps each element and flattens one array level from the results. It can combine mapping and expansion without recursively flattening arbitrary nested arrays.

### 214. Does `sort` mutate an array?
Yes, `sort` sorts in place and by default compares string representations. Supply a numeric comparator for numbers or use `toSorted()` for a copied result.

### 215. What is a well-formed numeric sort comparator?
It returns a negative number, zero, or positive number to place `a` before, equal to, or after `b`. Avoid subtraction when values may be non-numeric or special.

### 216. What is a stable sort?
A stable sort preserves the relative order of elements that compare equal. ECMAScript requires stable `Array.prototype.sort` behavior in modern conforming implementations.

### 217. What is `Array.prototype.with`?
It returns a copy with one indexed element replaced, supporting negative indexes from the end. It does not mutate the source array.

### 218. What does `toReversed` do?
It returns a new array with elements in reverse order, leaving the source unchanged. It is the copying counterpart to mutating `reverse`.

### 219. How do `toSorted` and `sort` differ?
`toSorted` returns a sorted copy; `sort` mutates the original. Both use the same comparator behavior.

### 220. What does `toSpliced` do?
It returns a copied array with elements removed, replaced, or inserted at an index. It is the non-mutating counterpart to `splice`.

### 221. How do you make an object iterable?
Implement `[Symbol.iterator]()` to return an iterator whose `next()` produces the sequence. Prefer generators when they make the iteration logic clearer.

### 222. What are generator `next` arguments used for?
An argument to a later `next(value)` becomes the result of the suspended `yield` expression. The first `next` argument is generally ignored because execution has not reached a yield.

### 223. What do generator `throw` and `return` do?
They resume a generator by throwing at its suspension point or requesting completion. `finally` blocks still run during generator closing.

### 224. What is an async iterable?
It has a method at `Symbol.asyncIterator` returning an async iterator. `for await...of` consumes its promise-valued iteration steps.

### 225. Can a synchronous iterable be used with `for await...of`?
Yes. The construct can adapt synchronous iterators, awaiting yielded promise values; asynchronous iterables are preferred when each `next()` itself is asynchronous.

## 14. Collections and data structures

### 226. How does `Map` compare keys?
It uses SameValueZero equality for keys and preserves insertion order. Object keys compare by identity rather than by structure.

### 227. What is a `Map` advantage over a plain object?
It supports keys of any value, has explicit key/value operations, and avoids accidental inherited keys. Its `size` property gives the number of entries.

### 228. What does `Map.set` return?
It returns the map itself, enabling chained `set` calls. Setting an existing key updates its value without moving its insertion order.

### 229. What happens when a `Map` key is set twice?
The later value replaces the earlier value while the key retains its original insertion position. Different object instances remain different keys.

### 230. How does `WeakMap` differ from `Map`?
WeakMap keys must be objects (including non-registered symbols where supported) and do not keep key objects alive solely by being keys. It is not enumerable and has no `size`.

### 231. When is `WeakMap` useful?
It associates metadata with object lifetimes without creating a strong reference from the collection. It is useful for private per-object data and caches.

### 232. Why can a `WeakMap` not be enumerated?
Enumeration could expose garbage-collection timing and undermine weak-reference semantics. Its API deliberately omits iteration, `size`, and key listing.

### 233. How does `Set` determine uniqueness?
It uses SameValueZero semantics, preserving the first insertion position of each unique value. Objects are unique by identity.

### 234. How can set union be computed?
Create a new `Set` from both iterable inputs, for example `new Set([...a, ...b])`. Newer set-composition methods may be available depending on runtime support.

### 235. What is a `WeakSet` for?
It tracks object membership without keeping members alive solely through the set. It cannot be iterated or queried for size.

### 236. When is a plain object a good dictionary?
When keys are known strings and ordinary object behavior is desired, with safe own-key handling. For arbitrary keys or frequent additions/removals, use `Map`.

### 237. What does `Map` iteration order guarantee?
Entries iterate in insertion order, with updates retaining position and deletion followed by reinsertion adding the key at the end.

### 238. What is a typed array?
It is an array-like view over binary data with elements of a fixed numeric representation. Typed arrays are useful for buffers, binary formats, graphics, and numeric workloads.

### 239. How does a typed array differ from a normal array?
Its elements have fixed numeric types and are stored in an associated buffer; length is fixed. Normal arrays can hold mixed values and grow or shrink.

### 240. What is an `ArrayBuffer`?
It represents a fixed-length raw byte region. Typed arrays and `DataView` provide ways to interpret or access its contents.

### 241. What is a `DataView`?
It provides getters and setters for multiple integer and floating-point representations over an `ArrayBuffer`, with explicit byte offsets and endianness controls.

### 242. What is a detached buffer?
A detached `ArrayBuffer` no longer exposes its original bytes, typically after transfer. Views over it become unusable or have zero length according to their kind.

### 243. What does transferring an `ArrayBuffer` do?
It moves ownership to another context, commonly a worker, rather than copying bytes. The original buffer is detached, reducing transfer overhead.

### 244. What is a `SharedArrayBuffer`?
It is memory accessible by multiple agents, subject to platform isolation requirements in browsers. Coordinate shared access with `Atomics` to avoid data races.

### 245. What are `Atomics` used for?
They provide atomic operations and waiting/notification primitives for shared typed-array memory. They support synchronization, not general-purpose automatic race prevention.

### 246. What is `BigInt64Array`?
It views buffer data as signed 64-bit BigInt elements. Values wrap to the fixed-width representation; use `BigInt` arithmetic for unbounded integer values.

### 247. How does `DataView` handle endianness?
Multi-byte getters and setters accept an endianness argument; the default is big-endian when omitted. Choose the byte order required by the external format.

### 248. What is a `WeakRef`?
It holds a weak reference to an object that may be garbage-collected. Use it only for optional caches or similar features, never for correctness-critical state.

### 249. What is a `FinalizationRegistry`?
It schedules cleanup callbacks after registered targets become unreachable, but timing and execution are nondeterministic. It must not replace explicit resource cleanup.

### 250. When should weak references be avoided?
Avoid them whenever behavior depends on whether or when garbage collection occurs. Prefer explicit disposal and strong ownership for correctness.

## 15. Modules and packaging

### 251. What is a named export?
It exports a binding under an explicit name, which consumers can import by that name or alias. A module can expose multiple named exports.

### 252. What is a default export?
It exports one default binding that importers may name freely. A module has at most one default export.

### 253. How can an export be renamed?
Use `export { localName as publicName }` or alias it during import. Aliases clarify public API names without renaming the local binding.

### 254. What is a re-export?
It forwards another module's export through the current module, useful for a package entry point. Re-exporting can create ambiguous names in complex graphs.

### 255. What does `export *` do?
It re-exports named exports except the default export. Conflicting names from multiple star exports become ambiguous and are not available under that name.

### 256. What is dynamic `import()`?
It loads a module at runtime and returns a promise for its module namespace object. It enables conditional loading and code splitting.

### 257. Does dynamic import return the default export directly?
No. It resolves to a module namespace object; the default export is available as its `default` property.

### 258. What is top-level `await`?
It allows `await` in an ECMAScript module's top-level body. Dependent modules wait for the awaited module's evaluation to complete.

### 259. What is a module graph?
It is the set of modules and dependency edges needed from an entry point. Linking and evaluation proceed according to dependency relationships.

### 260. What does module linking do?
It resolves imports to exports and prepares module environments before evaluation. Resolution details can depend on the host or build tool.

### 261. What is tree shaking?
It is build-time elimination of code believed to be unused, commonly aided by static ES module imports and exports. Side effects and dynamic behavior can limit safe removal.

### 262. What does a package entry point specify?
It identifies exposed module paths through metadata such as `exports` in Node packages. Restricting entry points can create a clearer, more stable public API.

### 263. What is conditional package exports?
It maps package subpaths to different files based on conditions such as `import`, `require`, or environment. Incorrect mappings can cause dual-package or interop problems.

### 264. What is the difference between ESM and CommonJS?
ES modules use static imports/exports and live bindings; CommonJS uses runtime `require` and `module.exports`. Their loading, resolution, and interop rules differ.

### 265. What is CommonJS `require` caching?
Node caches loaded CommonJS modules by resolved filename, returning the cached exports on later `require` calls. Mutating shared exports can therefore affect other consumers.

### 266. How does an ES module execute only once?
Within a module graph, a successfully loaded module is instantiated and evaluated once per module identity; importers share its bindings and state.

### 267. What is import map configuration?
In supported browsers, an import map maps module specifiers to URLs before module resolution. It does not replace server-side package resolution.

### 268. Why should module side effects be minimized?
Importing a module evaluates its top-level code. Hidden initialization can affect startup order, tests, tree shaking, and reuse.

### 269. What is a side-effect import?
`import "./setup.js"` loads and evaluates a module without importing bindings, often for registration or initialization. Keep such effects explicit and controlled.

### 270. What is module resolution?
It is the host-specific process that maps a module specifier to a module resource. Browser URLs, Node package rules, and bundler aliases are not identical.

### 271. What is an import assertion or attribute used for?
It lets import syntax include host-interpreted metadata, for example identifying JSON modules where supported. Exact syntax and support depend on current standards and host implementation.

### 272. Why can two package copies create identity bugs?
Each resolved module copy can create distinct constructors, symbols, or singleton state. Objects from one copy may fail identity checks against another.

### 273. What is a dual package hazard?
A package exposed in both ESM and CommonJS forms may be loaded twice through different entry points, splitting singleton state or constructor identity.

### 274. What is code splitting?
It divides a bundle into separately loadable chunks, often using dynamic imports. It can reduce initial transfer but adds loading and failure paths.

### 275. How should a public module API be designed?
Export a small, documented surface with stable names and predictable side effects. Keep implementation details private so internals can change safely.

## 16. Promises and asynchronous patterns

### 276. What are the states of a promise?
A promise is pending, fulfilled, or rejected; fulfilled and rejected promises are settled. A settled promise never changes state again.

### 277. What is promise resolution?
Resolving a promise adopts another promise or thenable's eventual state. Resolution is not always immediate fulfillment with the supplied object.

### 278. What is a thenable?
An object with a callable `then` property. Promise resolution assimilates thenables, which means accessing or invoking their `then` can run user code.

### 279. What does `.then()` return?
It returns a new promise resolved from the handler's return value. A thrown error or rejected returned promise rejects the new promise.

### 280. What does `.catch()` return?
It is a shorthand for `.then(undefined, onRejected)` and returns a promise. If its handler returns normally, it can recover the chain.

### 281. What does `.finally()` do?
It runs a cleanup handler after either fulfillment or rejection, usually preserving the original outcome. A thrown error or rejected promise from cleanup replaces that outcome.

### 282. Does `Promise.all` cancel its other inputs on rejection?
No. It rejects as soon as an input rejects, but other operations continue unless they have a separate cancellation mechanism.

### 283. What does `Promise.all` return for an empty iterable?
It returns a promise fulfilled with an empty array. Results otherwise preserve input order, not completion order.

### 284. What does `Promise.allSettled` return?
It fulfills after every input settles, with an ordered array describing each fulfillment or rejection. It does not fail fast on input rejection.

### 285. What does `Promise.race` do?
It settles with the first input promise to settle, whether fulfilled or rejected. It does not cancel the remaining operations.

### 286. What does `Promise.any` do?
It fulfills with the first fulfilled input. If all inputs reject, it rejects with an `AggregateError`; an empty iterable also rejects.

### 287. How do you handle an unhandled rejection?
Attach a rejection handler to the relevant promise chain or await it within `try...catch`. Runtime unhandled-rejection reporting is a diagnostic, not a substitute for handling.

### 288. Why can an error be lost in a promise chain?
A rejection handler may consume it by returning normally, or a detached promise created in a callback may reject independently. Return nested promises so failures propagate.

### 289. What does `await` accept?
It accepts any value; non-promises are treated as already fulfilled. For thenables, it uses promise-resolution behavior.

### 290. Does `await` block the JavaScript thread?
No. It suspends the current async function and resumes it later, allowing the thread to perform other work. Synchronous CPU work still blocks.

### 291. How do you run asynchronous tasks in sequence?
Await each operation in order, typically within a loop. This is appropriate when later work depends on earlier results or concurrency must be limited.

### 292. How do you run independent asynchronous tasks concurrently?
Start them before awaiting, then aggregate with `Promise.all` or a related method. Bound concurrency when launching all work at once would overload resources.

### 293. Why is `forEach(async () => ...)` often a bug?
`forEach` ignores callback return promises, so the caller cannot await completion or catch failures through that loop. Use `for...of` or `Promise.all(array.map(...))`.

### 294. What is an `AbortController`?
It creates an `AbortSignal` that can notify cooperating operations to stop. Cancellation support is API-specific and may not undo work already completed.

### 295. How should cancellation be propagated?
Pass an `AbortSignal` through each supporting layer, check it at appropriate boundaries, and release resources in `finally`. Define whether cancellation rejects or returns a special result.

### 296. What is a timeout wrapper for a promise?
It races work against a timer and rejects or resolves when the timer wins. A race alone does not stop the underlying operation; use cancellation when supported.

### 297. What is promise chaining?
It sequences asynchronous transformations by returning each promise from `.then()`. Returning nested promises flattens them into the chain.

### 298. What is a promise executor?
The function passed to `new Promise`, invoked synchronously with resolve and reject functions. Use constructors mainly to adapt callback APIs, not around existing promises.

### 299. What if a promise executor throws?
The promise constructor catches the synchronous throw and rejects the promise. Exceptions thrown later in unrelated asynchronous callbacks are not automatically captured by that executor.

### 300. What is the difference between `resolve` and `reject`?
Resolve adopts the state of a supplied promise or thenable; reject settles immediately as rejected with a reason. Use the operation matching the intended outcome.

## 17. Event loop and concurrency

### 301. What is the JavaScript event loop?
It coordinates execution of queued tasks, microtasks, and host events around the call stack. Exact scheduling details vary by host and event-loop phase.

### 302. What is a task (macrotask)?
It is a unit of host-scheduled work, such as a timer callback, input event, or message event. “Macrotask” is common informal terminology; host standards define specific task sources.

### 303. What is a microtask?
It is queued work run at microtask checkpoints, typically after the current task's synchronous execution and before the next task. Promise reactions and `queueMicrotask` use microtasks.

### 304. What runs first: a resolved promise callback or a zero-delay timer?
After the current synchronous stack finishes, promise reactions normally run as microtasks before the timer's later task. A zero delay does not mean immediate execution.

### 305. When does the browser usually render?
Browsers generally perform rendering opportunities between tasks, with microtask checkpoints and other scheduling rules affecting timing. Long synchronous work can delay rendering.

### 306. Can microtasks starve rendering?
Yes. Continuously enqueueing microtasks can prevent the event loop from reaching rendering or later tasks. Break up sustained work with appropriate task scheduling.

### 307. What does `queueMicrotask` do?
It schedules a callback in the host's microtask queue. Exceptions are reported as uncaught errors rather than becoming a promise rejection automatically.

### 308. How is a promise callback different from a timer callback?
Promise reactions are microtasks, while timers schedule host tasks after a delay threshold. Both run later than the current synchronous stack, but have different ordering.

### 309. What does `setTimeout(fn, 0)` guarantee?
It requests a timer callback after a minimum delay, subject to clamping, throttling, and event-loop scheduling. It does not guarantee a precise execution time.

### 310. What is timer clamping?
Hosts may increase very small timer delays, especially for nested or background-page timers, to manage resources. Applications should not depend on timers as precision clocks.

### 311. What does `requestAnimationFrame` schedule?
It requests a callback before a future browser repaint, making it useful for visual updates. It may be paused or throttled in background tabs.

### 312. What is `requestIdleCallback` for?
Where supported, it schedules low-priority work during idle periods with a deadline hint. It is not universally available and should not be required for urgent work.

### 313. What is a Web Worker?
A worker runs JavaScript in a separate agent with its own event loop, allowing CPU work without blocking the page's main thread. Communication commonly uses messages.

### 314. Do workers share ordinary objects with the page?
No. Structured cloning typically copies supported data; transferable objects can move ownership, and `SharedArrayBuffer` is a special shared-memory case.

### 315. What is structured cloning?
It is an algorithm for copying supported data graphs between realms or APIs. It preserves cycles and many built-in types but not arbitrary functions or prototype behavior.

### 316. What is a transferable object?
It is an object whose underlying resource can be transferred between contexts, commonly detaching it from the sender. Transfer can avoid copying large buffers.

### 317. What is `postMessage` used for?
It sends messages between windows, workers, or other host contexts. Validate sender origin and message shape, especially when receiving cross-origin data.

### 318. What is concurrency versus parallelism?
Concurrency is managing multiple tasks whose progress overlaps; parallelism is executing work simultaneously. A single event loop can be concurrent without parallel JavaScript execution.

### 319. Does `Promise.all` create parallel CPU execution?
No. It coordinates promises; CPU-bound JavaScript on one agent still runs sequentially unless moved to workers or another parallel facility.

### 320. What is a race condition in asynchronous JavaScript?
It occurs when outcome depends on timing or completion order of overlapping operations. Use ordering, cancellation, locks, version checks, or idempotent updates as appropriate.

### 321. How can stale async results be prevented?
Associate each request with a sequence/version and ignore results older than the latest request, or abort superseded work where supported.

### 322. What is backpressure?
It is a mechanism that slows producers when consumers cannot keep up. Streams and bounded queues can provide backpressure; unbounded promise creation often cannot.

### 323. What is a worker pool?
A fixed set of workers that processes queued jobs, limiting parallelism and resource use. It can improve CPU throughput while avoiding one-worker-per-task overhead.

### 324. What is cooperative scheduling?
Tasks voluntarily yield control at safe points so other work can proceed. Chunking long computations can improve responsiveness without changing the result.

### 325. How can long synchronous work be split?
Process bounded chunks and schedule later chunks using host-appropriate task or rendering APIs. Measure latency and avoid flooding the queue with tiny callbacks.

## 18. Browser APIs and platform behavior

### 326. What is the DOM?
The Document Object Model is a host API representing documents as node trees. It is not part of ECMAScript itself.

### 327. What does `DOMContentLoaded` indicate?
It fires after the document has been parsed and deferred/module scripts have run, without waiting for all images and other subresources. It differs from the full `load` event.

### 328. What is event bubbling?
After target-phase handling, an event may propagate through ancestors toward the root. Some events do not bubble; consult the event's behavior.

### 329. What is event capturing?
It is the propagation phase from the root toward the target. A listener can request capture with the `capture` option.

### 330. What does `stopPropagation` do?
It stops further propagation to other nodes but does not stop other listeners on the current node. `stopImmediatePropagation` also stops later listeners on that node.

### 331. What does `preventDefault` do?
It cancels a cancelable event's default browser action when allowed. It does not stop propagation, and passive listeners cannot prevent default.

### 332. What is event delegation?
It handles events on a common ancestor and identifies the originating descendant, often using bubbling. It reduces listener count for dynamic child lists.

### 333. What is the `once` listener option?
It removes the event listener automatically after the first invocation. Other listener options include `capture`, `passive`, and `signal`.

### 334. What is a passive event listener?
It promises the listener will not call `preventDefault`, allowing the browser to optimize some input handling. A passive listener cannot cancel the event.

### 335. What is `localStorage`?
It is synchronous, origin-scoped string storage that generally persists across browser sessions. Avoid large or sensitive data, and handle quota and access errors.

### 336. How does `sessionStorage` differ?
It is string storage scoped to an origin and page session, commonly isolated by top-level browsing context. It is still synchronous and subject to browser policies.

### 337. What is IndexedDB?
It is an asynchronous transactional database API in browsers for structured and indexed data. It supports larger datasets and richer queries than Web Storage.

### 338. What is the Cache API?
It stores request/response pairs, often for service workers and offline behavior. It is distinct from the browser's HTTP cache and requires deliberate invalidation.

### 339. What is a service worker?
It is an event-driven worker that can mediate network requests and support offline features. It has lifecycle, scope, and secure-context requirements.

### 340. What is the Fetch API?
It provides promise-based requests and responses in browser and some server runtimes. HTTP error statuses generally fulfill the fetch promise; inspect `response.ok`.

### 341. Does `fetch` reject on HTTP 404?
Usually no; the promise fulfills with a `Response` for HTTP error statuses. Check `ok` or `status` and handle network failures separately.

### 342. How do you send JSON with `fetch`?
Set an appropriate content type and pass `JSON.stringify(data)` as the body for methods that support one. Verify server expectations and response status.

### 343. What is CORS preflight?
For certain cross-origin requests, the browser sends an `OPTIONS` request to ask permission before the actual request. The server must return suitable CORS headers.

### 344. Does CORS prevent all cross-origin requests?
No. It primarily controls whether browser scripts can read certain cross-origin responses; other requests may still be sent. It is not a CSRF defense by itself.

### 345. What does `URLSearchParams` do?
It parses and constructs URL query parameters with encoding rules. Repeated keys are supported; use `getAll` when multiple values are meaningful.

### 346. What is the History API?
It allows same-origin session-history entries to be added or replaced and reports navigation changes. It does not itself load arbitrary remote pages.

### 347. What is the Page Visibility API?
It reports whether a document is visible, enabling apps to reduce unnecessary background work. Visibility does not guarantee a page is actively viewed.

### 348. What is `IntersectionObserver`?
It asynchronously reports when observed elements intersect a viewport or root. It is useful for lazy loading and visibility-triggered behavior without continuous scroll handlers.

### 349. What is `ResizeObserver`?
It reports changes to an element's content or border box size. Avoid feedback loops by preventing observer callbacks from repeatedly causing the observed resize.

### 350. What is the `Intl` API?
It provides locale-sensitive formatting and comparison, including dates, numbers, and collation. Use it rather than hand-written locale rules.

## 19. Node.js and runtime differences

### 351. What is Node.js?
It is a JavaScript runtime built around a JavaScript engine and host APIs, including filesystem and networking facilities. Those APIs are not part of browser ECMAScript.

### 352. What is the Node.js event loop?
It is the runtime's event-processing model, implemented with libuv and Node-specific phases. Browser scheduling details should not be assumed to match it exactly.

### 353. What is `process.nextTick`?
It queues a callback in Node's next-tick queue, which is drained before the event loop proceeds to later phases. Excessive recursive use can starve I/O.

### 354. How does `setImmediate` differ from `setTimeout` in Node?
`setImmediate` runs in the check phase after poll processing; timers run according to timer scheduling. Relative ordering can depend on where they are scheduled.

### 355. What is a Node.js stream?
It is an abstraction for handling data incrementally, including readable, writable, duplex, and transform streams. Streams can avoid buffering an entire dataset.

### 356. What is stream backpressure in Node?
It signals that a writable destination cannot accept data as quickly as a source produces it. Respect `write()` return values or use `pipeline` to manage flow and errors.

### 357. What is `Buffer`?
It is a Node.js byte sequence API, implemented as a `Uint8Array` subclass. Specify encodings explicitly when converting between text and bytes.

### 358. Why should `Buffer.allocUnsafe` be used carefully?
It may expose uninitialized memory contents until overwritten. Use `Buffer.alloc` for zero-filled memory or fully initialize every byte before reading.

### 359. What is the Node.js `EventEmitter`?
It is a host class for registering and emitting named events. An unhandled `'error'` event is special and can throw, so attach appropriate handlers.

### 360. How does Node.js resolve CommonJS modules?
It resolves built-ins, files, and packages using Node-specific search rules and package metadata. Do not assume browser URL-based ESM resolution follows those rules.

### 361. What is the `exports` field in `package.json`?
It defines public package entry points and can supply condition-specific targets. It can block access to unlisted internal paths.

### 362. How can Node execute an ES module?
Node recognizes ESM through extensions, package configuration, or input mode, depending on version and context. Follow the runtime's current module rules.

### 363. What is `node:` in a built-in module specifier?
It explicitly identifies a Node.js built-in such as `node:fs`, avoiding ambiguity with similarly named packages.

### 364. What is a worker thread in Node.js?
It runs JavaScript on a separate thread with messaging and optional shared memory. It is useful for CPU-intensive work, not generally for ordinary asynchronous I/O.

### 365. What is a child process?
It runs a separate operating-system process with its own memory and communicates through configured IPC or streams. Validate commands and arguments to avoid injection.

### 366. What is the difference between `spawn` and `exec`?
`spawn` streams process input/output, while `exec` typically buffers output and invokes a shell. Prefer argument arrays and avoid shell interpretation of untrusted input.

### 367. What is the Node.js global `fetch`?
It is a Fetch-compatible API available in current Node releases, though details can differ from browsers. Check the deployed Node version and API support.

### 368. How should Node secrets be managed?
Load them from a managed secret store or protected environment configuration, restrict access, and never log or commit them. Rotate exposed credentials.

### 369. What is an unhandled exception in Node?
It is an exception that escapes the current synchronous call stack or async context without a handler. Handle errors at meaningful boundaries and shut down gracefully when state is unsafe.

### 370. How should a Node process handle shutdown?
Stop accepting work, finish or cancel in-flight operations within a deadline, close resources, then exit. Signal handling should avoid indefinite hangs.

### 371. What is the Node.js module cache?
It stores loaded modules according to resolved identity, enabling reuse and singleton-like state. Tests can isolate modules by controlling the loader or process.

### 372. What is `process.env`?
It exposes environment variables as string values (or absent values) to the Node process. Parse and validate values explicitly, especially booleans and numbers.

### 373. Why is browser `window` unavailable in Node?
`window` is a browser host global, not an ECMAScript built-in. Node exposes different globals; portable libraries should inject or abstract host dependencies.

### 374. What is the `node:test` module?
It is Node's built-in test runner for defining and executing tests. Assertions are commonly supplied by `node:assert`.

### 375. How can code share browser and Node behavior?
Keep ECMAScript-only logic separate from host-specific adapters, and use feature detection or dependency injection. Test each supported runtime explicitly.

## 20. Security and robustness

### 376. What is prototype pollution?
It is unintended modification of shared prototypes through unsafe property assignment or merging. It can alter behavior across objects and create security issues.

### 377. How can prototype pollution be reduced?
Validate keys, avoid unsafe recursive merges, use `Map` or null-prototype dictionaries, and keep dependencies patched. Reject dangerous keys where appropriate.

### 378. What is cross-site request forgery?
CSRF tricks a browser into sending an authenticated request to a site. Use appropriate SameSite cookies, anti-CSRF tokens, origin checks, and safe HTTP method design.

### 379. What is cross-site scripting?
XSS executes attacker-controlled script in a trusted page. Use context-appropriate output encoding, safe DOM APIs, sanitization for rich HTML, and defense-in-depth policies.

### 380. Why is `innerHTML` risky?
It parses a string as markup and may create executable or dangerous content. Use `textContent` for text, or a vetted sanitizer when HTML is genuinely needed.

### 381. What is Content Security Policy?
CSP is a response-header policy that restricts resource loading and execution sources. Nonces or hashes can support scripts while reducing reliance on unsafe inline execution.

### 382. What does Subresource Integrity do?
SRI lets a browser verify fetched resource bytes against a supplied cryptographic hash. It is useful for eligible external resources and does not replace secure hosting.

### 383. Why is `eval` a security concern?
Evaluating attacker-influenced strings permits code execution and often conflicts with CSP. Parse data formats such as JSON with their dedicated parsers instead.

### 384. Is `JSON.parse` safe for every application input?
It parses JSON syntax, not application validity. Validate the resulting structure, types, lengths, and allowed keys before using it.

### 385. What is input validation?
It checks that input conforms to expected types, ranges, and business rules at a trust boundary. Client-side checks improve usability but must be repeated on trusted servers.

### 386. Why is output encoding context-specific?
HTML text, attributes, URLs, CSS, and JavaScript have different parsing rules. A correct encoding for one context may be unsafe in another.

### 387. What is a trusted types policy?
Where supported, Trusted Types can constrain dangerous DOM injection sinks to accept vetted typed values. It complements sanitization and CSP rather than replacing them.

### 388. What is a timing attack?
It infers secrets from execution-time differences. Use vetted cryptographic APIs and constant-time comparison where available for secret-dependent checks.

### 389. Why should cryptography not be implemented from scratch?
Cryptographic primitives are easy to implement incorrectly and hard to validate. Use well-reviewed platform cryptography or established libraries with safe defaults.

### 390. What is Web Crypto?
It is a browser and some-runtime API for cryptographic operations and secure random generation. Use `crypto.getRandomValues` or suitable cryptographic APIs, not `Math.random`, for secrets.

### 391. What is clickjacking?
It tricks users into interacting with a concealed or framed page. Defenses include frame restrictions such as CSP `frame-ancestors` and appropriate user-interface safeguards.

### 392. What is secure cookie configuration?
Use `Secure`, `HttpOnly`, and an appropriate `SameSite` setting, with narrow domain/path scope and short lifetimes where practical. Cookies remain subject to CSRF considerations.

### 393. Why are secrets in frontend JavaScript not secret?
Code and bundled configuration delivered to clients can be inspected. Keep credentials on trusted servers and issue narrowly scoped, short-lived client tokens where needed.

### 394. What is dependency supply-chain risk?
Malicious or compromised packages can execute code or expose data during installation or runtime. Pin and review dependencies, limit privileges, and monitor advisories.

### 395. What is a regular-expression denial of service?
It is excessive computation from pathological input and regex patterns, often due to catastrophic backtracking. Bound input and choose linear-time patterns or engines where possible.

### 396. How can URL input create security problems?
Untrusted schemes, hosts, redirects, or path components can enable phishing, script URLs, or server-side request forgery. Parse URLs and allowlist expected protocols and destinations.

### 397. Why should postMessage origins be checked?
Messages can come from unexpected frames or windows. Verify `event.origin`, expected source where suitable, and the message schema before acting.

### 398. What is deserialization risk?
Reconstructing objects from untrusted data can trigger unsafe behavior or exploit parser flaws. Prefer constrained formats, schema validation, and safe object construction.

### 399. What is least privilege?
It means granting code and processes only the permissions needed for their tasks. It limits the damage from bugs, compromised dependencies, or injection.

### 400. How should errors be exposed to clients?
Return useful but non-sensitive messages and keep diagnostic details in protected logs. Do not expose stack traces, secrets, internal paths, or raw exception data.

## 21. Performance and memory

### 401. How should JavaScript performance be measured?
Profile representative workloads with runtime tools, repeat measurements, and compare against a baseline. Optimize observed bottlenecks rather than guessing from source appearance.

### 402. What is a performance bottleneck?
It is the resource or operation limiting overall throughput or responsiveness, such as CPU, network, rendering, memory, or storage. Improving unrelated code may have no measurable effect.

### 403. What is JIT compilation?
Many JavaScript engines compile frequently executed code to machine code and optimize based on observed behavior. Optimization strategies vary; write clear code and measure on target runtimes.

### 404. What is deoptimization?
An engine may discard optimized code when assumptions no longer hold and resume less optimized execution. Do not depend on undocumented optimization heuristics.

### 405. What is garbage collection?
It automatically reclaims memory that is no longer reachable through program references. Collection timing is nondeterministic and cannot be directly controlled reliably.

### 406. What is a memory leak in JavaScript?
It is unintended retention of objects that are no longer useful but remain reachable. Common causes include long-lived listeners, caches, timers, and closures.

### 407. How can event listeners cause leaks?
A long-lived target retaining a listener may keep its captured objects reachable. Remove listeners when their lifecycle ends or register with an abort signal.

### 408. What is a memory snapshot?
It is a profiler capture of allocated objects and references at a point in time. Comparing snapshots can reveal unexpectedly retained objects and retaining paths.

### 409. What is memoization?
It caches a function's result for a key derived from its inputs. It is appropriate for deterministic expensive work when cache invalidation, identity, and memory bounds are addressed.

### 410. What is cache invalidation?
It is deciding when cached values are stale and should be refreshed or removed. Define validity rules and bounds explicitly rather than allowing unbounded accumulation.

### 411. What is lazy evaluation in JavaScript code?
It defers computing a value until it is needed, often through a function, generator, or lazy-loading mechanism. It can save work but may complicate timing and errors.

### 412. What is code splitting's performance trade-off?
It can reduce initial bundle cost but adds chunk requests, parsing, and loading boundaries. Measure startup and interaction performance under realistic network conditions.

### 413. What is tree shaking's limitation?
Dynamic access, side effects, and imperfect module metadata can prevent safe removal. A smaller bundle is not automatically faster if it shifts work elsewhere.

### 414. What is layout thrashing?
It occurs when code alternates layout-affecting writes and synchronous layout reads, forcing repeated recalculations. Batch reads before writes or use rendering phases.

### 415. How can rendering work be optimized?
Avoid unnecessary DOM updates, batch mutations, and keep expensive calculations off critical interaction paths. Profile actual rendering and layout costs.

### 416. What is a long task in a browser?
It is a lengthy uninterrupted task on the main thread that can delay input and rendering. Break up CPU work or move it to a worker.

### 417. What is debouncing's performance use?
It coalesces rapid events and runs after activity settles, reducing repeated expensive work such as search requests. Include cancellation and lifecycle cleanup.

### 418. What is throttling's performance use?
It caps how frequently work runs during a stream of events. Choose leading/trailing behavior to match user experience needs.

### 419. What is the cost of excessive object allocation?
It increases allocation and garbage-collection work, but modern engines optimize many short-lived objects. Profile before sacrificing clarity for reuse.

### 420. What is a hot path?
It is frequently executed code that materially affects performance. Optimize hot paths only after profiling representative usage.

### 421. How does string concatenation affect performance?
Engines may optimize many concatenations, so theoretical concerns are not enough to choose a strategy. Measure large workloads and avoid unnecessary intermediate data.

### 422. What is the cost of deep cloning?
It traverses and allocates a large object graph, potentially losing custom semantics or failing on unsupported data. Prefer targeted immutable updates or structured cloning when appropriate.

### 423. What is a performance budget?
It defines acceptable limits for metrics such as bundle size, interaction latency, or memory. Budgets turn performance goals into measurable regression checks.

### 424. Why should benchmarks warm up code?
JIT compilation, caches, and runtime startup can skew early measurements. Use repeated runs, representative data, and statistical summaries.

### 425. How do you improve startup performance?
Reduce critical-path code and resources, defer nonessential work, and avoid expensive synchronous initialization. Validate with real-user or production-like measurements.

## 22. Testing and developer tooling

### 426. What is a unit test?
It tests a small unit of behavior in isolation with controlled dependencies. Good unit tests are deterministic and focus on observable outcomes.

### 427. What is an integration test?
It verifies that multiple components work together across a meaningful boundary, such as a service with a database adapter. Keep external dependencies controlled.

### 428. What is an end-to-end test?
It exercises a user-visible workflow through the application stack, often in a real or simulated browser. Such tests catch integration issues but are slower and more brittle.

### 429. What is a test double?
It is a substitute used in tests, including stubs, fakes, spies, or mocks. Use the simplest double that verifies the behavior of interest.

### 430. What is a deterministic test?
It produces the same result for the same setup without relying on uncontrolled time, randomness, network, or shared state. Determinism improves diagnosis and CI reliability.

### 431. How should time-dependent code be tested?
Inject a clock or use the test framework's controlled timers. Avoid depending on wall-clock sleeps, which are slow and flaky.

### 432. What is property-based testing?
It generates many inputs from defined properties and checks invariants rather than only enumerated examples. It is effective for parsers, transforms, and edge-heavy logic.

### 433. What is a test assertion?
It states an expected condition and fails the test when that condition is false. Prefer assertions that describe externally meaningful behavior.

### 434. What is a flaky test?
A test that passes or fails nondeterministically without relevant code changes. Control timing and shared resources, and investigate rather than retrying away real defects.

### 435. What is code coverage?
It measures which code was executed by tests under a chosen metric. High coverage does not prove correctness or that assertions are meaningful.

### 436. What is mutation testing?
It introduces small code changes and checks whether tests detect them. Surviving mutations may indicate missing assertions or untested behavior.

### 437. What is a source map?
It maps generated or bundled code positions back to authored source. Protect production maps appropriately because they can expose source and implementation details.

### 438. What does a linter do?
It reports patterns that may indicate errors or inconsistent style. Linting complements rather than replaces tests, type checking, and code review.

### 439. What does a formatter do?
It applies consistent source formatting automatically. Formatting reduces style debates but does not validate program behavior.

### 440. What is static type checking in JavaScript projects?
Tools such as TypeScript or Flow analyze types before runtime, often using JavaScript-compatible syntax or declarations. Static checks do not validate untrusted runtime inputs by themselves.

### 441. What is a declaration file?
A TypeScript `.d.ts` file describes types for JavaScript modules or APIs without implementing runtime behavior. Declarations must stay aligned with the actual API.

### 442. What is a breakpoint?
It pauses execution at a chosen source location so state and call stacks can be inspected. Use conditional breakpoints for hard-to-reproduce cases.

### 443. What is a stack trace?
It describes the call path associated with an error or execution point. Async boundaries and source maps can change how stacks are presented.

### 444. Why use a reproducible build?
It ensures the same declared inputs and toolchain produce equivalent output. Lockfiles and pinned tool versions help reduce environment drift.

### 445. What is continuous integration?
It automatically runs checks such as tests, linting, and builds on code changes. CI catches regressions early but should reflect supported environments.

### 446. How should tests treat implementation details?
Avoid asserting private structure unless it is itself part of the contract. Behavior-focused tests are more resilient to refactoring.

### 447. What is contract testing?
It verifies that a service provider and consumer agree on an interface and expected messages. It can catch integration drift without requiring full end-to-end setup.

### 448. How should random behavior be tested?
Inject or seed a predictable random source when possible, then test boundaries and invariants. Do not assert one unseeded random outcome.

### 449. What is a regression test?
It captures a previously fixed bug or requirement so future changes are checked against it. Include a failing-before/fixed-after case where practical.

### 450. What makes a useful code review?
It checks correctness, maintainability, security, and fit with requirements while keeping feedback specific and actionable. Automated checks should handle routine formatting and simple defects.

## 23. Modern ECMAScript features

### 451. What is optional chaining?
`?.` short-circuits property access or calls when the preceding value is nullish, returning `undefined`. It does not suppress errors from unrelated operations or undeclared identifiers.

### 452. Can optional chaining be used on an undeclared variable?
No. Resolving an undeclared identifier still throws a `ReferenceError`; optional chaining protects only a nullish value that was successfully resolved.

### 453. What is nullish coalescing?
`left ?? right` returns the right operand only when the left is `null` or `undefined`. It differs from `||`, which also replaces other falsy values.

### 454. What is the logical assignment family?
`&&=`, `||=`, and `??=` conditionally update an existing reference using corresponding short-circuit rules. They avoid evaluating the right side when no assignment is needed.

### 455. What are numeric separators?
Underscores can group digits in numeric literals, such as `1_000_000`, without changing the value. They cannot be placed arbitrarily next to prefixes, decimals, or suffixes.

### 456. What is a BigInt literal?
A decimal integer followed by `n`, such as `9007199254740993n`, creates a BigInt. BigInt cannot be mixed with Number arithmetic without explicit conversion.

### 457. What are private class elements?
They are class fields, methods, accessors, or static elements named with `#`. Access is enforced by the language and is distinct from TypeScript's compile-time `private`.

### 458. What is a numeric literal with a binary prefix?
`0b` or `0B` introduces a binary integer literal, for example `0b1010`. Numeric separators may group digits in supported positions.

### 459. What are Unicode code points versus UTF-16 code units?
A code point is a Unicode scalar value; a UTF-16 code unit is a 16-bit unit. Some characters occupy two code units, so string indexing and `length` count units, not user-perceived characters.

### 460. How can a string be iterated by code point?
Use `for...of` or the string iterator, which yields Unicode code points rather than individual UTF-16 code units. Grapheme clusters may still contain multiple code points.

### 461. What is `String.prototype.normalize`?
It converts text to a Unicode normalization form such as NFC or NFD. Normalize when a domain requires canonical-equivalent strings to compare consistently.

### 462. What is `Intl.Segmenter`?
It segments text by locale-sensitive grapheme, word, or sentence boundaries where supported. It is more appropriate than indexing for user-perceived characters.

### 463. What are logical operators' mixing restrictions?
`??` cannot be directly mixed with `&&` or `||` without parentheses. Parenthesize intended grouping to satisfy syntax and make behavior clear.

### 464. What is a numeric separator's runtime effect?
Separators are part of source syntax only and have no effect on the numeric value. They cannot be used in runtime strings without parsing logic.

### 465. What are `Object.fromEntries` and `Object.entries`?
`Object.entries` produces enumerable own string-keyed pairs; `Object.fromEntries` constructs an object from entry pairs. Symbols require other APIs for full key coverage.

### 466. What does `Array.prototype.at` provide?
It accesses an indexed element and supports negative offsets from the end. It returns `undefined` when the computed index is out of range.

### 467. What is `Promise.withResolvers`?
Where supported, it returns a promise together with its externally accessible `resolve` and `reject` functions. Use it when separating promise creation from settlement improves clarity.

### 468. What is `Object.groupBy`?
It groups iterable elements into an object keyed by callback results, with each value an array. Use `Map.groupBy` when group keys should retain non-string identity.

### 469. What are set-composition methods?
Methods such as union, intersection, difference, and symmetric difference return sets based on relationships between set-like inputs, where supported by the runtime.

### 470. What is the `using` declaration?
In runtimes supporting explicit resource management, `using` registers synchronous disposal at scope exit, while `await using` supports asynchronous disposal. Verify target support before adopting it.

### 471. What is `Symbol.dispose`?
It is a well-known symbol naming a synchronous resource-disposal method in explicit resource management. The disposing scope invokes it when leaving the scope.

### 472. What is `Symbol.asyncDispose`?
It names an asynchronous disposal method used by `await using` where supported. Cleanup may be awaited as the scope exits.

### 473. What is `Error`'s `cause` option?
It lets a new error retain the underlying error or reason that caused it. This preserves diagnostic context while allowing a higher-level message.

### 474. What is `AggregateError`?
It represents multiple errors together, exposing them through an `errors` collection. `Promise.any` uses it when every input rejects.

### 475. Why check ECMAScript feature compatibility?
Language syntax and built-in APIs may be unsupported in target runtimes. Use compatibility data, testing, and transpilation or polyfills as appropriate.

## 24. Advanced coding scenarios

### 476. How would you implement a safe numeric input parser?
Require a documented format, validate the complete input string, convert it, and reject non-finite or out-of-range results. Avoid accepting partial parses unintentionally.

### 477. How would you group an array by a key?
Use `Map` when keys may be arbitrary values; otherwise reduce into a safe dictionary. Define behavior for missing keys and collisions.

### 478. How would you flatten a nested array of known depth?
Use `flat(depth)` or recursive traversal for custom behavior. Validate depth and account for sparse arrays if holes matter.

### 479. How would you deep-clone supported data?
Use `structuredClone` for supported built-ins and cycles where available. It does not preserve arbitrary functions, custom prototypes, or every host object's semantics.

### 480. How would you implement a stable unique-by-key operation?
Track seen keys in a `Set` and append the first item for each key in input order. Define key normalization and missing-key behavior explicitly.

### 481. How would you debounce an async search?
Cancel the previous timer and, if possible, abort its request; only apply the latest response. Handle rejection and cleanup on component or page teardown.

### 482. How would you limit asynchronous concurrency?
Maintain a bounded worker pool or queue that starts another task only when one completes. Propagate failures and cancellation according to a documented policy.

### 483. How would you retry a transient request?
Retry only failures considered transient, use bounded exponential backoff with jitter, and respect cancellation and server guidance. Ensure repeated requests are safe or idempotent.

### 484. How would you memoize a function with object arguments?
Use a deliberate key strategy such as nested `WeakMap`s for object identity or stable serialization for value semantics. Set cache bounds and invalidation rules.

### 485. How would you remove an event listener reliably?
Retain the same function reference and matching capture option, or register with an `AbortSignal` and abort it during cleanup. Avoid anonymous handlers that cannot be removed.

### 486. How would you avoid stale state in a UI request?
Use a request identifier, cancellation, or framework-supported latest-request pattern. Apply results only when they still correspond to current state.

### 487. How would you parse a CSV file?
Use a CSV parser that handles quoting, escapes, line breaks, and encoding rather than splitting on commas. Validate headers and cell values after parsing.

### 488. How would you compare two objects structurally?
Define which types, keys, ordering, cycles, and special values count as equal, then implement or use a tested comparator. `===` compares object identity only.

### 489. How would you safely merge configuration objects?
Validate allowed keys and value shapes, define precedence, and avoid unsafe prototype-mutating assignments. Prefer schema-aware merging over generic recursive merging.

### 490. How would you implement a timeout without leaking resources?
Use a timer and settle only once, clear the timer when the operation finishes, and abort the operation when possible. A timer race alone does not cancel work.

### 491. How would you preserve input order while running tasks concurrently?
Map each input to a promise at its index and aggregate with `Promise.all`, which returns results in input order. Add a concurrency limit if necessary.

### 492. How would you build a cancellation-aware queue?
Accept a signal per job, remove or skip aborted work, stop scheduling when globally cancelled, and settle each job once. Define cleanup for active jobs.

### 493. How would you process a large array without blocking the browser?
Process bounded chunks and yield between them, or move CPU-heavy work to a worker. Measure chunk duration and keep cancellation responsive.

### 494. How would you handle partial success from multiple requests?
Use `Promise.allSettled` and inspect every outcome, then present successes and failures according to product requirements. Preserve input association in the results.

### 495. How would you prevent duplicate form submissions?
Disable or guard submission while an operation is pending, and also use server-side idempotency for consequential requests. Re-enable controls on every completion path.

### 496. How would you safely render untrusted text?
Insert it with `textContent` or a framework's escaped text binding, not raw HTML. Sanitize with a well-maintained sanitizer only when rich HTML is required.

### 497. How would you implement a bounded cache?
Track usage or age, evict according to a defined policy such as LRU, and impose a size or time limit. Consider `Map` insertion ordering and synchronization needs.

### 498. How would you handle a promise that may reject after a timeout?
Attach handlers to both outcomes and define which result wins; abort the underlying operation when possible. Ensure late rejection is still observed to avoid unhandled rejection reports.

### 499. How would you design a resilient API boundary?
Validate shape and size, normalize supported inputs, return predictable errors, and avoid leaking internal details. Keep runtime validation even when static types are used.

### 500. How would you choose between a built-in and a custom implementation?
Prefer standardized built-ins for correctness and maintainability when their semantics fit. Write a custom implementation only for a demonstrated gap, then test edge cases and supported runtimes.
