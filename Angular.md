# Angular Interview Questions and Answers

> This guide focuses on modern Angular while noting that APIs and defaults have changed across major releases. Standalone components are the modern default in recent versions; NgModules remain supported in many existing applications.

## Contents

1. [Angular fundamentals](#1-angular-fundamentals)
2. [Components, templates, and directives](#2-components-templates-and-directives)
3. [Dependency injection and services](#3-dependency-injection-and-services)
4. [Forms, routing, and HTTP](#4-forms-routing-and-http)
5. [Change detection and RxJS](#5-change-detection-and-rxjs)
6. [Testing, performance, and security](#6-testing-performance-and-security)

## 1. Angular fundamentals

### 1. What is Angular?

Angular is a TypeScript-based framework for building web applications. It provides components, templates, dependency injection, routing, forms, HTTP support, and build tooling. “Angular” commonly refers to modern Angular; AngularJS refers to the older 1.x framework, which has a substantially different architecture.

### 2. What is the difference between Angular and AngularJS?

AngularJS is the legacy 1.x framework, primarily based on JavaScript and scope/controller patterns. Modern Angular is a rewritten framework using TypeScript, components, hierarchical DI, and a different template and change-detection architecture. Migration is not a simple version-number upgrade.

### 3. What is a component?

A component is a TypeScript class combined with metadata and a template that controls a view. It has a selector, template or template URL, and optional styles and providers. Components are the main building blocks of Angular UI.

### 4. What is a standalone component?

A standalone component declares its template dependencies directly through its `imports` and does not need to be declared in an NgModule. Standalone APIs simplify application composition and are the default for new components in recent Angular versions. Existing NgModule-based applications remain supported.

### 5. What is an NgModule?

An NgModule groups declarations, imports, providers, and exports for Angular applications. It was the traditional composition mechanism. Standalone components reduce the need for NgModules in new code, but modules remain useful and supported in many codebases and libraries.

### 6. What is a template?

An Angular template is HTML enhanced with bindings, directives, and expressions. Angular compiles templates and updates the view as component state changes. Templates should remain declarative; complex business logic belongs in TypeScript services or component methods with clear responsibilities.

### 7. What is a decorator?

A decorator is TypeScript/JavaScript syntax used by Angular to attach metadata or modify declarations. Angular decorators include `@Component`, `@Injectable`, `@Input`, and `@Directive`. Decorator support and emitted metadata depend on the TypeScript and Angular toolchain.

## 2. Components, templates, and directives

### 8. What are data-binding types in Angular?

Common template binding forms include:

- Interpolation: `{{ value }}` for text.
- Property binding: `[disabled]="isDisabled"`.
- Event binding: `(click)="save()"`.
- Two-way binding: `[(ngModel)]="name"` when the forms support is imported.

Bindings connect template output and user events to component state.

### 9. What is interpolation?

Interpolation evaluates a template expression and places its string representation in a text context, such as `{{ title }}`. Angular escapes text output rather than treating it as HTML by default. Use property binding for DOM properties and avoid unsafe HTML bypass APIs.

### 10. What is a directive?

A directive adds behavior to an element. Attribute directives modify behavior or appearance; structural directives change which views are rendered. Components are directives with templates. Newer Angular also offers built-in control-flow syntax such as `@if` and `@for`; exact syntax depends on version.

### 11. What is the difference between structural and attribute directives?

Structural directives add, remove, or transform views, traditionally using `*ngIf` or `*ngFor`. Attribute directives change the appearance or behavior of an existing element, such as a custom highlight directive. Structural shorthand is desugared into an embedded template.

### 12. What is a pipe?

A pipe transforms a value for display in a template, such as `date` or `currency`. Pure pipes generally run when input references/values change; impure pipes can run frequently and may affect performance. Keep pipes focused and avoid expensive repeated transformations.

### 13. What are `@Input` and `@Output`?

Inputs pass data from a parent into a child component. Outputs emit events from a child to a parent, traditionally using `EventEmitter`. Newer Angular versions provide signal-based input/output APIs; use the style consistent with the project and target version. Inputs should be validated and treated as externally supplied data.

### 14. What are lifecycle hooks?

Lifecycle hooks let a component or directive respond to creation, input changes, content/view initialization, checking, and destruction. Common hooks include `ngOnInit`, `ngOnChanges`, `ngAfterViewInit`, and `ngOnDestroy`. Use the hook whose lifecycle semantics fit the need; do not use initialization hooks to repeatedly derive values that could be computed directly.

### 15. What is content projection?

Content projection places parent-provided markup inside a child component template using `<ng-content>`. Selectors can define multiple projection slots. It enables reusable wrapper components while retaining caller control of projected content.

### 16. What is a view child query?

`ViewChild` and related query APIs retrieve a reference to a child component, directive, or DOM element in a component's view. Query results become available according to lifecycle and query configuration. Prefer inputs/outputs for normal component communication; use direct references when imperative interaction is genuinely needed.

## 3. Dependency injection and services

### 17. What is dependency injection in Angular?

Angular DI provides dependencies such as services to components and other services through injectors. Dependencies are declared through constructor parameters or the `inject()` function in supported injection contexts. DI makes collaborators replaceable and scopes instances according to provider configuration.

### 18. What is an Angular service?

A service is a class commonly used for shared logic, data access, or state that should not be embedded in a component. `@Injectable` metadata allows it to participate in DI. Services should have clear responsibilities and avoid becoming unstructured global state.

### 19. What does `providedIn: 'root'` mean?

`providedIn: 'root'` registers an injectable with the root injector, commonly giving the application one shared instance and allowing tree-shaking when unused. Other provider locations can create narrower instances. Exact provider scope depends on injector hierarchy and configuration.

### 20. What is hierarchical dependency injection?

Angular injectors form a hierarchy. A dependency can be provided at application, route, component, or other supported injector levels. The nearest matching provider is used, so a component-level provider can create an instance distinct from the root service instance.

### 21. What is the difference between a singleton service and a component-provided service?

A root-provided service is generally shared across the application injector. A component-provided service is scoped to that component's injector and descendants and can be recreated with the component. Choose scope based on state lifetime and sharing requirements.

### 22. What is an injection token?

An `InjectionToken<T>` provides a typed key for injecting values or abstractions that do not have a class token, such as configuration objects or interface-like contracts. Register a provider for the token and inject it where needed. TypeScript interfaces are erased at runtime and cannot themselves be DI tokens.

## 4. Forms, routing, and HTTP

### 23. What are template-driven forms?

Template-driven forms express much of the form structure and validation in the template, using directives such as `ngModel`. They can be convenient for simpler forms but may be harder to scale for complex dynamic validation. Required forms modules must be imported in the relevant standalone component or NgModule.

### 24. What are reactive forms?

Reactive forms define controls and validation in TypeScript using `FormControl`, `FormGroup`, or `FormArray`. They provide explicit, observable state and suit complex or dynamic forms. Validate on the client for usability and again on the server for security.

### 25. What is the difference between `setValue` and `patchValue`?

For Angular form groups, `setValue` requires a value matching the complete control structure, helping detect missing or unexpected shape. `patchValue` updates only supplied controls and leaves others unchanged. Choose strictness intentionally and validate server payloads independently.

### 26. What is Angular routing?

The Angular Router maps URLs to components and supports navigation, parameters, guards, resolvers, and nested routes. Routes are configured in application composition and displayed through `RouterOutlet`. Client-side route guards improve navigation UX but do not replace server-side authorization.

### 27. What are route guards?

Guards control client-side navigation decisions, such as whether a route can be entered or left. They can use authentication state or unsaved-change checks. They are not security enforcement: APIs must independently authorize every request.

### 28. What is a route resolver?

A resolver loads or prepares data before route activation, depending on router configuration. It can ensure required data is ready but may delay navigation. Handle loading failures and consider whether a component-level loading state gives a better user experience.

### 29. What is `HttpClient`?

Angular `HttpClient` sends HTTP requests and returns RxJS Observables. It supports typed response declarations, interceptors, and request options, but TypeScript generics do not validate untrusted JSON at runtime. Validate data at trust boundaries when correctness or security requires it.

### 30. What is an HTTP interceptor?

An interceptor transforms or observes outgoing requests and incoming responses, enabling cross-cutting concerns such as authentication headers, logging, or error handling. Keep interceptors focused and avoid intercepting in ways that cause recursive requests or leak credentials to untrusted origins.

## 5. Change detection and RxJS

### 31. What is change detection?

Change detection updates the rendered view when Angular determines application state may have changed. Its exact triggers and performance characteristics depend on Angular version, zone configuration, signals, and component strategy. Keep templates efficient and avoid expensive functions evaluated repeatedly.

### 32. What is `OnPush` change detection?

`OnPush` limits when Angular checks a component subtree based on signals such as changed inputs, events in the subtree, observable emissions through mechanisms like `async`, or explicit marking. It encourages clear data flow but does not mean a component can never be checked. Immutable updates and appropriate notifications are important.

### 33. What are Angular signals?

Signals are reactive values that track reads and notify dependent computations/effects when updated. Angular provides APIs such as `signal`, `computed`, and `effect` in current versions. Signals complement existing RxJS patterns; they are not identical to Observables and have different scheduling and composition semantics.

### 34. What is RxJS?

RxJS is a library for composing asynchronous and event-based data streams using Observables and operators. Angular uses it extensively, including in `HttpClient` and reactive forms. Operators such as `map`, `filter`, `switchMap`, and `catchError` transform or coordinate streams.

### 35. What is an Observable?

An Observable represents a potentially asynchronous stream of values over time, followed by completion or error. Subscribing starts or connects to execution depending on the Observable. Observables can be lazy and can emit zero, one, or many values.

### 36. Observable versus Promise: what is the difference?

A Promise typically settles once with a value or error and starts according to how it is created. An Observable can emit multiple values, can often be canceled by unsubscribing, and supports operators for stream composition. Convert only when the one-shot semantics are appropriate.

### 37. What do `switchMap`, `mergeMap`, `concatMap`, and `exhaustMap` do?

- `switchMap` switches to the latest inner stream and unsubscribes from the previous one; useful for latest-search requests.
- `mergeMap` runs inner streams concurrently, potentially with a concurrency limit.
- `concatMap` queues inner work sequentially.
- `exhaustMap` ignores new source values while an inner stream is active.

Choose based on whether work should be replaced, concurrent, queued, or ignored.

### 38. How do you prevent Observable subscription leaks?

Prefer template `async` pipe or lifecycle-aware interop APIs where available. Otherwise unsubscribe or complete subscriptions when their owner is destroyed. Some streams, such as many `HttpClient` one-shot requests, complete automatically, but long-lived event streams often do not.

### 39. What does the `async` pipe do?

The `async` pipe subscribes to an Observable or Promise in a template, exposes its latest value, marks the view for update as values arrive, and manages subscription cleanup. It reduces manual subscription code but does not eliminate the need to handle error and loading states appropriately.

## 6. Testing, performance, and security

### 40. How do you test Angular applications?

Use unit tests for isolated logic, component tests for template and interaction behavior, and integration/end-to-end tests for important workflows. Angular provides testing utilities; test runner defaults vary by Angular version and workspace setup. Mock or control external dependencies and test observable behavior.

### 41. What is `TestBed`?

`TestBed` configures an Angular testing environment with declarations/imports/providers and creates components or injectors for tests. Standalone components can be imported directly in modern setups. Keep tests isolated and avoid depending on internal framework details unnecessarily.

### 42. How do you improve Angular performance?

Profile first. Reduce unnecessary change detection and expensive template expressions, use appropriate `OnPush`/signal patterns, lazy-load routes, track list items with stable identities, avoid excessive subscriptions, and optimize network payloads. Performance depends on real workloads and version-specific behavior.

### 43. What is lazy loading?

Lazy loading defers loading a route or feature until it is needed, reducing initial bundle work. Angular routing supports lazy route/component loading through dynamic imports. It can increase navigation latency on first access, so use appropriate preloading and loading UI when helpful.

### 44. What is AOT compilation?

Ahead-of-time (AOT) compilation compiles Angular templates and components during the build rather than in the browser at runtime. It can improve startup, catch template errors earlier, and avoid shipping the JIT compiler. Modern production builds generally use AOT; inspect the actual build configuration.

### 45. How does Angular help prevent XSS?

Angular sanitizes values in many template binding contexts and treats interpolation as text. Bypass APIs such as `bypassSecurityTrustHtml` disable protections and must not be used with untrusted content. Avoid direct unsafe DOM manipulation and validate/sanitize rich HTML with a trusted policy where needed.

### 46. What are pipes that are pure or impure?

A pure pipe runs when Angular detects its input arguments have changed; an impure pipe can run during frequent change-detection cycles. Impure pipes can be costly. Prefer immutable input changes or move expensive transformations to a more appropriate computed or cached layer.

### 47. What is the difference between `Subject` and `BehaviorSubject`?

A `Subject` multicasts values to current subscribers but does not replay a prior value to new subscribers. A `BehaviorSubject` requires an initial value and emits its current value to new subscribers. Choose based on whether late subscribers need the latest state; do not expose writable subjects broadly when a read-only Observable is sufficient.

### 48. What is the `trackBy` function (or tracking expression)?

Tracking identifies list items by stable keys so Angular can reuse DOM views efficiently when a collection changes. Modern control flow supports a `track` expression; older `*ngFor` uses `trackBy`. Use stable unique IDs rather than indexes for lists that can reorder or change membership.

### 49. How do you handle errors from `HttpClient` Observables?

Use RxJS error handling such as `catchError` at a layer with enough context to recover, map to a domain result, or report the failure. Preserve cancellation and avoid swallowing errors into success-shaped values. Present safe user feedback and log diagnostic details without secrets.

### 50. Are client-side route guards secure?

No. A user controls the browser and can modify client code or call APIs directly. Route guards are useful for navigation and UX, but authorization must be enforced server-side for every protected resource and operation.
