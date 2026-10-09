# C# Interview Questions and Answers

> A broad interview-preparation guide to the C# language, from fundamentals to advanced features. C# evolves over time; language-version-specific behavior is called out where it matters. This guide aims for broad coverage, but no finite list can include every question an interviewer might ask.

## Contents

1. [Language and compilation](#1-language-and-compilation)
2. [Types, variables, and conversions](#2-types-variables-and-conversions)
3. [Object-oriented programming](#3-object-oriented-programming)
4. [Members, properties, and initialization](#4-members-properties-and-initialization)
5. [Methods, parameters, and delegates](#5-methods-parameters-and-delegates)
6. [Generics and collections](#6-generics-and-collections)
7. [LINQ and iterators](#7-linq-and-iterators)
8. [Exceptions and resource management](#8-exceptions-and-resource-management)
9. [Asynchronous and concurrent programming](#9-asynchronous-and-concurrent-programming)
10. [Memory, value semantics, and performance](#10-memory-value-semantics-and-performance)
11. [Modern C# features](#11-modern-c-features)
12. [Design, safety, and testing](#12-design-safety-and-testing)
13. [Common coding and output questions](#13-common-coding-and-output-questions)

## 1. Language and compilation

### 1. What is C#?

C# is a statically typed, general-purpose, object-oriented programming language in the .NET ecosystem. It supports procedural, functional, and asynchronous programming styles as well as object-oriented design. C# source code is compiled to Common Intermediate Language (CIL, also called IL), which the .NET runtime executes using just-in-time (JIT) compilation or, in some deployment modes, ahead-of-time compilation.

### 2. Is C# the same as .NET?

No. C# is a programming language. .NET is a development platform that includes a runtime, base libraries, SDK, compiler, and application frameworks. C# is commonly used with .NET, but the language and platform are distinct concepts. Other languages, such as F# and Visual Basic, can also target .NET.

### 3. What is the CLR?

The Common Language Runtime (CLR) is the managed execution environment used by .NET. It provides services such as garbage collection, exception handling, type safety, JIT compilation, and runtime support for threading and interoperability. Modern .NET implementations use the CoreCLR runtime; “CLR” is also used informally for the managed runtime concept.

### 4. What are IL, JIT, and metadata?

- **IL/CIL:** A platform-neutral instruction set emitted by the compiler.
- **JIT:** A runtime compiler that translates IL into machine code for the current process and architecture. It can optimize code using runtime information.
- **Metadata:** Describes assemblies, types, members, references, and other information. The runtime and tools use it for loading, reflection, verification, and execution.

The exact compilation path depends on runtime, platform, and deployment configuration.

### 5. What is an assembly?

An assembly is a compiled unit of deployment and versioning, commonly a `.dll` or `.exe`. It contains IL or native code, metadata, and resources. An assembly can contain multiple namespaces and types; a namespace is a logical naming construct, not a physical assembly boundary.

### 6. What is the difference between a namespace and an assembly?

A namespace organizes names and helps prevent naming collisions. An assembly packages compiled code and resources for loading and deployment. Types in one namespace may be spread across assemblies, and one assembly may contain types from many namespaces.

### 7. What is managed code?

Managed code executes under runtime services such as garbage collection and runtime type information. C# code targeting .NET is generally managed, although it can use `unsafe` blocks, native interop, or unmanaged resources. “Managed” does not mean that every resource is automatically released immediately: deterministic disposal is still needed for resources such as file handles.

### 8. What is the difference between compile-time and runtime errors?

A compile-time error is detected by the compiler, such as assigning an incompatible type without a valid conversion. A runtime error occurs during execution, such as dividing an integer by zero or accessing an array out of bounds. Some problems, including null dereferences and invalid external input, can only be fully detected at runtime.

## 2. Types, variables, and conversions

### 9. What are value types and reference types?

A value-type variable contains its value directly (or inline as part of another object), whereas a reference-type variable contains a reference to an object. Value types include `struct`, `enum`, and built-in numeric and Boolean types. Classes, arrays, delegates, and strings are reference types.

This distinction describes assignment and storage semantics, not an absolute rule that value types are always on the stack or reference types always on the heap. A value type can be a field inside a heap object or boxed, and the runtime is free to optimize storage.

### 10. What happens when a value or reference variable is assigned?

Assigning a value type copies its value. Assigning a reference type copies the reference, so both variables can refer to the same object:

```csharp
var first = new List<int> { 1 };
var second = first;
second.Add(2);
// first now observes { 1, 2 }
```

The variable `second` did not become an alias for the variable `first`; each variable stores a reference to the same list object.

### 11. What is boxing and unboxing?

**Boxing** converts a value type to `object` or an implemented interface type by wrapping a copy of the value in a reference-type object. **Unboxing** extracts the value type from a compatible boxed object and requires an explicit cast.

```csharp
int number = 42;
object boxed = number;       // boxing
int copy = (int)boxed;       // unboxing
```

Unboxing the wrong value type throws `InvalidCastException`; unboxing `null` to a non-nullable value type throws `NullReferenceException`. Generic collections such as `List<int>` avoid the boxing that older non-generic collections often required.

### 12. What is the difference between `var`, `dynamic`, and `object`?

- `var` asks the compiler to infer a **static** type from the initializer. The inferred type is fixed at compile time.
- `object` is a statically known base type. Operations generally require a cast or type test.
- `dynamic` defers member binding and many type checks until runtime. Invalid operations can fail at runtime, and dynamic binding can add overhead.

`var` is not a runtime type and does not make a variable dynamically typed.

### 13. What are nullable value types?

A nullable value type represents either a value of its underlying value type or no value. `int?` is shorthand for `Nullable<int>`.

```csharp
int? count = null;
int result = count ?? 0;
```

It provides `HasValue` and `Value`; reading `Value` when no value exists throws `InvalidOperationException`. Prefer `??`, pattern matching, or `GetValueOrDefault` when they express the intended behavior more clearly.

### 14. What are nullable reference types?

Nullable reference types (NRT) are compiler annotations and flow analysis that help identify possible null dereferences. `string` means the code intends a non-null reference, while `string?` permits null. They do not change the runtime representation or automatically add runtime checks.

NRT analysis is enabled and controlled through project/compiler settings and nullable context directives. Warnings should be addressed by correcting the logic, adding validation, or using a justified null-forgiving operator—not by suppressing warnings indiscriminately.

### 15. What are implicit, explicit, and user-defined conversions?

An implicit conversion is allowed without cast syntax when the language considers it safe for the applicable conversion, such as converting `int` to `long`. An explicit conversion requires a cast and may lose information or fail, such as converting a large `long` to `int`. Types can define conversion operators using `implicit` or `explicit`.

The `checked` and `unchecked` contexts affect overflow behavior for certain integral arithmetic and conversions. A cast does not necessarily mean a conversion is safe.

### 16. What is the difference between `const` and `readonly`?

`const` is a compile-time constant and must be initialized at its declaration. It is implicitly static, and its value is substituted into consuming assemblies at compile time. Changing a public constant may therefore require recompilation of consumers.

`readonly` fields may be initialized at declaration or in an instance/static constructor, depending on whether the field is instance or static. Their value cannot be reassigned after construction. For a reference-type field, `readonly` prevents replacing the reference but does not make the referenced object immutable.

### 17. What are enums and flags enums?

An enum is a named set of constants backed by an integral type. A flags enum represents combinable bit values, conventionally decorated with `[Flags]`; members should generally use distinct powers of two.

```csharp
[Flags]
enum Access
{
    None = 0,
    Read = 1,
    Write = 2,
    Execute = 4
}

Access permissions = Access.Read | Access.Write;
bool canWrite = (permissions & Access.Write) != 0;
```

The attribute improves formatting and communicates intent; it does not itself enforce valid combinations.

### 18. What is the difference between `==` and `Equals`?

`==` is an operator whose meaning depends on the operand types and can be overloaded. `Equals` is a method whose implementation can be overridden to define equality. For reference types, the default `object.Equals` behavior is reference equality; types such as `string` provide value-based equality.

When implementing value equality, keep `Equals`, `GetHashCode`, and equality operators consistent. Equal objects must produce equal hash codes.

### 19. What are `ref`, `out`, and `in` parameters?

- `ref` passes a variable by reference; the caller must initialize it first.
- `out` passes a variable by reference for the method to assign; the caller need not initialize it first, and the method must assign it before returning normally.
- `in` passes a readonly reference to an argument, commonly used to avoid copying a large struct while preventing reassignment through that parameter.

These modifiers affect the call contract and overload resolution. They are not the same as returning a reference.

## 3. Object-oriented programming

### 20. What are the main object-oriented programming principles?

- **Encapsulation:** Hide internal representation and expose controlled operations.
- **Abstraction:** Expose essential behavior while hiding implementation details.
- **Inheritance:** Derive a type from a base type to reuse or specialize behavior.
- **Polymorphism:** Use a common contract while different runtime types provide different implementations.

These are design concepts, not a requirement to model every system with deep class hierarchies.

### 21. What is a class, and what is an object?

A class defines a reference type’s members and behavior. An object is a runtime instance of a class, created directly or indirectly (for example, by a factory). A class can define fields, properties, methods, events, constructors, operators, and nested types.

### 22. What is a struct, and when should it be used?

A `struct` is a value type. It is suitable for small values whose identity is not important and whose semantics are naturally value-like, such as coordinates or measurements. Structs are copied by value, though they can contain reference fields.

Large or mutable structs can be surprising and expensive to copy. Immutable structs and `readonly struct` can help communicate value semantics. `record struct` provides record-style value equality for structs.

### 23. What is an interface?

An interface defines a contract that a type can implement. A class or struct can implement multiple interfaces, which is a common way to compose capabilities without multiple class inheritance. Interfaces can declare methods, properties, events, and indexers; modern C# also supports certain default interface implementations and static abstract members, subject to language/runtime support.

An interface is useful when callers need to depend on behavior rather than a concrete implementation.

### 24. What is an abstract class?

An abstract class cannot be instantiated directly. It can contain both implemented members and abstract members that derived concrete classes must implement. It can also maintain shared state and constructors. A class can inherit from only one class, whether abstract or concrete.

Use an abstract class when related types share implementation or state; use an interface to express a capability or contract that unrelated types can share.

### 25. What is the difference between an interface and an abstract class?

An interface primarily defines a contract and supports implementation by multiple types; a class can implement many interfaces. An abstract class can contain instance state, constructors, protected implementation details, and both abstract and concrete members, but a class has only one direct base class.

The right choice depends on whether shared implementation/state is essential or the design needs a capability contract. Default interface members do not make interfaces equivalent to base classes.

### 26. Explain inheritance and method overriding.

Inheritance derives a class from a base class using `:`. A base method must be marked `virtual`, `abstract`, or `override` to be overridden. A derived implementation uses `override`; it may call the base implementation with `base.Member()`.

```csharp
class Animal
{
    public virtual string Speak() => "sound";
}

class Dog : Animal
{
    public override string Speak() => "bark";
}
```

Prefer composition over inheritance when a “has-a” relationship or replaceable behavior better represents the design.

### 27. What is the difference between `override` and `new`?

`override` participates in runtime polymorphism for a virtual member: dispatch depends on the object’s runtime type. `new` hides a base member with a separate member; which member is called depends on the compile-time type of the reference.

Hiding is not a substitute for overriding and can lead to confusing behavior if used unintentionally.

### 28. What are access modifiers?

Access modifiers control where types and members can be used. Common modifiers are:

- `public`: accessible wherever the containing type is accessible.
- `private`: accessible within the containing type.
- `protected`: accessible within the containing type and derived types.
- `internal`: accessible within the same assembly.
- `protected internal`: accessible from the same assembly **or** derived types.
- `private protected`: accessible from derived types within the same assembly.

The accessibility of a member cannot expose a type that is less accessible than the member itself.

### 29. Does C# support multiple inheritance?

C# classes have one direct base class, avoiding the ambiguity associated with inheriting implementation from multiple classes. A class or struct can implement multiple interfaces. Shared behavior can also be composed through fields, services, and delegation.

### 30. What is polymorphism?

Polymorphism lets code work against a shared base type or interface while invoking behavior appropriate to the concrete runtime type. Virtual dispatch applies to overridden instance members. This makes implementations replaceable, but a good design should keep contracts meaningful and avoid type checks that undo the abstraction.

### 31. What is a sealed class or sealed override?

A `sealed` class cannot be inherited. A `sealed override` prevents further derived classes from overriding that member. Sealing communicates design constraints and may allow optimizations, but should be used when the inheritance restriction is intentional.

### 32. What is a static class?

A static class cannot be instantiated or inherited and may contain only static members. It is useful for stateless operations, though dependency injection and instance services are often easier to test and replace. Static state can create hidden coupling and concurrency problems.

## 4. Members, properties, and initialization

### 33. What is the difference between a field and a property?

A field stores data directly. A property exposes get/set accessors, which can validate, compute, or control access to data. Properties provide a stable public API while allowing implementation to evolve.

```csharp
public decimal Price
{
    get => _price;
    set
    {
        if (value < 0) throw new ArgumentOutOfRangeException(nameof(value));
        _price = value;
    }
}
private decimal _price;
```

### 34. What is an auto-property?

An auto-property uses compiler-generated backing storage:

```csharp
public string Name { get; set; } = "";
public int Id { get; }
```

Accessors can have different accessibility, such as `public string Name { get; private set; }`. A property with no setter is read-only to ordinary callers, although initialization rules and `init` accessors may also apply.

### 35. What is an indexer?

An indexer lets an instance be accessed with bracket syntax, similar to an array. It is declared using `this[...]`, can accept one or more parameters, and can have get/set accessors. Indexers are useful for collection-like types.

### 36. What is a constructor? Can constructors be overloaded?

A constructor initializes an instance and has the same name as its containing type. Constructors can be overloaded with different parameter lists. A constructor initializer `: this(...)` delegates to another constructor in the same type; `: base(...)` invokes a base-class constructor.

If no instance constructor is declared, the compiler typically supplies a parameterless one when permitted. A static constructor initializes type-level state once before the type is first used under runtime rules; it has no access modifier or parameters.

### 37. What is a finalizer?

A finalizer is a nondeterministic cleanup hook that the garbage collector may run before reclaiming an object. It is not guaranteed to run promptly or at all before process termination. Finalizers are generally unnecessary for ordinary managed objects.

For resources requiring reliable release, implement `IDisposable` and use `using`; a finalizer is appropriate only when a type directly owns unmanaged resources and needs a fallback cleanup path.

### 38. What are static members?

Static members belong to the type rather than a particular instance. They are accessed through the type name, and a static method cannot access instance members without an object reference. Static fields represent shared state, so changes may require synchronization when accessed concurrently.

### 39. What is a partial class?

A partial class allows one type declaration to be split across multiple source files, each marked `partial`. The compiler combines the declarations into one type. It is often used by code generators or designer tooling; manually splitting a small class can make code harder to navigate.

### 40. What is an extension method?

An extension method is a static method that can be called with instance-method syntax on a receiver of the extended type. Traditionally it is declared in a non-nested, non-generic static class, with `this` on its first parameter.

```csharp
public static class StringExtensions
{
    public static bool HasText(this string? value) =>
        !string.IsNullOrWhiteSpace(value);
}
```

Extension methods do not modify the target type. Instance members take precedence in member lookup over extension methods.

## 5. Methods, parameters, and delegates

### 41. What is method overloading?

Overloading declares multiple methods with the same name but different parameter signatures. The compiler selects an overload based on argument types, conversions, optional parameters, and overload-resolution rules. Return type alone cannot distinguish overloads.

### 42. What is a delegate?

A delegate is a type-safe reference to one or more methods with a compatible signature. Delegates enable callbacks, event handlers, and passing behavior as data. A delegate can be invoked directly or composed with other delegates.

### 43. What are `Action`, `Func`, and `Predicate`?

These are common generic delegate types:

- `Action<T...>` represents a method returning `void`.
- `Func<T..., TResult>` represents a method returning a value.
- `Predicate<T>` represents a method taking `T` and returning `bool`.

They are convenient alternatives to declaring simple custom delegate types. A named delegate can be clearer when it expresses an important domain concept.

### 44. What is a lambda expression?

A lambda is an inline function expression, commonly written `x => x * 2` or `(x, y) => x + y`. It can be converted to a compatible delegate or, in supported contexts, an expression tree. It can capture variables from the enclosing scope; such variables may live longer than the scope in which the lambda was created.

### 45. What is an event?

An event provides a publisher/subscriber pattern with controlled subscription. Subscribers use `+=` and `-=`, while only the declaring type can raise the event in the normal event pattern. Events are commonly backed by delegates, but event accessors can customize subscription.

Long-lived publishers can retain subscriber objects through event handlers; unsubscribe when the subscriber’s lifetime ends if the publisher may outlive it.

### 46. What are optional and named arguments?

An optional parameter has a default value in its declaration and may be omitted by callers. A named argument identifies a parameter by name, improving clarity and allowing arguments to be supplied out of positional order subject to language rules.

Optional parameter defaults are generally bound at the call site, so changing a public default may not affect already-compiled consumers until they are recompiled.

### 47. What are `params` parameters?

A `params` parameter allows a method to accept a variable number of arguments, represented as an array (or supported collection form in newer language versions). It must be the final parameter. The caller may pass individual values or an array/collection compatible with the declaration.

### 48. What is a local function?

A local function is a method declared inside another method. It can improve locality and can use generic parameters and overloads unlike a lambda in some scenarios. It can access variables in the enclosing scope. Unlike lambdas, local functions can be recursive and can be declared after their use.

### 49. What are expression trees?

An expression tree represents code as a data structure, commonly using `Expression<TDelegate>`. It can be inspected, transformed, or translated by a provider, for example translating a query into SQL. Expression trees support a subset of C# constructs and are not interchangeable with arbitrary executable code.

## 6. Generics and collections

### 50. What are generics?

Generics let types and methods operate over type parameters while preserving compile-time type checking. `List<T>` is an example. Generics reduce casts and often avoid boxing for value types. The runtime uses reified generic type information, with implementation details depending on the runtime and type arguments.

### 51. What are generic constraints?

Constraints restrict which type arguments are valid and what operations generic code may perform. Common constraints include `where T : class`, `struct`, `notnull`, `unmanaged`, `new()`, a base class, or an interface. Constraints should express actual requirements rather than overconstrain callers.

### 52. What is covariance and contravariance?

Variance controls assignment compatibility of constructed generic types:

- **Covariance (`out`)** allows a more derived type argument where a less derived one is expected, for output-only positions. For example, `IEnumerable<Dog>` can be used as `IEnumerable<Animal>`.
- **Contravariance (`in`)** allows a less derived type argument where a more derived one is expected, for input-only positions. For example, an `Action<Animal>` can be used where an `Action<Dog>` is expected.

Variance is supported on generic interfaces and delegates when the type parameter’s use satisfies the variance rules; generic classes such as `List<T>` are invariant.

### 53. Compare arrays, `List<T>`, and `IEnumerable<T>`.

- An **array** has a fixed length and direct indexed access.
- A **`List<T>`** is a resizable collection with indexed access and mutation.
- **`IEnumerable<T>`** represents a sequence that can be iterated; it does not promise indexing, mutability, or that enumeration is repeatable.

Expose the narrowest abstraction that satisfies the caller. Avoid assuming that an `IEnumerable<T>` can be enumerated repeatedly or cheaply.

### 54. Compare `IEnumerable<T>`, `ICollection<T>`, and `IList<T>`.

`IEnumerable<T>` supports enumeration. `ICollection<T>` adds operations such as count, add/remove (when supported), and containment. `IList<T>` adds ordered, index-based access and insertion/removal by index. An interface implementation may still reject mutation, so consult its contract and behavior.

### 55. What are `Dictionary<TKey, TValue>` and `HashSet<T>`?

`Dictionary<TKey, TValue>` maps unique keys to values and uses a comparer and hash codes for lookup. `HashSet<T>` stores unique values and supports set operations such as union and intersection. Average lookup is typically constant time, but collisions and resizing can affect performance.

Keys and set elements must not change in ways that alter equality or hash codes while stored, or lookups can behave incorrectly. Provide an appropriate `IEqualityComparer<T>` when default equality is not suitable.

### 56. What is the difference between `IComparable<T>` and `IComparer<T>`?

`IComparable<T>` defines a type’s natural ordering through `CompareTo`. `IComparer<T>` supplies an external comparison strategy, allowing multiple orderings without changing the compared type. Comparers are often passed to sorting and ordered collection APIs.

### 57. What is a concurrent collection?

Concurrent collections, such as `ConcurrentDictionary<TKey, TValue>`, provide thread-safe operations designed for concurrent access. They do not make arbitrary multi-step business logic atomic. Use collection methods that perform the required operation atomically, or coordinate larger operations with synchronization.

## 7. LINQ and iterators

### 58. What is LINQ?

Language Integrated Query (LINQ) provides query operations over data sources through standard operators such as `Where`, `Select`, `OrderBy`, and `GroupBy`. The same syntax can operate on in-memory sequences (`IEnumerable<T>`) or provider-backed queries (`IQueryable<T>`), but those forms have different execution and translation behavior.

### 59. What is deferred execution?

Many sequence operators build a query that runs when enumerated rather than immediately. For example, `Where` and `Select` are commonly deferred. Changes to the source between query creation and enumeration may affect the results, and each enumeration may rerun the work.

Operators such as `ToList`, `ToArray`, `Count`, and many aggregate operations execute immediately. Materialize a sequence when a stable snapshot or repeated access is needed.

### 60. Compare `IEnumerable<T>` and `IQueryable<T>`.

`IEnumerable<T>` represents in-process enumeration using delegates. `IQueryable<T>` represents a query as an expression tree plus a provider, which may translate it to another query language such as SQL. Provider translation supports only a subset of .NET operations and can fail or differ from in-memory semantics.

Be deliberate about where a query switches from provider execution to client execution; materialization such as `ToList` makes the boundary explicit.

### 61. What is the difference between `Select` and `SelectMany`?

`Select` maps each source element to one result. `SelectMany` maps each source element to a sequence and flattens the resulting sequences into one sequence. It is useful for one-to-many relationships and corresponds to a flattening operation.

### 62. What does `yield return` do?

`yield return` produces one element from an iterator method and suspends execution until the next element is requested. The compiler generates a state machine implementing enumeration. This supports lazy sequences without manually writing an enumerator.

The iterator body may not execute until enumeration starts; exceptions and side effects can therefore occur later than the method call that created the sequence.

### 63. What is the difference between `First`, `FirstOrDefault`, `Single`, and `SingleOrDefault`?

- `First` requires at least one match and returns the first; otherwise it throws.
- `FirstOrDefault` returns the first or the type’s default value if none exists.
- `Single` requires exactly one match and throws for zero or multiple matches.
- `SingleOrDefault` allows zero or one match, but throws if there are multiple.

Choose based on the data invariant. For value types, a default result can be ambiguous; nullable types or an explicit check may be preferable.

## 8. Exceptions and resource management

### 64. How does exception handling work?

A `try` block contains code that may fail. Matching `catch` blocks handle exceptions, and `finally` runs during normal or exceptional control flow when execution leaves the `try` construct (subject to process termination and similar limits). Catch specific exception types when recovery is possible.

Exceptions should represent exceptional failure, not normal control flow. Preserve useful diagnostic context and avoid silently swallowing failures.

### 65. How do you rethrow an exception without losing its stack trace?

Inside a `catch`, use `throw;` to rethrow the current exception while preserving its original stack trace. `throw exception;` resets the stack trace origin to the rethrow location. To propagate later or across a boundary, use the appropriate exception-dispatch mechanism rather than storing and throwing it naively.

### 66. What is the difference between `throw` and returning an error value?

Exceptions are useful for exceptional failures that callers are expected to handle at a higher boundary. Return values or result types can model expected outcomes, such as validation failures or “not found,” especially when callers must handle them explicitly. Follow the surrounding API’s established contract and avoid using exceptions as routine branching.

### 67. What is `IDisposable`?

`IDisposable` defines deterministic cleanup through `Dispose()`. It is used for resources that should be released promptly, such as streams, database connections, and locks. It is separate from garbage collection: collecting the wrapper object does not guarantee timely release of an external resource.

### 68. What does a `using` statement or declaration do?

`using` ensures that an `IDisposable` resource is disposed when control leaves its scope, including when an exception occurs. A using declaration disposes at the end of its containing scope:

```csharp
using var stream = File.OpenRead(path);
// use stream
```

For asynchronous disposal, use `await using` with `IAsyncDisposable`.

### 69. What is the dispose pattern?

The dispose pattern defines how a type releases owned resources. A type owning disposable managed objects should dispose them from `Dispose`. A finalizer is generally needed only if the type directly owns unmanaged resources and must provide a fallback. Types should make disposal idempotent where practical and prevent use after disposal when required by the contract.

Use `SafeHandle`-based wrappers for operating-system handles rather than implementing fragile raw-handle finalization where possible.

## 9. Asynchronous and concurrent programming

### 70. What do `async` and `await` do?

`async` enables a method to use `await` and commonly return `Task`, `Task<T>`, or `ValueTask`/`ValueTask<T>`. `await` asynchronously waits for an awaitable operation: if it is incomplete, the method yields control and later resumes; if already complete, continuation may proceed immediately.

`async` does not automatically create a new thread. It is primarily a way to compose asynchronous work without blocking the current thread.

### 71. What is the difference between `Task` and `Thread`?

A `Thread` is an operating-system execution thread with its own scheduling and stack. A `Task` represents an operation or eventual result; it may use a thread-pool thread, perform asynchronous I/O without occupying a thread while waiting, or complete synchronously. Use `Task`-based APIs for most asynchronous application code.

### 72. What is the difference between CPU-bound and I/O-bound work?

CPU-bound work spends time computing and benefits from parallel execution when resources permit. I/O-bound work waits for external operations such as network or file access and should use asynchronous APIs to avoid blocking threads while waiting. Wrapping synchronous I/O in `Task.Run` does not make the underlying I/O truly asynchronous.

### 73. Why should `async void` usually be avoided?

An `async void` method cannot be awaited and its exceptions cannot be observed through a returned task. It is appropriate mainly for event-handler signatures that require `void`. Other asynchronous methods should return `Task` or `Task<T>` so callers can await completion and observe errors.

### 74. What is a `CancellationToken`?

A `CancellationToken` communicates a cooperative cancellation request. The caller requests cancellation through a `CancellationTokenSource`; the operation checks the token or passes it to APIs that support cancellation. Cancellation is not forced termination: code must cooperate and can choose how to respond, commonly by throwing `OperationCanceledException`.

### 75. What is a race condition?

A race condition occurs when the result depends on the timing or interleaving of concurrent operations. Shared mutable state is a common cause. Reduce races by avoiding shared mutable state, using immutable data, atomic operations, locks, or thread-safe abstractions.

### 76. What is a deadlock?

A deadlock is a state in which operations wait indefinitely for resources or progress that cannot occur. With tasks, blocking on asynchronous work using `.Wait()` or `.Result` can deadlock in environments with a synchronization context, and can also cause thread-pool starvation. Prefer `await` end-to-end.

### 77. What is a lock, and what are its limitations?

`lock` provides mutual exclusion around a critical section using a monitor. The lock should be a private, stable object that unrelated code cannot acquire. Keep the protected section short; do not `await` inside a `lock`; and do not hold locks while calling unknown or external code, which can create deadlocks.

For asynchronous coordination, use appropriate async-aware primitives such as `SemaphoreSlim` rather than blocking locks across awaits.

### 78. What is `ConfigureAwait(false)`?

`ConfigureAwait(false)` asks an await not to capture and resume on the current synchronization context (or scheduler context as applicable). It can be useful in reusable library code that does not need a particular context. It is not a universal performance fix, and application code should follow its framework and project conventions.

### 79. What is `Task.WhenAll` versus `Task.WhenAny`?

`Task.WhenAll` completes when all supplied tasks complete and provides their results for generic tasks. `Task.WhenAny` completes when one supplied task completes and returns the completed task; other tasks continue unless explicitly canceled or otherwise handled. Both compose tasks and do not inherently start work—tasks usually begin when created.

### 80. What is the difference between concurrency and parallelism?

Concurrency is structuring multiple operations so their progress can overlap. Parallelism is executing multiple operations at the same time, usually on multiple cores. Asynchronous I/O can be concurrent without parallel CPU execution; parallel loops can execute CPU work simultaneously.

## 10. Memory, value semantics, and performance

### 81. How does garbage collection work in .NET?

The garbage collector reclaims managed objects that are no longer reachable from GC roots, such as active stack references and static references. It uses generations to optimize collection of objects with different lifetimes and may compact the managed heap. Collection timing is nondeterministic; applications should not depend on an object being collected at a specific time.

### 82. What is a memory leak in a garbage-collected program?

A managed memory leak occurs when an object remains reachable even though the application no longer needs it. Common causes include unremoved event handlers, unbounded caches, static collections, and long-lived references to short-lived objects. Garbage collection cannot reclaim reachable objects.

### 83. What is the Large Object Heap?

The Large Object Heap (LOH) is a managed heap area for large allocations, commonly arrays or objects at or above a runtime-defined threshold (historically around 85,000 bytes, subject to runtime details). Large allocations can increase memory pressure and collection costs. Avoid repeated unnecessary large allocations; pool buffers when appropriate and safe.

### 84. What is a span?

`Span<T>` and `ReadOnlySpan<T>` represent contiguous regions of memory without requiring a separate array allocation. They can refer to arrays, stack memory, or other supported memory. `Span<T>` is a `ref struct`, so it is restricted from escaping to heap storage or across async/yield boundaries in ways that would violate its lifetime rules.

Use spans when measured performance or allocation reduction justifies the added complexity.

### 85. What is `stackalloc`?

`stackalloc` allocates a small block of memory on the current stack frame and is commonly used to create a span over temporary storage. It must be used carefully: large stack allocations can cause stack overflow, and the memory is uninitialized unless explicitly initialized. Prefer safe managed allocations unless profiling justifies stack allocation.

### 86. What is the difference between `string` and `StringBuilder`?

`string` is immutable; operations that appear to modify a string create new strings. `StringBuilder` is mutable and is useful when repeatedly assembling a string in a loop or through many edits. For a small fixed number of concatenations, the compiler/runtime may optimize concatenation, so use `StringBuilder` when it improves clarity or measured allocation behavior.

### 87. What is interning?

String interning allows identical string values to share a canonical instance. String literals are commonly interned, but dynamically created strings should not be assumed to be interned. Calling `String.Intern` can keep strings alive for a long time, so it is not a general-purpose memory optimization.

### 88. What is `IDisposable` versus a finalizer from a memory perspective?

`IDisposable` provides deterministic cleanup initiated by code, normally through `using`. A finalizer is nondeterministic and adds GC/finalization overhead. Neither changes the fundamental reachability rule for managed memory. Dispose resources promptly; use finalization only as a carefully designed fallback for direct unmanaged-resource ownership.

### 89. What is a `ref struct`?

A `ref struct` is a stack-constrained value type, such as `Span<T>`. The compiler restricts its use so references to stack-bound memory cannot escape their valid lifetime. Restrictions vary with language version and context, so verify the target language/runtime rules when using newer ref-safety features.

## 11. Modern C# features

### 90. What is a record?

A record is a class or struct form intended for data-centric types. Records provide value-oriented equality and convenient syntax for construction and copying. `record class` is a reference type; `record struct` is a value type. Records can still contain mutable members, so they are not automatically deeply immutable.

```csharp
public record Person(string Name, int Age);
var older = person with { Age = person.Age + 1 };
```

The `with` expression creates a shallow copy with selected members changed.

### 91. What is an init-only property?

An `init` accessor allows a property to be set during object initialization, including in an object initializer or constructor, but not ordinarily reassigned afterward:

```csharp
public string Name { get; init; } = "";
var item = new Item { Name = "Book" };
```

It supports immutable-style APIs but does not guarantee deep immutability of referenced objects.

### 92. What are pattern matching and switch expressions?

Pattern matching tests an input against shapes or conditions and can bind values. Patterns include type, constant, relational, property, positional, and list patterns, with available forms depending on language version. A switch expression maps an input to a result:

```csharp
string Category(object value) => value switch
{
    int n when n < 0 => "negative integer",
    int => "integer",
    string { Length: 0 } => "empty string",
    string => "string",
    null => "null",
    _ => "other"
};
```

Use patterns to make branching explicit; ensure intended cases are covered, including `null` where relevant.

### 93. What is a tuple?

A tuple groups multiple values, commonly using value tuples such as `(int Count, string Name)`. Tuple element names improve readability but are not distinct types by themselves. Tuples are useful for lightweight local results; a named type is usually clearer for a stable public API or complex domain concept.

### 94. What is deconstruction?

Deconstruction assigns components of a tuple or a type with a `Deconstruct` method to separate variables:

```csharp
var (name, age) = person;
```

It can make APIs convenient, but excessive deconstruction may hide the meaning of values.

### 95. What are nullable and null-conditional operators?

The null-coalescing operator `??` supplies a fallback when the left operand is null; `??=` assigns only when the target is null. The null-conditional operators `?.` and `?[...]` stop member/index access when the receiver is null and produce a nullable result (or null for reference results).

```csharp
string displayName = user?.Name ?? "Unknown";
```

These operators do not replace validation when null is invalid.

### 96. What is a `required` member?

A `required` member must be initialized by callers when constructing the object, subject to the language’s initialization rules. It helps communicate required object state at compile time. It is not a runtime validation mechanism and can be bypassed through reflection or other mechanisms.

### 97. What are global using directives and file-scoped namespaces?

A global using directive makes a namespace or alias available throughout the compilation. A file-scoped namespace declares that the remainder of a file belongs to one namespace using a concise syntax. These features reduce repetition but should be used consistently to keep dependencies and file organization clear.

### 98. What are top-level statements?

Top-level statements allow executable statements directly in a program file without explicitly declaring the conventional entry-point class and `Main` method. The compiler generates the entry point. They are useful for small programs and application startup code; larger programs may still benefit from explicit organization.

### 99. What are raw string literals and interpolated strings?

Interpolated strings use `$"..."` to insert expressions. Raw string literals use one or more quote characters as delimiters and preserve content with fewer escape sequences; interpolated raw strings combine `$` prefixes with raw delimiters. They are useful for multiline text, JSON, regular expressions, and embedded code, but interpolation does not automatically make inserted content safe for SQL, HTML, or shell commands.

### 100. What are collection expressions?

Collection expressions provide concise syntax such as `[1, 2, 3]` in supported C# versions. The target type or context determines the resulting collection representation, so they are not simply a universal new array syntax. Confirm the project’s language version and target types before using them in shared code.

## 12. Design, safety, and testing

### 101. What is dependency injection?

Dependency injection (DI) supplies an object’s collaborators from outside rather than having the object construct concrete dependencies internally. Constructor injection makes required dependencies visible and supports substitution in tests. DI is a design technique; a container is optional and should not become a service locator hidden throughout the code.

### 102. What is an immutable object?

An immutable object cannot be observably changed after construction. Immutability simplifies reasoning and concurrency because state does not change unexpectedly. `readonly`, `init`, records, and defensive copying can help, but immutability must include referenced mutable objects if deep immutability is required.

### 103. What are SOLID principles?

- **Single Responsibility:** A module should have one coherent reason to change.
- **Open/Closed:** Extend behavior without repeatedly modifying stable code, often using composition or polymorphism.
- **Liskov Substitution:** Subtypes should honor the behavioral contract expected of their base type.
- **Interface Segregation:** Prefer focused contracts over broad interfaces clients do not need.
- **Dependency Inversion:** High-level policy should depend on abstractions rather than concrete low-level details.

These are design heuristics; applying them mechanically can create unnecessary abstraction.

### 104. What is the difference between unit, integration, and end-to-end tests?

- **Unit tests** verify a small unit of behavior, ideally with few external dependencies.
- **Integration tests** verify collaboration with real or representative infrastructure and components.
- **End-to-end tests** exercise a user-visible workflow through a broad portion of the system.

Use each level to catch different classes of defects. A test should assert observable behavior, not merely mirror implementation details.

### 105. How should equality and hash codes be implemented for a custom type?

Define equality based on the type’s intended identity/value semantics. If overriding `Equals`, also override `GetHashCode`, and implement equality operators only if they improve the type’s API. Ensure `x.Equals(y)` implies `x.GetHashCode() == y.GetHashCode()`. Equality fields should not mutate in ways that invalidate use as dictionary keys or set members.

### 106. How do you prevent SQL injection in C# applications?

Use parameterized queries or a data-access API that binds parameters; do not concatenate untrusted input into SQL. Parameters represent values, not arbitrary SQL identifiers or clauses, so dynamic identifiers must be chosen from a strict allowlist and safely handled for the database. Input validation is useful but does not replace parameterization.

### 107. Why is `async` not a guarantee of better performance?

Async avoids blocking while waiting and can improve scalability for I/O-bound work. It adds state-machine and scheduling/continuation costs and does not make CPU work faster by itself. Use asynchronous APIs for genuinely asynchronous operations and measure performance in the actual workload.

### 108. How do you handle errors at application boundaries?

Catch exceptions where the application can add context, recover, translate to a stable result, or log/report appropriately. Preserve stack traces and useful inner exceptions. Avoid empty catches and avoid exposing implementation details or sensitive data to users. At boundaries, convert failures into the application’s defined error contract.

### 109. What does “program to an interface” mean?

It means consuming behavior through a stable abstraction rather than depending unnecessarily on a concrete implementation. This allows implementations to vary and improves testability. It does not mean every class needs an interface; create abstractions where there is a real variation, boundary, or decoupling benefit.

## 13. Common coding and output questions

### 110. What is printed by this code?

```csharp
int a = 5;
int b = a;
b++;
Console.WriteLine($"{a}, {b}");
```

**Answer:** `5, 6`. `int` is a value type, so `b` receives a copy of `a`.

### 111. What is printed by this code?

```csharp
var first = new List<int> { 1 };
var second = first;
second.Add(2);
Console.WriteLine(first.Count);
```

**Answer:** `2`. The reference is copied, so both variables refer to the same list.

### 112. What is printed by this code?

```csharp
class Counter
{
    public int Value;
}

var counter = new Counter { Value = 1 };
Change(counter);
Console.WriteLine(counter.Value);

static void Change(Counter value) => value.Value = 2;
```

**Answer:** `2`. The reference is passed by value, but the copied reference points to the same object. Reassigning the method’s parameter would not reassign the caller’s variable.

### 113. What is printed when a virtual method is called through a base reference?

```csharp
Animal animal = new Dog();
Console.WriteLine(animal.Speak());
```

Assuming `Dog` overrides the virtual `Speak` method shown earlier, the result is **`bark`**. Virtual dispatch uses the runtime object type, not just the variable’s declared type.

### 114. Does a LINQ query execute when it is declared?

```csharp
var query = numbers.Where(n => n > 10);
```

Usually not for `IEnumerable<T>` query operators such as `Where`; the query is deferred until enumeration. Enumerating it again may run the predicate again. Provider-backed `IQueryable<T>` queries are likewise generally executed when materialized or enumerated, according to the provider.

### 115. What does this method return?

```csharp
static IEnumerable<int> Values()
{
    Console.Write("A");
    yield return 1;
    Console.Write("B");
    yield return 2;
}
```

Calling `Values()` alone prints nothing because the iterator body is deferred. Enumerating it prints `A` before yielding `1`, then `B` before yielding `2`.

### 116. Why does this code risk a closure bug?

```csharp
var actions = new List<Action>();
for (int i = 0; i < 3; i++)
{
    actions.Add(() => Console.WriteLine(i));
}
foreach (var action in actions) action();
```

The lambdas capture the loop variable, not a separate snapshot per iteration. With the shown `for` loop, enumeration after the loop prints `3` three times. Capture a per-iteration local if each action should retain its own value:

```csharp
for (int i = 0; i < 3; i++)
{
    int captured = i;
    actions.Add(() => Console.WriteLine(captured));
}
```

### 117. What is the safer way to concatenate many strings in a loop?

For repeated incremental construction, use `StringBuilder` or collect pieces and call `string.Join`, depending on the shape of the work:

```csharp
var builder = new StringBuilder();
foreach (string part in parts)
{
    builder.Append(part);
}
string result = builder.ToString();
```

For a small fixed number of values, ordinary interpolation or concatenation is usually simpler and may be optimized.

### 118. How would you safely parse an integer from user input?

Use `int.TryParse` when malformed input is an expected possibility:

```csharp
if (int.TryParse(input, out int value))
{
    // Use value.
}
else
{
    // Report or handle invalid input.
}
```

Specify a culture or `NumberStyles` when the input format must be stable across machines. Do not assume user input is valid.

### 119. How would you find duplicates in a sequence?

Use a `HashSet<T>` to track seen values. The set’s comparer determines what counts as equal:

```csharp
var seen = new HashSet<string>(StringComparer.OrdinalIgnoreCase);
var duplicates = new List<string>();

foreach (string item in items)
{
    if (!seen.Add(item))
        duplicates.Add(item);
}
```

This is typically O(n) average time and O(n) additional space, assuming well-behaved hashing.

### 120. How would you make asynchronous work cancellable?

Accept a `CancellationToken`, pass it to supported asynchronous APIs, and observe it during any long-running work:

```csharp
static async Task<string> DownloadAsync(
    HttpClient client,
    Uri address,
    CancellationToken cancellationToken)
{
    using var response = await client.GetAsync(address, cancellationToken);
    response.EnsureSuccessStatusCode();
    return await response.Content.ReadAsStringAsync(cancellationToken);
}
```

Cancellation is cooperative; it communicates a request and does not forcibly stop arbitrary code.

## Final interview preparation tips

- Explain the trade-off and reasoning behind an answer, not only a definition.
- Clarify whether a question concerns the C# language, the .NET runtime, or a particular framework.
- State assumptions about language version, nullability settings, and execution context when they affect the answer.
- Prefer simple, correct examples and discuss edge cases such as null, overflow, repeated enumeration, cancellation, and concurrency.
- For performance questions, describe the likely cost and recommend measuring representative workloads rather than asserting that one construct is always faster.
