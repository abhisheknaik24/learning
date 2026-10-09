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
14. [Advanced questions: types and runtime semantics](#14-advanced-questions-121-170-types-and-runtime-semantics)
15. [Advanced questions: language features](#15-advanced-questions-171-220-language-features)
16. [Advanced questions: design and generics](#16-advanced-questions-221-270-design-and-generics)
17. [Advanced questions: collections and LINQ](#17-advanced-questions-271-320-collections-and-linq)
18. [Advanced questions: async and concurrency](#18-advanced-questions-321-370-async-and-concurrency)
19. [Advanced questions: runtime and performance](#19-advanced-questions-371-420-runtime-and-performance)
20. [Advanced questions: interop and security](#20-advanced-questions-421-460-interop-and-security)
21. [Advanced questions: testing and scenarios](#21-advanced-questions-461-500-testing-and-scenarios)

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

## 14. Advanced Questions (121-170): Types and Runtime Semantics

### 121. What is the difference between `struct` and `class` allocation?

Structs are value types and may be stored inline or boxed; classes are reference types. Actual physical placement is an implementation detail, not a dependable stack-versus-heap rule.

### 122. What is a readonly struct?

A `readonly struct` promises that instance state cannot be mutated through its members, helping express value semantics and avoid defensive copies in some cases.

### 123. What is a ref readonly parameter?

It passes a readonly reference to a value, avoiding a copy while disallowing mutation through the parameter; exact syntax depends on language version.

### 124. What is a `ref` return?

A method can return an alias to existing storage with `ref`, allowing callers to read or modify that storage; lifetime and escape rules are enforced by the compiler.

### 125. What is a `ref local`?

It declares a local alias to a variable returned by reference, rather than copying its value.

### 126. What is a ref safety rule?

It prevents references to shorter-lived storage from escaping into a longer-lived context where they could become invalid.

### 127. What does `scoped` mean?

In supported language versions, `scoped` restricts a ref-like value's escape scope, making lifetime intent explicit.

### 128. What is a `ref struct` restriction?

It cannot be boxed or stored in ordinary heap fields, and historically cannot cross async/yield boundaries; newer language versions relax selected contexts under safety rules.

### 129. What is boxing of a struct implementing an interface?

Converting the struct to an interface reference usually boxes a copy unless constrained generic dispatch or another optimization avoids it.

### 130. What is constrained virtual dispatch?

Generic code with a value-type type parameter can use a `constrained.` call to invoke interface or virtual members without necessarily boxing.

### 131. What is a nullable value type representation?

`Nullable<T>` represents either no value or a value of `T`; its runtime representation is specialized and is not simply a heap wrapper.

### 132. What does lifted operator behavior mean?

An operator on a non-nullable value type can have a nullable form whose result propagates null according to the operator's rules.

### 133. What is a checked context?

It requests overflow checking for certain integral arithmetic and conversions, throwing `OverflowException` on overflow.

### 134. What is an unchecked context?

It permits unchecked integral overflow behavior, typically wrapping, for operations covered by the language rules.

### 135. Does checked arithmetic apply to floating point?

Ordinary floating-point overflow follows IEEE behavior and typically produces infinity rather than `OverflowException`.

### 136. What is definite assignment?

The compiler's analysis ensures a local variable has been assigned on every control-flow path before it is read.

### 137. What is definite null-state analysis?

Nullable analysis tracks whether references may be null across control flow and reports warnings when declared contracts may be violated.

### 138. What does the null-forgiving operator do?

`!` suppresses a nullable warning at that expression but performs no runtime check or conversion.

### 139. What is a nullable annotation context?

It controls whether nullable reference annotations and flow warnings are active for source files or project regions.

### 140. What is `default(T)`?

It yields the default value for a type parameter or type: zero-like values for many value types and null for reference types.

### 141. What is `default` literal inference?

The `default` literal takes its type from the target context, avoiding an explicit generic type argument.

### 142. What is `typeof`?

It obtains a `System.Type` object for a compile-time type, including types that cannot be named through a runtime instance.

### 143. What is `GetType()`?

It returns the runtime type of an object instance and cannot be called on a null reference.

### 144. What is the difference between `is` and `as`?

`is` tests type compatibility and can bind a typed variable; `as` attempts a reference/nullable conversion and returns null on failure.

### 145. What is pattern matching's type pattern?

It checks whether an expression is compatible with a type and can bind the converted value.

### 146. What is a nullable boxing conversion?

Boxing a nullable value with a value boxes the underlying value; boxing an empty nullable produces null.

### 147. How does equality work for floating-point NaN?

IEEE-style comparisons make `NaN == NaN` false; use `double.IsNaN` or `float.IsNaN` to test it.

### 148. What is string interning's equality implication?

Interning may cause identical literals to share references, but string value equality should use string equality rather than depend on reference identity.

### 149. What is a tuple's runtime type?

Value tuples are generic `System.ValueTuple` structs; tuple element names are compile-time metadata and do not create distinct runtime types.

### 150. What is a `dynamic` call site?

The compiler emits runtime binder machinery that resolves dynamic operations based on runtime values and caches binding rules.

### 151. What is `object` equality's default?

`object.Equals` defaults to reference identity unless a type overrides it; value types receive value-based behavior through generated/runtime implementations.

### 152. Why must `GetHashCode` match `Equals`?

Hash-based collections first group by hash; equal values must have equal hashes or lookups can fail.

### 153. Can unequal objects have the same hash code?

Yes. Collisions are allowed; equality comparison distinguishes values within a hash bucket.

### 154. What is a comparer?

An `IComparer<T>` defines ordering; an `IEqualityComparer<T>` defines equality and hashing for collection operations.

### 155. What is culture-sensitive string comparison?

It compares text according to linguistic rules; use ordinal comparison for programmatic identifiers and culture-aware comparison for user-facing text.

### 156. What is Unicode normalization?

It canonicalizes equivalent Unicode sequences into a selected normalization form; visual equivalence does not guarantee identical code-point sequences.

### 157. What is a `char`?

It is a 16-bit UTF-16 code unit, not necessarily a complete Unicode scalar value or user-perceived character.

### 158. What is a `Rune`?

`System.Text.Rune` represents a Unicode scalar value, avoiding assumptions that one UTF-16 code unit equals one character.

### 159. What is the difference between UTF-8 and UTF-16 in .NET?

They encode Unicode using different code-unit sizes; UTF-8 is common for interchange, while .NET strings use UTF-16 code units.

### 160. What is a surrogate pair?

It is two UTF-16 code units representing one Unicode scalar value above the Basic Multilingual Plane.

### 161. What is a `Span<T>` lifetime?

It is restricted to a safe stack-bounded lifetime and cannot outlive the memory it references.

### 162. What is `Memory<T>`?

It is a heap-storable memory region abstraction that can cross async boundaries, unlike `Span<T>`.

### 163. What is `ReadOnlyMemory<T>`?

It exposes a memory region without mutation through that view; it does not guarantee the underlying storage is immutable elsewhere.

### 164. What is `MemoryMarshal`?

It provides low-level operations for reinterpreting or accessing memory representations; incorrect use can violate safety assumptions.

### 165. What is `Unsafe` code?

It permits pointer operations and other low-level behavior under an unsafe context, transferring memory-safety responsibility to the programmer.

### 166. What is a fixed statement?

`fixed` pins a movable managed object for a limited scope so its address can be used by unsafe code.

### 167. What is pinning?

Pinning prevents the GC from relocating an object temporarily; excessive or long-lived pinning can fragment the managed heap.

### 168. What is a `GCHandle`?

It creates a managed handle with specified lifetime behavior, including pinning when required for native interop.

### 169. What is a finalization queue?

It holds finalizable objects for finalizer processing before their memory can be reclaimed.

### 170. What is resurrection?

A finalizer can make an object reachable again, but resurrection complicates lifetime and is generally discouraged.

## 15. Advanced Questions (171-220): Language Features

### 171. What is a primary constructor?

It declares constructor parameters in a type declaration, with availability and capture semantics varying by type kind and C# version.

### 172. What is a static abstract interface member?

It declares a static member contract that implementing types must provide, enabling generic algorithms over type-level operations.

### 173. What is a generic math interface?

It is a .NET interface family for numeric operations that lets constrained generic code perform arithmetic for compatible types.

### 174. What are required members?

They require object creation sites to initialize designated members, with compile-time checks subject to language rules.

### 175. What is the `SetsRequiredMembers` attribute?

It tells the compiler a constructor initializes required members, placing responsibility on the constructor author to honor that claim.

### 176. What is a `with` expression?

It creates a shallow copy with selected properties or fields initialized to new values for records and supported types.

### 177. What is a record's equality contract?

Records generate value-oriented equality from relevant members and type identity rules; customization must preserve symmetry and hashing.

### 178. What is a record's copy constructor?

It supports cloning record instances for `with` expressions, and can be customized for record classes under language rules.

### 179. What is a positional record?

It declares primary constructor parameters that generate properties and deconstruction support.

### 180. What is a `with` expression on a record struct?

It copies the value and applies selected init-settable member changes to the copy.

### 181. What is a collection expression?

It is bracket syntax that constructs a target-compatible collection using collection builder or supported construction rules.

### 182. What is a collection expression spread element?

The `..` element spreads items from another enumerable or collection into a collection expression.

### 183. What is a collection builder?

It allows a type to define construction from a span-like input for collection-expression support.

### 184. What are list patterns?

They match elements of list-like inputs by positions and optionally a slice, where the type supports the required pattern shape.

### 185. What is a slice pattern?

It matches the remaining portion of a sequence in a list pattern, often using `..`.

### 186. What are raw string literals?

They use multiple quote delimiters to represent text with fewer escape sequences and preserve multiline content.

### 187. What is interpolated raw string indentation?

The closing delimiter determines indentation trimming for multiline content; incorrect placement can change literal whitespace.

### 188. What are interpolated string handlers?

They let a target type customize how interpolated strings are constructed, potentially avoiding allocations or skipped formatting work.

### 189. What is a caller information attribute?

It lets optional parameters receive compiler-supplied call-site details such as file, line, member, or expression text.

### 190. What does `CallerArgumentExpression` provide?

It captures source text of an argument expression for diagnostics, useful in guard and assertion APIs.

### 191. What is a file-local type?

It restricts a type's visibility to one source file, reducing naming collisions for generated or implementation-specific helpers.

### 192. What is a global using?

It imports a namespace or alias across all source files in a compilation.

### 193. What is an extern alias?

It distinguishes types from assemblies containing conflicting fully qualified names.

### 194. What is an alias directive?

It assigns a shorter or disambiguating name to a namespace or type.

### 195. What is a top-level statement restriction?

Only one compilation unit may contain top-level statements for an executable entry point, and they must precede namespace/type declarations.

### 196. What is a module initializer?

It is a method marked to execute during module initialization; use sparingly because it introduces implicit startup work.

### 197. What is a global namespace alias?

`global::` starts name resolution at the global namespace to avoid conflicts with local names.

### 198. What is a `nameof` expression?

It produces a compile-time identifier string and remains correct when the referenced symbol is renamed.

### 199. What is a `using` alias?

It gives a namespace or type a local alias, often to resolve ambiguity or simplify long names.

### 200. What is a file-scoped namespace?

It declares one namespace for the remainder of a file using a concise syntax.

### 201. What is a global namespace?

It is the root namespace containing names not nested inside another namespace.

### 202. What is an extern method?

It declares a method implemented outside managed C# code, commonly through platform invocation.

### 203. What is a partial method?

It allows declarations and implementations to be split across partial type parts; modern versions support broader accessibility and return types.

### 204. What is a default interface implementation?

It supplies an interface member body that implementing types may inherit, enabling some contract evolution but requiring careful versioning.

### 205. Can an interface hold instance fields?

No ordinary instance fields; interfaces define contracts, though modern features add static members and implementation capabilities.

### 206. What is an explicit interface implementation?

It implements a member accessible through the interface reference, useful to resolve member-name conflicts or hide specialized APIs.

### 207. What is a covariant return?

An override may return a more derived reference type than the base virtual member, where supported by C# and runtime rules.

### 208. What is an init accessor?

It permits setting a property during initialization while restricting reassignment after initialization completes.

### 209. What is an init-only backing field?

The compiler-generated backing field stores the property value; init-only is an API assignment restriction, not deep immutability.

### 210. What is a required init property?

It combines required initialization with init-only assignment, enforcing caller initialization at compile time.

### 211. What is a static local function?

It cannot capture variables from the enclosing scope, making dependencies explicit and sometimes reducing closure allocations.

### 212. What is a local function's advantage over a lambda?

It supports recursion and overloads and can avoid delegate allocation when invoked directly.

### 213. What is a closure allocation?

Captured variables may require a compiler-generated environment object whose lifetime extends until delegates no longer reference it.

### 214. How can closure allocations be reduced?

Avoid unnecessary captures, use static lambdas/local functions, and measure before optimizing.

### 215. What is a static lambda?

It cannot capture outer variables, making accidental closure dependencies a compile-time error.

### 216. What is an expression-bodied member?

It uses `=>` syntax for a single expression-bodied method, property, or other supported member.

### 217. What is a target-typed conditional expression?

In supported language versions, the target type can help infer the result type of conditional branches.

### 218. What is target-typed `new`?

It omits the type after `new` when the surrounding target type determines the constructed type.

### 219. What is a raw interpolated string's brace count?

The number of `$` prefixes determines how many braces begin an interpolation, allowing literal braces in embedded formats.

### 220. How should language-version features be selected?

Align compiler, target framework, analyzer, and deployment support; use `LangVersion` deliberately and verify generated output.

## 16. Advanced Questions (221-270): Design and Generics

### 221. What is substitutability in type design?

A subtype should honor the behavioral expectations of its base contract, not merely share method signatures.

### 222. What is composition over inheritance?

It builds behavior by containing collaborators, reducing rigid coupling to a base-class hierarchy.

### 223. What is dependency inversion?

High-level policy depends on abstractions, and concrete details implement those abstractions at composition boundaries.

### 224. What is an abstraction?

It exposes essential operations while hiding implementation decisions that callers should not depend on.

### 225. What is an anemic domain model?

It places state in domain objects but leaves behavior in services; whether this is problematic depends on domain complexity and design goals.

### 226. What is a rich domain model?

It places invariant-preserving behavior close to the state it governs, rather than allowing arbitrary external mutation.

### 227. What is a value object?

It is an immutable concept defined by its values rather than identity, such as a currency amount or postal address.

### 228. What is an entity?

It is a domain object with identity that remains meaningful as its state changes.

### 229. What is an aggregate root?

It is the entry point for modifying a consistency boundary of related domain objects.

### 230. What is a domain invariant?

It is a rule that must remain true for a valid domain state and should be enforced at an appropriate boundary.

### 231. What is a factory method?

It constructs an object while encapsulating selection or validation logic beyond a simple constructor.

### 232. What is a builder pattern?

It assembles a complex object in explicit steps, often improving readability for optional configuration.

### 233. What is the strategy pattern?

It encapsulates interchangeable algorithms behind a common contract.

### 234. What is the decorator pattern?

It wraps an object to add behavior while preserving its interface.

### 235. What is the adapter pattern?

It translates one interface into another expected by a client.

### 236. What is the observer pattern?

It notifies subscribers when a subject's state or events change; .NET events are a common form.

### 237. What is the mediator pattern?

It routes interactions through a mediator to reduce direct component dependencies, but can obscure simple flows if overused.

### 238. What is the repository pattern?

It abstracts data access behind collection-like operations; avoid duplicating capabilities already well served by a persistence abstraction.

### 239. What is a unit of work?

It coordinates changes and commits them as one logical persistence operation; many ORMs provide this role.

### 240. What is a specification pattern?

It encapsulates a reusable business predicate or query criterion, with care needed for translation and composition.

### 241. What is a generic type constraint?

It restricts type arguments and communicates available operations to generic implementations.

### 242. What is the `unmanaged` constraint?

It requires a type argument to contain no managed references, enabling certain low-level memory operations.

### 243. What is the `notnull` constraint?

It expresses that a type argument should be non-nullable under nullable analysis.

### 244. What is the `allows ref struct` anti-constraint?

In newer C# versions, it permits a generic parameter to be a ref struct while requiring the generic implementation to obey ref-safety restrictions.

### 245. What is generic variance?

It changes assignment compatibility for generic interfaces/delegates in safe input/output positions.

### 246. Why are generic classes invariant?

Allowing covariance/contravariance for mutable classes could permit unsafe reads or writes through incompatible type arguments.

### 247. What is a covariant interface parameter?

An `out` parameter is produced by the interface and supports safe conversion from a more specific type argument to a broader one.

### 248. What is a contravariant interface parameter?

An `in` parameter is consumed by the interface and supports safe conversion from a broader argument type to a narrower one.

### 249. Can variance apply to value types?

Variance conversions require reference-type conversions; value-type generic instantiations do not gain ordinary variance conversions.

### 250. What is an open generic type?

It is a generic type definition with unspecified type parameters, such as `Dictionary<,>`.

### 251. What is a closed generic type?

It is a generic type constructed with concrete type arguments, such as `Dictionary<string, int>`.

### 252. What is a generic method?

It declares type parameters on a method independently of whether its containing type is generic.

### 253. How does generic type inference work?

The compiler infers type arguments from method arguments and constraints where possible; inference can fail when evidence is ambiguous.

### 254. What is a generic static field?

Each constructed generic type has its own static field storage for each closed type argument combination.

### 255. What is a generic singleton pitfall?

A generic singleton static field creates one instance per closed generic type, not one global instance.

### 256. What is a type parameter's default constraint?

The `default` constraint disambiguates override/interface scenarios involving nullable annotations and generic type parameters in newer C#.

### 257. What is an interface constraint?

It guarantees that a type argument implements specified members, allowing constrained calls in generic code.

### 258. What is a base-type constraint?

It limits type arguments to a particular base class hierarchy.

### 259. Why must `new()` appear after other constraints?

C# specifies constraint ordering, with constructor constraints following primary type constraints.

### 260. What is static polymorphism in C# generics?

Static abstract interface members enable compile-time constrained dispatch for operations such as generic numeric algorithms.

### 261. What is a discriminated union in C#?

C# lacks a built-in general algebraic-union type; developers can model closed alternatives with records, enums, and controlled constructors.

### 262. What is a sealed hierarchy?

It restricts derivation so a set of possible runtime types is controlled, aiding exhaustive reasoning.

### 263. What is an invariant-preserving constructor?

It validates or normalizes inputs so every successfully created instance begins in a valid state.

### 264. What is defensive copying?

It copies mutable input/output data to prevent callers from changing internal state through shared references.

### 265. What is a thread-safe immutable type?

It exposes no mutation after construction and safely publishes fully initialized state, simplifying concurrent reads.

### 266. What is an API compatibility break?

It changes source or binary behavior expected by existing consumers, requiring versioning and migration consideration.

### 267. What is binary compatibility?

It is the ability of already-compiled consumers to run against a changed library without recompilation, subject to runtime rules.

### 268. What is source compatibility?

It means consumer source code still compiles against a changed API, even if recompilation is required.

### 269. What is semantic versioning?

It communicates compatibility expectations through version numbers; actual policy should match the package's documented guarantees.

### 270. Why expose abstractions sparingly?

Every abstraction adds maintenance and indirection; create one when it meaningfully improves changeability, testing, or domain clarity.

## 17. Advanced Questions (271-320): Collections and LINQ

### 271. What is an iterator block?

It is a method using `yield` that the compiler transforms into an enumerator state machine.

### 272. When does an iterator method execute?

Its body generally begins when enumeration starts, not when the enumerable is first returned.

### 273. What happens if an iterator is enumerated twice?

It commonly executes its body twice and may repeat side effects or database/network work.

### 274. What is a lazy sequence?

It computes or retrieves elements as enumeration requests them rather than materializing all values up front.

### 275. What is eager materialization?

It immediately consumes a sequence into a collection such as `List<T>` or an array.

### 276. What is deferred LINQ execution risk?

Repeated enumeration may rerun expensive work, observe changed data, or repeat side effects.

### 277. What is an `IQueryable<T>` provider?

It interprets an expression tree and executes or translates it for a backing data source.

### 278. What is an expression tree translation boundary?

It is the point where a provider can no longer translate an expression and execution must be changed or moved client-side.

### 279. Why avoid returning `IQueryable<T>` across layers?

It leaks provider execution and deferred behavior, making data access, lifetime, and query translation boundaries unclear.

### 280. What does `AsEnumerable` do?

It changes subsequent extension-method binding to LINQ-to-Objects without materializing the sequence itself.

### 281. What does `ToList` do?

It enumerates the sequence immediately and stores results in a list.

### 282. What does `ToLookup` do?

It materializes a one-to-many keyed lookup, allowing multiple values for a key.

### 283. What does `GroupBy` do in LINQ?

It groups elements by a key; execution and memory behavior depend on the source and overload.

### 284. What is `GroupJoin`?

It performs a grouped join, associating each outer element with matching inner elements.

### 285. What is `Join`?

It pairs elements whose selected keys match according to an equality comparer.

### 286. How can a left join be expressed in LINQ?

Use `GroupJoin` followed by `SelectMany` and `DefaultIfEmpty`, or an equivalent query expression.

### 287. What does `Zip` do?

It combines corresponding elements from sequences until the shorter sequence ends.

### 288. What does `Append` do?

It returns a sequence with one value appended; it is usually deferred until enumeration.

### 289. What does `Prepend` do?

It returns a sequence with one value before the source sequence.

### 290. What is `DistinctBy`?

It returns one element per distinct selected key in supported .NET versions.

### 291. What is a stable LINQ sort?

LINQ-to-Objects ordering is stable, preserving source order for equal keys; provider translations may differ.

### 292. What does `ThenBy` do?

It adds a secondary ordering to an existing `OrderBy` sequence.

### 293. What is the difference between `OrderBy` and `ThenBy`?

`OrderBy` establishes a new primary order; `ThenBy` extends the prior ordering as a tie-breaker.

### 294. What does `Aggregate` do?

It folds a sequence into an accumulated result using a supplied function.

### 295. Why prefer named aggregate methods over `Aggregate`?

Methods like `Sum`, `Count`, or `Any` communicate intent more clearly and may be optimized by providers.

### 296. What does `Any` do?

It tests whether a sequence contains an element or any element satisfying a predicate, often short-circuiting.

### 297. What does `All` do?

It tests whether every element satisfies a predicate; for an empty sequence, it returns true.

### 298. What does `SequenceEqual` compare?

It compares two sequences element-by-element using a comparer, including order and length.

### 299. What is LINQ query syntax?

It is SQL-like syntax compiled into method calls such as `Where`, `Select`, and `Join`.

### 300. What is LINQ method syntax?

It chains extension methods and lambdas to compose queries.

### 301. Does query syntax offer additional execution capabilities?

No; it is translated to method calls, though it can express joins and query shapes more readably.

### 302. What is `IAsyncEnumerable<T>`?

It represents a sequence whose elements are asynchronously produced and consumed with `await foreach`.

### 303. What does `await foreach` do?

It asynchronously requests elements from an async enumerator and disposes that enumerator when iteration ends.

### 304. What does `yield return` do in an async iterator?

It asynchronously yields sequence elements from an `async IAsyncEnumerable<T>` method.

### 305. How does cancellation work in async streams?

Pass or annotate a cancellation token and ensure the producer observes it during enumeration.

### 306. What is a collection's equality comparer?

It defines key equality and hashing for dictionaries and sets, such as ordinal-ignore-case string comparison.

### 307. Why can mutable dictionary keys break lookups?

Changing fields used by equality or hashing moves the logical key to a different hash location without updating the collection.

### 308. What is a sorted dictionary?

It stores key/value pairs ordered by a comparer, with logarithmic operations commonly provided by a balanced tree.

### 309. What is `SortedList<TKey,TValue>`?

It stores sorted keys and values in indexed arrays; lookup is efficient but insertion/removal can shift elements.

### 310. What is `PriorityQueue<TElement,TPriority>`?

It dequeues elements by priority using a heap; equal-priority ordering is not generally stable.

### 311. What is an immutable collection?

It returns new collection values for updates while prior instances remain unchanged.

### 312. What is a frozen collection?

It is an immutable collection optimized for lookup-heavy use after a one-time construction cost in supported .NET versions.

### 313. What is `ArrayPool<T>`?

It rents and returns reusable arrays to reduce allocation pressure; returned arrays must not be used afterward.

### 314. What is `MemoryPool<T>`?

It rents disposable memory owners, often used when memory size and lifetime are managed explicitly.

### 315. What is a bounded collection?

It limits capacity or queued work, providing backpressure or memory protection.

### 316. What is a producer-consumer collection?

It coordinates producers and consumers, often with thread-safe queueing and optional bounded capacity.

### 317. What is `Channel<T>`?

It is an asynchronous producer-consumer primitive with configurable bounded/unbounded buffering.

### 318. What is backpressure?

It is controlling production when consumers cannot keep up, preventing unbounded resource growth.

### 319. What is a collection view?

It exposes a filtered, projected, or read-only perspective over data, often without copying all source elements.

### 320. How do you choose a collection type?

Choose based on ordering, mutation, lookup, concurrency, memory, and equality semantics rather than habit.

## 18. Advanced Questions (321-370): Async and Concurrency

### 321. What is a task state machine?

The compiler transforms an async method into a state machine that stores state across await suspension points.

### 322. What is an awaiter?

It implements the protocol used by `await` to determine completion, register continuation, and retrieve a result.

### 323. What makes a type awaitable?

It exposes a compatible `GetAwaiter` pattern whose awaiter provides completion, continuation, and result members.

### 324. What is a custom awaitable?

It implements the awaiter pattern to customize asynchronous completion, though ordinary Tasks are preferable unless there is a clear need.

### 325. What is a synchronization context?

It represents an execution context that can schedule continuations, historically important in UI frameworks.

### 326. What is a task scheduler?

It controls where and how Tasks are queued and executed; most application code uses the default scheduler.

### 327. What is continuation capture?

An await may capture a context for its continuation; `ConfigureAwait(false)` can opt out where appropriate.

### 328. What does `Task.Run` do?

It queues CPU-bound work to the thread pool; it should not wrap naturally asynchronous I/O just to appear asynchronous.

### 329. What is thread-pool starvation?

It occurs when blocked or excessive work exhausts available pool threads, delaying unrelated tasks.

### 330. What is sync-over-async?

It blocks on asynchronous work via `.Result` or `.Wait()`, risking deadlock and thread-pool starvation.

### 331. What is fire-and-forget work?

It starts work without awaiting/observing completion, risking lost exceptions and premature process shutdown.

### 332. How should background work be hosted?

Use a managed hosted service or durable job system with cancellation, logging, retries, and shutdown behavior.

### 333. What is a `ValueTask<T>`?

It can avoid allocations when results often complete synchronously, but has usage constraints and should be introduced after measurement.

### 334. Can a `ValueTask` be awaited multiple times?

Not generally; treat it as a single-consumption awaitable unless its documented source permits reuse.

### 335. What is `TaskCompletionSource<T>`?

It bridges callback-based or externally signaled completion into a Task; configure continuations and completion ownership carefully.

### 336. Why use `RunContinuationsAsynchronously`?

It prevents continuations from running inline on the thread completing a source, reducing reentrancy and lock-related hazards.

### 337. What is cooperative cancellation?

Code observes a token and exits at safe points rather than being forcibly terminated.

### 338. What is linked cancellation?

It creates a token source canceled when any of several source tokens are canceled.

### 339. What is cancellation timeout?

It requests cancellation after a deadline; cancellation still requires the operation to cooperate.

### 340. What is a cancellation race?

Completion and cancellation may happen nearly simultaneously; callers should handle whichever outcome wins according to the API contract.

### 341. What is a semaphore?

It limits concurrent access to a resource using a permit count.

### 342. What is `SemaphoreSlim`?

It is a lightweight semaphore with async wait support for coordinating in-process operations.

### 343. What is a reader-writer lock?

It allows multiple concurrent readers or exclusive writing, useful only when its overhead and fairness match the workload.

### 344. What is `Monitor`?

It provides mutual exclusion and condition signaling; C# `lock` is built on monitor semantics in common implementations.

### 345. What is a volatile field?

It constrains certain reads/writes for visibility and ordering; it does not make compound operations atomic.

### 346. What is `Interlocked`?

It provides atomic operations such as increment, exchange, and compare-exchange for shared values.

### 347. What is compare-exchange?

It atomically updates a value only if it equals an expected value, enabling lock-free algorithms when used carefully.

### 348. What is a memory barrier?

It constrains reordering and visibility of memory operations across threads.

### 349. What is false sharing?

Independent variables on the same cache line can cause cache-coherence contention between processor cores.

### 350. What is lock contention?

It occurs when many threads compete for the same lock and spend time waiting rather than doing work.

### 351. What is a reentrant lock?

It allows the owning thread to acquire the same lock repeatedly, with matching release count requirements.

### 352. What is lock ordering?

It is acquiring multiple locks in a globally consistent order to reduce deadlock cycles.

### 353. What is a concurrent dictionary's `GetOrAdd` caveat?

The value factory may execute more than once under contention even though only one result is stored.

### 354. What is an atomic operation?

It appears indivisible to concurrent observers for the supported operation and memory location.

### 355. What is thread affinity?

It is a requirement that work execute on a particular thread, common for UI controls and some native APIs.

### 356. What is a dedicated thread useful for?

It can serve long-blocking or thread-affine work, but increases resource cost compared with asynchronous I/O.

### 357. What is parallel LINQ?

PLINQ partitions sequence work across threads, helping CPU-bound independent operations when overhead and ordering permit.

### 358. What is the `Parallel` class?

It provides parallel loops and operations for CPU-bound workloads; use cancellation and avoid shared mutable state.

### 359. What is a data race?

It is unsynchronized concurrent access where at least one operation writes, producing nondeterministic behavior.

### 360. What is thread-safe code?

It preserves its invariants under concurrent access through immutability, synchronization, or safe ownership.

### 361. What is a thread-safe singleton?

It initializes exactly once safely under concurrency, using language/runtime-supported initialization patterns.

### 362. What is `Lazy<T>`?

It defers value creation until access and offers configurable thread-safety modes.

### 363. What is a barrier?

It synchronizes multiple participating threads at a phase boundary.

### 364. What is a countdown event?

It signals when a configured number of operations have completed.

### 365. What is a reader-writer channel?

Channels coordinate asynchronous producers and consumers; they do not imply a fixed reader/writer lock model.

### 366. What is an async lock?

It is a coordination abstraction that allows asynchronous waiting without holding a thread-blocking monitor across `await`.

### 367. Why not use `lock` around `await`?

C# forbids awaiting in a `lock` body because monitor ownership is thread-bound and continuation may resume elsewhere.

### 368. What is a starvation problem?

A thread or operation waits indefinitely because others continually receive access to needed resources.

### 369. What is fairness in synchronization?

It describes whether waiting participants receive access in an equitable order; many primitives do not guarantee strict fairness.

### 370. How do you test concurrent code?

Use deterministic coordination primitives and invariant assertions, minimize timing sleeps, and supplement with stress testing.

## 19. Advanced Questions (371-420): Runtime and Performance

### 371. What is tiered compilation?

The JIT can initially compile quickly and later recompile hot methods with stronger optimizations.

### 372. What is tiered PGO?

It uses runtime profiling feedback to guide optimization decisions for frequently executed code.

### 373. What is ReadyToRun?

It precompiles portions of assemblies to native code to improve startup while retaining runtime/JIT capabilities.

### 374. What is Native AOT?

It compiles an application ahead of time to native code and imposes constraints on reflection and dynamic code.

### 375. What is a JIT intrinsic?

It is a method the JIT recognizes and replaces with specialized machine instructions where supported.

### 376. What is inlining?

The JIT substitutes a method body at a call site to reduce call overhead and enable further optimization.

### 377. What is devirtualization?

The JIT proves a virtual call target and turns it into a direct call or inlineable operation.

### 378. What is a managed heap generation?

It categorizes objects by collection age; short-lived objects are collected more frequently than long-lived ones.

### 379. What is a GC root?

It is a reference source, such as a stack/register, static field, or handle, from which reachability is traced.

### 380. What is a pinned object?

It is an object the GC cannot move while pinned, often for native interop; prolonged pinning can fragment memory.

### 381. What is a finalizable object cost?

It requires finalization processing and usually survives an additional collection before memory is reclaimed.

### 382. What is a `SafeHandle`?

It is a robust wrapper for native handles that provides reliable release and integrates with finalization safely.

### 383. What is a memory leak in managed code?

It is unnecessary object retention through reachable references, event subscriptions, caches, or static state.

### 384. What is a LOH compaction?

It compacts large-object heap segments under supported runtime policies, with potentially significant pause costs.

### 385. What is server GC?

It configures garbage collection for throughput-oriented server workloads, commonly using multiple heaps.

### 386. What is workstation GC?

It configures GC behavior often suited to interactive applications, balancing latency and resource use.

### 387. What is background GC?

It performs some collection work concurrently with application threads, reducing pauses with trade-offs.

### 388. What is a GC pause?

It is time during which managed threads are suspended for collection phases; latency depends on workload and runtime.

### 389. What is allocation pressure?

It is the rate and volume of allocations, which can trigger more frequent collections and increase CPU use.

### 390. What is an allocation-free hot path?

It minimizes transient allocations in frequently executed code, but should be optimized only after profiling.

### 391. What is object pooling?

It reuses objects to reduce allocations; pooling can retain memory and introduce reset/thread-safety bugs.

### 392. What is `ArrayPool<T>.Shared`?

It is a shared array pool for renting reusable arrays, with no guarantee of zeroed contents unless requested.

### 393. What is stack allocation's risk?

Large stack allocations can exhaust thread stacks; stack capacity is limited and often smaller than heap capacity.

### 394. What is UTF-8 string support in modern .NET?

Modern APIs include UTF-8-oriented representations and overloads, reducing conversions for byte-oriented protocols where appropriate.

### 395. What is `GC.GetAllocatedBytesForCurrentThread`?

It reports managed allocation bytes for the current thread, useful for focused measurement but not a complete memory profile.

### 396. What is BenchmarkDotNet?

It is a benchmarking library that manages warmup, repeated measurements, and environment reporting for .NET microbenchmarks.

### 397. Why are microbenchmarks misleading?

They may omit real workload, runtime warmup, I/O, contention, and deployment effects.

### 398. What is profiling?

It samples or traces execution to identify CPU, allocation, contention, or I/O bottlenecks in representative runs.

### 399. What are EventCounters?

They expose runtime or application metrics through .NET diagnostics infrastructure.

### 400. What is EventPipe?

It is a cross-platform diagnostics transport for collecting runtime events, traces, and counters.

### 401. What is `dotnet-trace`?

It collects EventPipe traces from .NET processes for runtime and application diagnostics.

### 402. What is `dotnet-counters`?

It monitors live runtime and application counters exposed through diagnostics.

### 403. What is `dotnet-dump`?

It captures or analyzes process dumps for managed diagnostics, subject to platform and runtime support.

### 404. What is a core dump?

It is a process memory snapshot used to investigate crashes, hangs, or state, and may contain sensitive data.

### 405. What is tiered PGO's deployment concern?

Its effectiveness depends on warmup and workload representativeness; cold-start and short-lived processes may benefit differently.

### 406. What is an allocation profile?

It attributes memory allocations to call paths, helping find high-volume transient objects.

### 407. What is a stack trace?

It records the call path associated with an exception or sampled execution point.

### 408. What is a minidump?

It is a reduced process dump containing selected state; its usefulness depends on included memory and symbols.

### 409. What is a symbol file?

It maps compiled instructions to source-level names and lines, essential for readable diagnostics.

### 410. What is a PDB?

It is a program database file containing debug symbols and metadata for source-level debugging.

### 411. What is trimming's reflection risk?

The trimmer may remove members accessed only via reflection unless dependencies are annotated or preserved.

### 412. What is dynamic code generation?

It emits executable code at runtime, which may be unsupported or restricted under Native AOT and trimming.

### 413. What is a source generator?

It runs during compilation to generate source code from syntax or metadata without runtime reflection.

### 414. What is an analyzer?

It statically inspects code and can report diagnostics or offer fixes during development/build.

### 415. What is a Roslyn code fix?

It is an IDE action associated with an analyzer diagnostic that transforms source code.

### 416. What is incremental source generation?

It uses an incremental pipeline to cache generator results and minimize repeated compilation work.

### 417. What is a runtime identifier?

It identifies a target platform/architecture for runtime-specific restore or publishing, such as a platform-specific RID.

### 418. What is crossgen?

It refers to ahead-of-time compilation tooling for .NET assemblies, with modern workflows exposed through ReadyToRun publishing.

### 419. What is a native dependency?

It is an unmanaged library required by an application and must be packaged/resolved for each target platform.

### 420. How do you tune GC?

Measure allocation, latency, and throughput first; select runtime GC settings based on workload rather than guessing.

## 20. Advanced Questions (421-460): Interop and Security

### 421. What is P/Invoke?

Platform Invocation Services call exported functions in native libraries from managed code.

### 422. What is marshalling?

It converts data representations between managed and unmanaged code across an interop boundary.

### 423. What is a calling convention?

It defines how arguments and return values cross a native function boundary, including stack cleanup and register usage.

### 424. What is `DllImport`?

It declares a native function import; newer interop source-generation patterns may improve AOT and performance support.

### 425. What is `LibraryImport`?

It is a source-generated P/Invoke mechanism available in modern .NET, reducing runtime marshalling work in supported cases.

### 426. What is COM interop?

It bridges .NET code with Component Object Model components, primarily relevant to Windows interoperability.

### 427. What is COM callable wrapper?

It exposes a managed object to COM clients through a runtime-generated interop wrapper.

### 428. What is runtime callable wrapper?

It represents a COM object to managed code and handles interop services.

### 429. What is `SafeHandle` interop benefit?

It provides reliable native-handle ownership and cleanup across exceptions and finalization.

### 430. What is `MarshalAs`?

It specifies how a managed value is represented when marshalled to native code.

### 431. What is `StructLayout`?

It controls field layout for interop or binary compatibility scenarios.

### 432. What is blittable data?

It has compatible managed and unmanaged representations and can often cross interop without field-by-field conversion.

### 433. What is unsafe pointer arithmetic risk?

Incorrect bounds or lifetime assumptions can corrupt memory or crash the process.

### 434. What is a cryptographic random number generator?

It produces unpredictable values for security uses; do not use `System.Random` for keys, tokens, or secrets.

### 435. How should passwords be stored?

Use a vetted password-hashing algorithm with salts and appropriate work factors, not reversible encryption or plain hashes.

### 436. What is constant-time comparison?

It compares secrets without timing differences proportional to matching prefixes, reducing timing side channels.

### 437. What is secret zeroization?

It attempts to overwrite sensitive memory, though managed copies and runtime behavior make guarantees limited.

### 438. What is TLS certificate validation?

It verifies server identity and trust chain; disabling validation exposes communication to interception.

### 439. What is secure serialization?

It validates inputs, restricts type activation, and avoids unsafe polymorphic deserialization of untrusted payloads.

### 440. Why is `BinaryFormatter` unsafe?

It can instantiate attacker-controlled object graphs and is obsolete/disabled in modern .NET; use safer formats.

### 441. What is JSON polymorphic deserialization risk?

Unrestricted type selection can instantiate unintended types; use explicit discriminators and allowlisted models.

### 442. What is path traversal?

It uses malicious path segments to escape an intended directory; canonicalize and enforce an allowed root.

### 443. What is command injection?

It changes an invoked process's command semantics using untrusted input; prefer argument APIs and validate permitted values.

### 444. What is SQL injection prevention in .NET?

Use database parameters and never concatenate untrusted values into SQL command text.

### 445. What is an authorization policy?

It expresses access requirements evaluated by an application or framework authorization system.

### 446. What is the principle of least privilege?

Grant only the permissions necessary to perform a specific task.

### 447. What is a secret store?

It securely manages credentials and keys outside source code and ordinary configuration.

### 448. What is security patch management?

It tracks and updates vulnerable runtimes and dependencies through a tested deployment process.

### 449. What is an attack surface?

It is the set of interfaces and behaviors an attacker could reach, including network endpoints, parsers, and native interop.

### 450. What is defense in depth?

It layers independent security controls so one failure does not expose the entire system.

### 451. What is an untrusted deserialization boundary?

It is any point where external bytes become objects; validate schema and avoid unsafe type activation.

### 452. What is cryptographic agility?

It is the ability to update algorithms and key sizes as security guidance changes without redesigning the whole system.

### 453. What is key rotation?

It replaces cryptographic keys according to a controlled lifecycle while preserving necessary decryption of prior data.

### 454. What is a secure random token?

It is generated with a cryptographic RNG, sufficient entropy, limited lifetime, and protected storage/transport.

### 455. What is a replay attack?

It resubmits a previously valid message; nonces, timestamps, expiry, and idempotency controls can reduce risk.

### 456. What is input validation?

It checks that untrusted values meet type, range, format, and business constraints before use.

### 457. What is output encoding?

It encodes data for the output context, preventing interpretation as executable markup or commands.

### 458. What is a dependency vulnerability scan?

It identifies known vulnerable package versions; findings require prioritization and verified remediation.

### 459. What is a threat model?

It identifies assets, trust boundaries, attackers, threats, and mitigations before or during system design.

### 460. How should a .NET application handle a security exception?

Fail safely, log useful non-sensitive diagnostics, return an appropriate response, and avoid exposing secrets or stack traces.

## 21. Advanced Questions (461-500): Testing and Scenarios

### 461. What is a test double?

A substitute dependency used to control or observe interactions in a test, including stubs, fakes, spies, and mocks.

### 462. What is a fake?

A lightweight working implementation used in tests, such as an in-memory repository.

### 463. What is a stub?

It returns predetermined values to support a test's setup.

### 464. What is a mock?

It verifies expected interactions, though behavior-focused assertions are often more resilient.

### 465. What is a contract test?

It verifies that independently developed components agree on an interface or message contract.

### 466. What is property-based testing?

It generates many inputs and checks general properties rather than only hand-picked examples.

### 467. What is fuzz testing?

It supplies unexpected or malformed inputs to find crashes, hangs, or security defects.

### 468. What is mutation testing?

It changes code deliberately to see whether tests detect the altered behavior.

### 469. What is a flaky test?

A test whose outcome varies across runs due to timing, shared state, environment, or nondeterminism.

### 470. How can test time be controlled?

Inject a clock abstraction or use a testable time provider rather than reading global system time everywhere.

### 471. How do you test asynchronous methods?

Await the returned task and assert outcomes; never use `async void` for ordinary testable operations.

### 472. How do you test cancellation?

Cancel a token source, await the operation, and verify the documented cancellation outcome and cleanup.

### 473. How do you test a race condition?

Use barriers or task completion sources to force specific interleavings instead of relying on sleeps.

### 474. What is integration testing?

It verifies collaboration among real components such as database, HTTP host, serialization, and configuration.

### 475. What is an in-memory database testing limitation?

It may not reproduce relational constraints, transactions, SQL translation, or provider behavior.

### 476. What is a test container?

A disposable containerized dependency, such as a real database, used to improve integration-test fidelity.

### 477. What is a golden-file test?

It compares output with a checked-in expected artifact; updates require review to avoid blindly accepting regressions.

### 478. What is code coverage?

It measures which code executed during tests, not whether assertions are sufficient or behavior is correct.

### 479. What is a regression test?

It captures a previously observed defect so future changes detect its recurrence.

### 480. What is a smoke test?

It quickly verifies critical application startup and basic operations after deployment.

### 481. How do you debug a production-only issue?

Collect safe correlated logs, metrics, traces, runtime/version/config details, and a minimal reproduction without exposing sensitive data.

### 482. What is structured exception logging?

It records exception details and contextual properties in queryable fields while avoiding secret leakage.

### 483. What is distributed tracing?

It links operations across services using trace and span context, helping diagnose latency and failures.

### 484. What is a correlation identifier?

It associates related logs and requests across system boundaries.

### 485. What is a health check?

It reports application liveness/readiness; dependency checks should not cause inappropriate restart loops.

### 486. What is readiness versus liveness?

Readiness indicates whether traffic should be sent; liveness indicates whether the process should be restarted.

### 487. What is graceful degradation?

It preserves core functionality when optional dependencies or features fail.

### 488. What is a circuit breaker?

It temporarily blocks calls to a failing dependency to prevent cascading resource exhaustion.

### 489. What is a retry storm?

Many clients retry simultaneously and intensify an outage; use bounded retries, backoff, jitter, and budgets.

### 490. What is exponential backoff?

It increases retry delays exponentially, often with jitter to reduce synchronized retries.

### 491. What is idempotency in an API?

Repeating a request produces the same intended result, making safe retries possible.

### 492. What is a graceful deployment?

It drains traffic, handles in-flight work, starts compatible versions, and supports rollback.

### 493. What is configuration validation?

It checks required settings and ranges at startup so misconfiguration fails clearly before serving traffic.

### 494. What is a feature toggle risk?

Unmanaged flags accumulate complexity and may expose functionality without server-side authorization.

### 495. What is a backward-compatible API change?

It preserves existing consumer behavior, such as adding optional fields without changing established semantics.

### 496. How should an API evolve?

Add compatible capabilities, monitor usage, deprecate deliberately, and version breaking changes with migration guidance.

### 497. What is a memory dump privacy risk?

Dumps can contain credentials, personal data, and request content; restrict access and retention.

### 498. What is a deployment rollback consideration?

Database changes and external side effects may not be reversible, so use backward-compatible migration phases.

### 499. How do you choose between sync and async APIs?

Use async for naturally asynchronous I/O and composable nonblocking paths; keep CPU work synchronous unless parallel execution is justified.

### 500. How do you answer an advanced C# design question?

Clarify constraints, explain trade-offs, state runtime/version assumptions, and demonstrate correctness with a focused example or test.

## Final interview preparation tips

- Explain the trade-off and reasoning behind an answer, not only a definition.
- Clarify whether a question concerns the C# language, the .NET runtime, or a particular framework.
- State assumptions about language version, nullability settings, and execution context when they affect the answer.
- Prefer simple, correct examples and discuss edge cases such as null, overflow, repeated enumeration, cancellation, and concurrency.
- For performance questions, describe the likely cost and recommend measuring representative workloads rather than asserting that one construct is always faster.
