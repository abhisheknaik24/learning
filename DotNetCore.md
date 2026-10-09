# .NET Core / .NET Interview Questions and Answers

> “.NET Core” refers to the cross-platform .NET Core line through .NET Core 3.1. From .NET 5 onward the platform is named .NET. This guide covers the shared modern platform and calls out version differences where useful.

## Contents

1. [Platform and project structure](#1-platform-and-project-structure)
2. [Runtime and application configuration](#2-runtime-and-application-configuration)
3. [Dependency injection and hosting](#3-dependency-injection-and-hosting)
4. [Web APIs and middleware](#4-web-apis-and-middleware)
5. [Configuration, logging, and security](#5-configuration-logging-and-security)
6. [Performance, deployment, and testing](#6-performance-deployment-and-testing)
7. [Runtime and execution](#7-runtime-and-execution)
8. [SDK, CLI, and MSBuild](#8-sdk-cli-and-msbuild)
9. [Assemblies, loading, and reflection](#9-assemblies-loading-and-reflection)
10. [Memory management, JIT, and AOT](#10-memory-management-jit-and-aot)
11. [ASP.NET Core servers and pipeline](#11-aspnet-core-servers-and-pipeline)
12. [Hosting, DI, configuration, and logging](#12-hosting-di-configuration-and-logging)
13. [HTTP APIs, data, and serialization](#13-http-apis-data-and-serialization)
14. [Application and platform security](#14-application-and-platform-security)
15. [Observability, diagnostics, and performance](#15-observability-diagnostics-and-performance)
16. [Deployment, containers, and cloud](#16-deployment-containers-and-cloud)
17. [Testing and resilience](#17-testing-and-resilience)
18. [Globalization and networking](#18-globalization-and-networking)
19. [Advanced platform scenarios](#19-advanced-platform-scenarios)

## 1. Platform and project structure

### 1. What is .NET Core?

.NET Core was Microsoft's cross-platform, open-source implementation of .NET, designed for Windows, Linux, and macOS. .NET Core 1.0 through 3.1 evolved into the unified .NET platform beginning with .NET 5. Modern .NET continues the cross-platform runtime, SDK, libraries, and application frameworks.

### 2. What is the difference between .NET Framework, .NET Core, and modern .NET?

.NET Framework is the original Windows-focused implementation, commonly used by older applications. .NET Core was a modular, cross-platform redesign. Beginning with .NET 5, Microsoft unified the product name as .NET and continued the .NET Core lineage. Compatibility depends on APIs and frameworks used; migrating a .NET Framework application can require code and dependency changes.

### 3. What are the main components of .NET?

The platform includes a runtime to execute applications, base class libraries, a compiler toolchain, SDK and CLI tools, and application frameworks such as ASP.NET Core. The SDK includes the runtime needed for development/building; a deployment may use an installed shared runtime or bundle a runtime with the application.

### 4. What is the difference between the SDK and runtime?

The **SDK** includes tools to create, build, test, and publish applications, and includes runtime components. The **runtime** is sufficient to execute compatible applications but does not include the full development toolchain. Installing an SDK is normally the developer setup; production servers may install a runtime or use a self-contained deployment.

### 5. What is the `dotnet` CLI?

The `dotnet` CLI is the cross-platform command-line interface for .NET operations. Common commands include `dotnet new`, `dotnet restore`, `dotnet build`, `dotnet test`, `dotnet run`, and `dotnet publish`. The exact available options depend on SDK version and project type.

### 6. What is a `.csproj` file?

A `.csproj` is an MSBuild project file describing target frameworks, package references, compiler options, and build items. Modern SDK-style projects infer many source files and resources by convention:

```xml
<PropertyGroup>
  <TargetFramework>net8.0</TargetFramework>
  <Nullable>enable</Nullable>
</PropertyGroup>
```

Use a target framework supported by the installed SDK and deployment environment.

### 7. What is a target framework moniker (TFM)?

A TFM identifies the API/runtime target for a project, for example `net8.0` or `netstandard2.0`. A project may target multiple frameworks with `TargetFrameworks`. A library targeting a broad framework can be consumed by more platforms but may have fewer APIs available.

### 8. What is .NET Standard?

.NET Standard is an API specification intended to help libraries share a common API surface across multiple .NET implementations. For new applications, target a supported modern .NET TFM. For libraries needing compatibility with older .NET implementations, choose a target based on the actual consumer matrix; .NET Standard is not a runtime.

### 9. What is NuGet?

NuGet is the .NET package-management ecosystem. Projects declare dependencies in project files or central package management files; restore resolves packages from configured feeds and creates assets for build. Pin and update dependencies deliberately, review transitive dependencies, and use trusted feeds.

### 10. What is the difference between a solution and a project?

A project describes one buildable component, such as a library or web application. A solution (`.sln` or newer solution format) groups projects for development and coordinated builds. A solution does not itself define the runtime composition of the application.

## 2. Runtime and application configuration

### 11. What is Kestrel?

Kestrel is ASP.NET Core's cross-platform web server. It can serve requests directly or run behind a reverse proxy such as IIS, Nginx, or a cloud load balancer. Reverse proxies commonly handle TLS termination, forwarding, or edge concerns; deployment topology determines the configuration required.

### 12. What is the Generic Host?

The Generic Host provides a common application lifetime, dependency injection container, configuration, logging, and hosted-service infrastructure. ASP.NET Core's web host builds on these hosting concepts. The host manages startup and graceful shutdown and is configured before running the application.

### 13. What is the request pipeline?

The request pipeline is an ordered sequence of middleware components. Each component can inspect or change a request/response, call the next component, or short-circuit. Ordering is significant: for example, routing must precede endpoint execution, and authentication must run before authorization.

### 14. What are environments in ASP.NET Core?

An environment name, commonly `Development`, `Staging`, or `Production`, selects environment-specific configuration and behavior. `IHostEnvironment` exposes the name. Environment names are not security boundaries: production secrets must not be placed in a development-only file that is accidentally deployed.

### 15. How does .NET configuration work?

Configuration providers expose key/value settings through `IConfiguration`. Providers can include JSON files, environment variables, command-line arguments, user secrets for development, and custom sources. Later providers commonly override earlier values according to registration order. Bind related values to options classes and validate required settings at startup.

### 16. What is the Options pattern?

The Options pattern binds configuration sections to typed classes and injects them as `IOptions<T>`, `IOptionsSnapshot<T>`, or `IOptionsMonitor<T>`. `IOptions<T>` is generally a stable value; snapshot supports scoped refresh behavior in supported hosts; monitor can observe changes and is suitable for singleton consumers. Validate options to fail early on invalid configuration.

### 17. What is a hosted service?

A hosted service performs work tied to the host lifetime. Implement `IHostedService` for start/stop behavior or derive from `BackgroundService` for a long-running asynchronous loop. Honor the stopping cancellation token, avoid blocking startup indefinitely, and create scopes when using scoped dependencies in a singleton hosted service.

### 18. What are assemblies and load contexts?

An assembly is a compiled unit containing types and metadata. Assembly loading resolves dependencies into an application. `AssemblyLoadContext` provides isolation and unloading capabilities for some plugin scenarios. Unloading requires removing all references to loaded objects; it is not equivalent to unloading a single type while references remain.

## 3. Dependency injection and hosting

### 19. What is dependency injection in .NET?

Dependency injection (DI) supplies a class's dependencies from outside, usually through constructors. ASP.NET Core includes a built-in service container. DI reduces hidden coupling, enables substitution in tests, and centralizes object lifetime management. Register the abstraction and implementation at application composition boundaries.

### 20. What are the service lifetimes?

- **Transient:** A new instance is created each time it is requested.
- **Scoped:** One instance is used within a scope; web applications commonly create a scope per request.
- **Singleton:** One instance is used for the container's lifetime.

Choose lifetime based on state and thread-safety. A singleton must not directly capture a shorter-lived scoped dependency.

### 21. What is a captive dependency?

A captive dependency occurs when a longer-lived service retains a shorter-lived service, such as a singleton retaining a scoped database context. The shorter-lived object then behaves as though it has the longer lifetime, risking stale state and concurrency problems. Inject a scope factory and create a scope for each unit of work when appropriate.

### 22. What is constructor injection?

Constructor injection declares required dependencies as constructor parameters and stores them for use. It makes the class's dependencies explicit and generally allows tests to provide substitutes. Avoid constructors with excessive dependencies; they may indicate the class has too many responsibilities.

### 23. Does the built-in container support every DI feature?

The built-in container supports common registrations, constructor injection, scopes, open generics, and disposal. It is intentionally simple and does not provide every feature of third-party containers. Add a different container only for a concrete requirement, and preserve correct scope and disposal behavior.

### 24. How should scoped services be used from a singleton?

Do not inject a scoped service directly into a singleton. Inject `IServiceScopeFactory`, create and dispose a scope for each operation, then resolve the scoped dependency within it. In hosted work, this prevents one request-like scoped object from being reused indefinitely.

## 4. Web APIs and middleware

### 25. What is middleware?

Middleware is a component in the HTTP pipeline, commonly registered with `Use...` methods. It can perform cross-cutting tasks such as exception handling, HTTPS redirection, authentication, or request logging. Middleware is usually constructed once, so do not store per-request mutable state in its instance.

### 26. Why does middleware order matter?

Middleware executes in registration order on the way in and typically in reverse order on the way out. An exception handler should surround components whose exceptions it must catch. Routing selects endpoints; authentication establishes identity; authorization enforces access; endpoint execution runs the selected handler. Verify order against the framework version and chosen middleware.

### 27. What is routing?

Routing matches an incoming HTTP request to an endpoint based on route templates, HTTP method, constraints, and other metadata. Attribute routing associates routes with controllers/actions; minimal APIs register endpoint handlers in code. Route values are not a substitute for validating all untrusted input.

### 28. What is the difference between controllers and minimal APIs?

Controllers provide an established MVC structure with attributes, filters, model binding, and conventions. Minimal APIs register endpoint handlers directly with less ceremony and can use endpoint filters and route groups. They both run on ASP.NET Core; choose based on application complexity, team conventions, and maintainability rather than assuming one is always faster.

### 29. What is model binding?

Model binding maps request data—such as route, query, header, and body values—to action parameters or models. Validation can report invalid input through model state or endpoint-specific behavior. Treat bound values as untrusted; enforce domain invariants and authorization independently.

### 30. What is the difference between authentication and authorization?

Authentication determines who the caller is; authorization determines whether that caller may perform an operation. ASP.NET Core authentication middleware establishes a `ClaimsPrincipal`; authorization policies, roles, or requirements evaluate access. An authenticated user is not automatically authorized for every resource.

### 31. What is a filter?

MVC filters run at specific stages around controller/action execution and can implement concerns such as authorization, action behavior, exception handling, or result handling. Middleware applies more broadly to the HTTP pipeline. Minimal API endpoint filters operate around endpoint handlers. Prefer the mechanism with the right scope and lifecycle.

### 32. How should an API handle errors?

Return appropriate HTTP status codes and a consistent, non-sensitive error body, commonly using Problem Details where configured. Log diagnostic information on the server, correlate requests, and avoid exposing stack traces or secrets in production responses. Central exception-handling middleware can standardize unexpected failures.

## 5. Configuration, logging, and security

### 33. How does logging work in .NET?

Applications use `ILogger<T>` and configured providers to emit structured log events. Prefer message templates with named values rather than interpolated strings, and select levels meaningfully (`Trace` through `Critical`). Avoid logging credentials, tokens, personal data, or untrusted text without appropriate handling.

### 34. How should secrets be stored?

Do not commit production secrets to source control or ordinary configuration files. Use a secret manager or platform identity/vault in production; user secrets can help with local development but are not encrypted production storage. Apply least privilege, rotate credentials, and avoid exposing secret values in logs.

### 35. What is CORS?

Cross-Origin Resource Sharing (CORS) is a browser-enforced mechanism that controls whether a web page from one origin may access a server response from another. Configure explicit trusted origins and required methods/headers. CORS is not authentication and does not protect an API from non-browser clients.

### 36. What is HTTPS redirection and HSTS?

HTTPS redirection sends eligible HTTP requests to HTTPS. HSTS tells browsers to use HTTPS for future requests to a domain. Enable these carefully according to deployment topology; when TLS terminates at a proxy, forwarded headers must be configured from trusted proxies so the application correctly understands the original scheme.

### 37. How do you protect an API?

Use TLS, authenticate callers, enforce authorization at resource/action boundaries, validate input, apply rate/abuse controls as appropriate, secure secrets, and keep dependencies patched. Use parameterized database access. Do not trust client-side validation, CORS, route obscurity, or a valid token alone as complete protection.

### 38. What is data protection in ASP.NET Core?

The Data Protection system provides APIs for protecting and unprotecting data, commonly for authentication cookies and antiforgery tokens. In a multi-instance deployment, instances that must read each other's protected data need an appropriately shared key ring and compatible application identity. Protect keys at rest and manage rotation and access.

## 6. Performance, deployment, and testing

### 39. What is a self-contained deployment?

A self-contained deployment includes the .NET runtime needed by the application for a target runtime identifier. A framework-dependent deployment relies on a compatible runtime installed on the target. Self-contained publishing can simplify runtime prerequisites but increases deployment size and requires republishing to receive runtime updates.

### 40. What is trimming?

Trimming removes code believed to be unused during publish to reduce output size. Reflection, dynamic activation, serialization, and plugin patterns can hide dependencies from the trimmer. Treat trim warnings seriously and test the published artifact, not just a development build.

### 41. What is Native AOT?

Native AOT compiles an application ahead of time to native code for a target platform, potentially improving startup time and reducing runtime requirements. It imposes compatibility constraints, especially around reflection and dynamic code generation. Verify library support and measure the real deployment before adopting it.

### 42. How can you improve ASP.NET Core performance?

Measure first with representative load and profiling. Common techniques include asynchronous I/O, reducing allocations and serialization work, efficient database queries, response compression/caching where appropriate, limiting payloads, and avoiding blocking calls. Performance depends on bottlenecks and deployment configuration; micro-optimizations without measurement can reduce maintainability.

### 43. What is health checking?

Health checks expose whether an application or its dependencies are ready or live. A liveness check should indicate whether the process can continue; a readiness check indicates whether it should receive traffic. Avoid making a liveness endpoint fail merely because a transient external dependency is unavailable, which can cause restart loops.

### 44. How do you test a .NET application?

Use unit tests for isolated logic, integration tests for component interactions and infrastructure boundaries, and end-to-end tests for critical workflows. ASP.NET Core's test host and `WebApplicationFactory` can host an application in-process for integration tests. Keep test configuration isolated and do not use production secrets or data.

### 45. What is structured logging, and why is it useful?

Structured logging records named properties alongside an event message, allowing filtering and aggregation by fields such as request ID, operation, or status. For example, `logger.LogInformation("Order {OrderId} created", orderId)` preserves `OrderId` as data. Ensure values are safe to retain and do not place sensitive information in event properties.

### 46. What is graceful shutdown?

Graceful shutdown gives the host and hosted services an opportunity to stop accepting work, finish or cancel in-flight operations, and release resources. Observe cancellation tokens, set realistic shutdown timeouts, and make operations idempotent where retries or interruption are possible.

## 7. Runtime and execution

### 47. What does the .NET runtime do when an application starts?

The runtime initializes the process, loads the entry assembly and dependencies, creates the managed execution environment, and invokes the application's entry point.

### 48. What is the Common Language Runtime (CLR)?

The CLR is the runtime implementation for .NET that provides services such as managed execution, garbage collection, exception handling, and runtime type information. Modern .NET uses CoreCLR for most JIT-based deployments.

### 49. What is the Common Intermediate Language (CIL)?

CIL is the processor-independent instruction set emitted by managed compilers. At execution, the runtime or an ahead-of-time compiler turns it into code appropriate for the target.

### 50. What is metadata in a .NET assembly?

Metadata describes assemblies, references, types, members, and attributes in a machine-readable form. The runtime and tools use it for loading, reflection, verification, and diagnostics.

### 51. What is managed code?

Managed code runs under a .NET runtime and uses its services, including type safety and garbage collection. It can still call unmanaged code, which requires explicit lifetime and safety care.

### 52. What is the difference between managed and unmanaged resources?

Managed objects are reclaimed by the garbage collector when unreachable; unmanaged resources such as file handles need deterministic release, typically through `IDisposable` or `SafeHandle`.

### 53. What is the Base Class Library (BCL)?

The BCL is the foundational set of APIs for common operations such as collections, files, text, networking, and threading. Its exact API surface depends on the target framework and runtime.

### 54. What is a framework-dependent application?

A framework-dependent application ships application files but uses a compatible shared .NET runtime installed on the target. It is usually smaller, but requires runtime installation and servicing on that machine.

### 55. What is a runtime identifier (RID)?

A RID identifies a target platform and architecture, for example `linux-x64` or `win-arm64`. RID-specific publishing can select native assets and produce platform-specific output.

### 56. What is the purpose of `global.json`?

`global.json` influences which installed SDK the CLI selects for a directory tree. It can pin an SDK version or define roll-forward behavior, but does not select the application's runtime.

### 57. What is roll-forward in SDK selection?

SDK roll-forward controls whether the CLI may use a compatible installed SDK when the requested version is unavailable. Runtime roll-forward is a separate policy governing selection of a runtime for an application.

### 58. What is a shared framework?

A shared framework is a set of runtime and library files installed once and used by framework-dependent applications, such as `Microsoft.NETCore.App` or `Microsoft.AspNetCore.App`.

### 59. How does runtime version selection work?

A framework-dependent app declares required frameworks and versions; the host resolves installed frameworks according to runtime roll-forward rules and configuration. Self-contained apps carry their runtime and are serviced by republishing.

### 60. What is the .NET host (`dotnet` host)?

The native host starts .NET applications, locates the runtime, resolves framework and application configuration, and launches managed code. `dotnet` can also invoke SDK commands.

### 61. What is a framework reference?

A framework reference tells a project to compile against a shared framework, making its APIs available without treating each framework assembly as an ordinary NuGet package. It is commonly implicit for web SDK projects.

### 62. What is the difference between a package reference and framework reference?

A package reference restores a NuGet package, while a framework reference uses APIs supplied by an installed or bundled shared framework. Runtime availability and deployment behavior differ accordingly.

### 63. What does `AssemblyLoadContext.Default` do?

The default load context resolves the application's primary dependency graph. Additional contexts can isolate plugin dependency versions, but type identity and sharing across contexts require deliberate design.

### 64. What is an application dependency manifest?

The generated `.deps.json` records an application's compile/runtime dependencies, assets, and applicable runtime targets. The host and runtime use it to resolve files during startup.

### 65. What is a runtime configuration file?

The `.runtimeconfig.json` file records target framework and runtime settings used by the host. It can include framework requirements and selected runtime configuration properties.

### 66. What is a RID-specific native asset?

A native asset is a platform-compiled library selected for a matching RID, often from a NuGet package. It must match the operating system, architecture, and native ABI expected by the process.

### 67. What is the difference between `AnyCPU` and a RID?

`AnyCPU` describes managed assembly processor targeting; a RID specifies a deployment platform and architecture for runtime/native asset resolution. A managed assembly can be architecture-neutral while its native dependencies are not.

### 68. What is a process architecture mismatch?

A mismatch occurs when a process architecture cannot load a native dependency built for another architecture, such as x64 code in an arm64-only process. Publish and install compatible runtime/native assets.

### 69. What is the role of `System.Runtime`?

`System.Runtime` is a foundational reference assembly defining core runtime contracts and types. At execution, implementation details are provided by the selected runtime rather than by the reference assembly used at compile time.

### 70. What is a reference assembly?

A reference assembly contains API metadata for compilation but generally not executable implementation. It enables compilation against a target API contract without copying runtime implementation code.

### 71. What does API compatibility mean in .NET?

API compatibility means a newer library or runtime preserves expected public contracts for consumers. Source, binary, and behavioral compatibility are distinct, and compatibility guarantees depend on the component and release policy.

### 72. What is a breaking change?

A breaking change causes existing source or binaries to fail, or changes behavior in a way consumers cannot safely rely on. Check release notes, analyzers, and compatibility tools before upgrading major platform versions.

### 73. What is the difference between an LTS and STS release?

They are support-track labels used for .NET releases; LTS releases have a longer support window than STS releases. Exact release cadence and end-of-support dates change, so consult the current official support policy.

### 74. What is a servicing update?

A servicing update is a supported patch release that addresses fixes such as security or reliability issues within a release line. Apply supported updates and validate them through normal deployment testing.

### 75. What does `Environment.Version` tell you?

It returns a `System.Version` value associated with the runtime, but it is not a reliable general-purpose way to infer the product release or target framework. Use explicit framework and deployment metadata when identifying those.

### 76. What is `RuntimeInformation` used for?

`RuntimeInformation` exposes runtime and operating-system information such as OS description and process architecture. Prefer capability checks where possible instead of branching on platform names unnecessarily.

### 77. How should platform-specific code be isolated?

Keep platform-dependent operations behind small abstractions or guarded code paths and use platform annotations where available. Test on each supported platform because API availability does not guarantee identical behavior.

### 78. What is a native interop boundary?

It is the boundary where managed code calls native functions or exposes callbacks to native code, commonly through P/Invoke or source-generated interop. Correct ABI, ownership, marshalling, and error handling are essential.

### 79. What does `DllImport` do?

`DllImport` declares a managed method that calls an exported native function using P/Invoke. Newer .NET versions also provide source-generated alternatives such as `LibraryImport` for supported scenarios.

### 80. Why can native dependencies make deployment platform-specific?

Native libraries are compiled for particular operating systems and architectures and may depend on system libraries or ABI versions. Publish and test for each intended RID and container/base-image combination.

## 8. SDK, CLI, and MSBuild

### 81. What is the .NET SDK?

The SDK combines the CLI, MSBuild, compilers, templates, and targeting packs needed to build .NET projects. Multiple SDK versions can coexist on one machine.

### 82. How does `dotnet restore` work?

Restore resolves declared dependencies for target frameworks and runtime targets, downloads packages from configured sources, and writes assets metadata. Locked mode can enforce a previously recorded dependency graph in reproducible builds.

### 83. What happens during `dotnet build`?

The CLI invokes MSBuild to restore when needed and execute the configured build targets, including compilation and resource processing. Properties, target framework, configuration, and SDK version affect the result.

### 84. What does `dotnet publish` produce?

Publish produces application output intended for deployment, including selected dependencies and optionally runtime assets. Options determine framework-dependent versus self-contained, RID-specific, trimmed, single-file, or AOT output.

### 85. What does `dotnet clean` remove?

It invokes MSBuild clean targets to remove generated build outputs such as `bin` and `obj` artifacts for the selected project/configuration. It does not remove source files or NuGet's global package cache.

### 86. What is MSBuild?

MSBuild is the build engine that evaluates project files, properties, items, targets, and tasks to construct projects. The .NET SDK supplies common targets and tasks on top of it.

### 87. What is an MSBuild target?

A target is a named set of tasks that runs at a defined point in the build graph, with dependencies and conditions. Custom targets should integrate with established SDK targets rather than rely on fragile execution order.

### 88. What is an MSBuild task?

A task performs a unit of build work, such as compiling source, copying files, or generating resources. Tasks can be built in or supplied by SDKs and packages.

### 89. What are MSBuild properties?

Properties are named scalar values that configure project evaluation and target execution, such as `Configuration`, `TargetFramework`, and `PublishAot`. They can be set in project files, imported files, or command-line arguments.

### 90. What are MSBuild items?

Items represent collections of files or other values, often with metadata. SDK-style projects implicitly include common source files, while item updates and removals adjust those defaults.

### 91. What is `Directory.Build.props`?

It is a conventionally imported MSBuild file for sharing properties and item definitions across projects below a directory. A matching `Directory.Build.targets` commonly contains shared target customizations.

### 92. What is `Directory.Packages.props`?

It is the convention used by NuGet Central Package Management to declare package versions centrally. Projects can then reference package IDs without duplicating version numbers.

### 93. What is central package management?

Central Package Management keeps package versions in a shared props file, helping multi-project repositories align dependency versions. It does not itself guarantee vulnerability-free or reproducible packages.

### 94. What is a NuGet lock file?

A `packages.lock.json` records the resolved dependency graph for a project. Committing it and restoring in locked mode helps detect unintended dependency changes in controlled builds.

### 95. What is the difference between `PackageReference` and `packages.config`?

`PackageReference` is the modern project-file-based dependency model with transitive restore and broad SDK integration. `packages.config` is an older per-project manifest primarily associated with legacy project systems.

### 96. What is a NuGet source?

A source is a feed from which NuGet restores packages, such as nuget.org or a private registry. Control source trust and package-source mapping to reduce dependency-confusion risks.

### 97. What does `dotnet test` do?

It builds and runs test projects through their configured test SDK and adapter. Test framework selection and command options determine discovery, filtering, logging, and result output.

### 98. What does `dotnet run` do?

It builds and runs a project, typically using the selected SDK and development configuration. It is convenient for development, not a substitute for testing the published deployment artifact.

### 99. How do you pass MSBuild properties through the CLI?

Use `-p:Name=Value` with a `dotnet` command that forwards MSBuild arguments, for example `dotnet publish -p:PublishSingleFile=true`. Confirm the property is meaningful for the selected target and SDK.

### 100. What is a configuration in MSBuild?

A configuration is a named build variant, commonly `Debug` or `Release`, that selects properties such as optimization and debug symbols. Projects may define additional configurations.

### 101. What is multi-targeting?

Multi-targeting builds one project for several TFMs, declared with `TargetFrameworks`. Conditional properties or source can adapt to APIs available in each target, but every target should be built and tested.

### 102. What is conditional compilation?

Conditional compilation uses symbols and directives such as `#if` to include code only under selected build conditions. Prefer shared APIs or multi-targeting abstractions unless separate source is necessary.

### 103. How do you add a project reference?

Add a `ProjectReference` so MSBuild builds and links the referenced project's output and metadata. Project references are appropriate within a coordinated source tree; package references suit separately versioned reusable components.

### 104. What is an SDK-style project?

An SDK-style project uses the `Sdk` attribute and convention-based defaults for common build items and targets. It reduces boilerplate and supports modern .NET workflows, though customizations still use MSBuild.

### 105. What does `ImplicitUsings` change?

When enabled for supported SDK projects, the SDK generates global using directives for commonly used namespaces. It changes compilation convenience, not the APIs available to the project.

### 106. What are nullable reference types?

Nullable reference types enable compiler analysis distinguishing intended nullable and non-null references. The annotations and warnings improve static checking but do not add runtime null checks.

### 107. What are analyzers in a .NET project?

Analyzers inspect source for correctness, style, performance, or security patterns during development or build. Rules can come from the SDK, packages, or editor configuration and can be elevated to errors selectively.

### 108. What is a source generator?

A source generator runs during compilation and adds source code based on the compilation or inputs. It can reduce runtime reflection or boilerplate, but generated output and diagnostics should be inspected when debugging.

### 109. What is deterministic compilation?

Deterministic compilation aims to produce identical outputs from identical inputs, independent of incidental machine state. Source paths, timestamps, compiler versions, and embedded generated inputs can affect reproducibility.

### 110. How can a build be made more reproducible?

Pin the SDK, control package sources and versions, use lock files where suitable, avoid time-dependent inputs, and build from clean source in a consistent environment. Record provenance and validate published artifacts.

## 9. Assemblies, loading, and reflection

### 111. What is an assembly's identity?

An assembly identity includes its simple name, version, culture, and public key token when strong-named. Runtime resolution also depends on load context and deployment assets.

### 112. What is a strong name?

A strong name gives an assembly a cryptographic identity based on its name and signing key. It is not a security boundary or proof that the code is trustworthy.

### 113. Does strong naming sign an assembly for trust?

No. Strong names identify assemblies and support versioning scenarios; they do not establish publisher trust in the way Authenticode signing or a trusted distribution process may.

### 114. What is the Global Assembly Cache (GAC)?

The GAC is a Windows mechanism used by .NET Framework for shared assemblies. Modern .NET does not use the GAC as its general application dependency store; applications typically deploy dependencies with their output or use shared frameworks.

### 115. What is `AssemblyLoadContext`?

`AssemblyLoadContext` is the modern .NET mechanism for resolving assemblies into an isolated loading context, useful for dependency isolation and collectible plugin designs. Context boundaries affect type identity.

### 116. What is a collectible `AssemblyLoadContext`?

A collectible context can be unloaded after its loaded assemblies and all associated objects become unreachable. Unload is cooperative and garbage-collection-driven, not an immediate force-unload operation.

### 117. How can plugin dependency conflicts be isolated?

Load plugin assemblies into separate contexts with an explicit resolver and share only agreed contract assemblies. Avoid exchanging types from independently loaded copies of the same contract assembly.

### 118. Why can two identically named types be incompatible?

Type identity includes the defining assembly and loading context, not just the namespace and type name. Duplicate contract assemblies loaded in different contexts can therefore yield distinct runtime types.

### 119. What does reflection provide?

Reflection inspects metadata and types at runtime and can invoke members dynamically. It is powerful for frameworks and plugins, but can add cost and complicate trimming and Native AOT.

### 120. What is `Type.GetType` used for?

`Type.GetType` resolves a type from a name according to the supplied assembly-qualified name and resolution rules. Prefer direct type references or explicit assembly resolution when possible.

### 121. What are custom attributes?

Custom attributes attach metadata to assemblies, types, or members. Reflection can read them, while source generators and analyzers can inspect them at compile time.

### 122. What is reflection-based activation?

Reflection-based activation creates objects by discovering types and constructors at runtime, for example through `Activator.CreateInstance`. Dynamic discovery may need explicit preservation annotations for trimmed or AOT applications.

### 123. What is trimming compatibility metadata?

Annotations such as `DynamicallyAccessedMembers` communicate to the trimmer which reflected members must be retained. They help static analysis but must accurately describe runtime reflection behavior.

### 124. What is a plugin contract assembly?

A plugin contract assembly defines shared interfaces and data types used between host and plugin. Keep it stable and load a single shared instance across contexts to preserve type identity.

### 125. What is a satellite assembly?

A satellite assembly contains culture-specific resources separate from the main assembly. Resource lookup uses culture fallback to select localized resources.

### 126. What is an assembly manifest?

An assembly manifest records metadata such as identity and references to modules or resources. It is part of the assembly's metadata used by tools and the runtime.

### 127. What is a module in .NET?

A module is a portable executable file containing metadata and code; an assembly can contain one or more modules, although typical modern projects produce a single-module assembly.

### 128. What is IL verification?

Verification checks whether CIL satisfies rules that support type safety and safe execution. Runtime behavior and verification guarantees vary by implementation and scenario; unsafe code and interop can bypass managed safety.

### 129. What is unsafe code?

Unsafe C# permits pointer operations and other low-level constructs that bypass some runtime type-safety guarantees. Use it only when needed, tightly scope it, and validate memory ownership and bounds.

### 130. What does `Reflection.Emit` do?

`Reflection.Emit` creates dynamic assemblies or types at runtime on supported runtimes. Dynamic code generation is constrained or unavailable in some Native AOT scenarios.

### 131. What is dynamic code?

Dynamic code is executable code generated or compiled at runtime, including some expression compilation and emit APIs. It can conflict with AOT, trimming, sandboxing, and startup goals.

### 132. What is an expression tree?

An expression tree represents code as data that can be inspected, transformed, or translated, for example by query providers. Compiling an expression into executable code may require dynamic code support.

### 133. What is the difference between `dynamic` and reflection?

C# `dynamic` defers member binding to runtime through the runtime binder; reflection explicitly discovers and invokes metadata-driven members. Both reduce compile-time checking compared with statically typed calls.

### 134. What are metadata load contexts?

`MetadataLoadContext` enables inspection of assembly metadata without loading assemblies for execution, useful for tooling. It is distinct from an execution `AssemblyLoadContext`.

### 135. Why should assembly resolution be explicit in plugin systems?

Explicit resolution makes dependency versions, probing paths, and shared contracts predictable. Uncontrolled probing can load unintended binaries, create type identity problems, or introduce security and reliability risks.

## 10. Memory management, JIT, and AOT

### 136. What is garbage collection?

Garbage collection reclaims managed memory for objects that are no longer reachable from GC roots. It does not deterministically release external resources or guarantee an exact collection time.

### 137. What are GC generations?

The generational collector groups objects by survival age so it can often collect short-lived objects cheaply. The classic managed heap uses generations 0, 1, and 2, with large objects handled on a separate heap.

### 138. What is the large object heap?

The large object heap stores sufficiently large allocations, with the threshold commonly around 85,000 bytes for objects but implementation details can vary. Large allocation patterns can increase memory pressure and fragmentation.

### 139. What is the pinned object heap?

The pinned object heap is a dedicated heap introduced in modern .NET for pinned objects, helping reduce fragmentation of other heaps. Exact allocation behavior depends on runtime and API usage.

### 140. What is a GC root?

A GC root is a reference location the collector treats as a starting point for reachability, such as a stack local, static field, or handle. Objects reachable from roots remain alive.

### 141. What is a memory leak in a garbage-collected application?

A managed memory leak occurs when objects remain reachable unintentionally and therefore cannot be collected. Event subscriptions, caches, static references, and unbounded queues are common causes.

### 142. What is object finalization?

A finalizer provides a nondeterministic fallback opportunity to release unmanaged resources. Finalization delays reclamation and should generally be avoided in favor of `SafeHandle` or deterministic disposal.

### 143. What is `IDisposable`?

`IDisposable` defines deterministic cleanup through `Dispose`, usually invoked by `using`. It is a convention for resource ownership and does not itself force garbage collection.

### 144. What does `IAsyncDisposable` provide?

`IAsyncDisposable` allows asynchronous cleanup through `DisposeAsync`, useful when releasing a resource requires asynchronous work. Use `await using` where ownership and scope require it.

### 145. What is `SafeHandle`?

`SafeHandle` wraps an unmanaged handle and provides reliable release behavior, including protection against many finalization and exception-path hazards. Prefer it over hand-written finalizers for native handles.

### 146. What is a weak reference?

A weak reference allows an object to be observed without keeping it alive solely because of that reference. It is useful for some caches, but does not replace clear ownership or bounded cache policies.

### 147. What is a `Span<T>`?

`Span<T>` is a stack-only view over contiguous memory that can refer to arrays, native memory, or other storage. It enables low-allocation slicing but cannot be stored on the managed heap or across `await`.

### 148. What is `Memory<T>`?

`Memory<T>` is a heap-storable memory view that can cross asynchronous boundaries. Access its span when needed, while ensuring the underlying memory remains valid for the operation.

### 149. What is boxing?

Boxing wraps a value type in an object representation, often causing an allocation; unboxing extracts it with a runtime type check. Generic APIs can avoid many boxing cases.

### 150. What is stack allocation with `stackalloc`?

`stackalloc` reserves a bounded block of memory on the stack, often for short-lived spans. Large or unbounded stack allocations can cause stack overflow and must be sized carefully.

### 151. What does the JIT compiler do?

The just-in-time compiler translates managed intermediate language into native machine code as methods are needed. It can optimize using runtime and processor information.

### 152. What is tiered compilation?

Tiered compilation allows the runtime to initially use quickly generated code and later recompile hot methods with more optimization. Behavior and defaults can vary across runtime versions and configuration.

### 153. What is dynamic PGO?

Dynamic profile-guided optimization uses runtime-observed behavior to improve optimized code generation in supported modern .NET versions. Its benefits depend on workload, runtime version, and warm-up behavior.

### 154. What is ReadyToRun?

ReadyToRun compiles portions of managed assemblies ahead of time to native code to improve startup, while retaining IL for fallback and runtime optimization. It can increase publish size and is not equivalent to full Native AOT.

### 155. What is crossgen2?

Crossgen2 is the toolchain used for ReadyToRun compilation in modern .NET. SDK publishing typically invokes it when the relevant ReadyToRun options are enabled.

### 156. How does Native AOT differ from JIT execution?

Native AOT compiles the application to native code at publish time and does not require a JIT at runtime. It has different constraints around dynamic code, reflection, and deployment targets.

### 157. What is a trimming warning?

A trimming warning indicates that static analysis cannot guarantee some code or metadata required at runtime will be preserved. Resolve warnings or document and test a deliberate suppression against the published app.

### 158. What is a source-generated serializer?

A source-generated serializer emits serialization metadata and code at compile time, reducing runtime reflection and improving compatibility with trimming or AOT. `System.Text.Json` supports source generation in modern .NET.

### 159. What is a GC pause?

A GC pause is a period when managed threads may be suspended for collection work. Pause duration depends on heap size, allocation patterns, collector mode, and workload.

### 160. What are workstation and server GC modes?

Workstation GC is tuned primarily for client responsiveness, while server GC is designed for throughput on server workloads and can use multiple heaps. Choose based on deployment characteristics and measure.

### 161. What is concurrent garbage collection?

Concurrent GC performs much of certain collection work alongside application threads to reduce pause time, with tradeoffs in CPU and memory. Exact supported modes depend on runtime and GC configuration.

### 162. What is the difference between allocation rate and retained memory?

Allocation rate measures how quickly objects are created; retained memory measures live objects still reachable. High allocation can cause frequent collections even when the live heap remains modest.

### 163. Why are immutable objects useful for concurrency?

Immutable objects cannot be modified after construction, reducing synchronization needs and making sharing easier to reason about. They may still incur copying or allocation costs.

### 164. What is false sharing?

False sharing occurs when threads update different values that occupy the same cache line, causing cache-coherence contention. It is a low-level performance issue to address only after profiling.

### 165. What is a memory barrier?

A memory barrier constrains reordering and visibility of memory operations across threads. Most application code should use higher-level synchronization primitives rather than manual barriers.

### 166. What is `Interlocked`?

`Interlocked` provides atomic operations such as increment, exchange, and compare-exchange for supported values. Atomic operations do not make arbitrary multi-step state transitions automatically thread-safe.

### 167. What does `volatile` mean in C#?

`volatile` constrains certain reads and writes for visibility and ordering, but does not make compound operations atomic. Locks, `Interlocked`, or other synchronization are usually clearer.

### 168. What is a lock convoy?

A lock convoy occurs when many threads repeatedly queue for a contended lock, reducing throughput and increasing latency. Reduce critical-section work or use an appropriate concurrency design after measuring contention.

### 169. What is a thread pool starvation symptom?

Requests or queued work stall while thread-pool threads are blocked or unavailable, often producing high latency despite low CPU. Avoid sync-over-async and diagnose with runtime counters and traces.

### 170. Why should asynchronous I/O not be wrapped in `Task.Run` on a server?

Native asynchronous I/O generally does not need a worker thread while waiting; wrapping it in `Task.Run` adds scheduling overhead and can consume thread-pool capacity. Use `Task.Run` mainly for suitable CPU-bound work.

### 171. What is `ValueTask<T>`?

`ValueTask<T>` can avoid allocating a `Task<T>` when a result is often available synchronously. It has usage constraints, so use it only when measurements justify the added complexity.

### 172. What is a `Task`?

A `Task` represents an asynchronous operation's completion, result, or failure. It is not necessarily a dedicated thread; awaiting it allows execution to continue without blocking the current thread.

### 173. What is a cancellation token?

A `CancellationToken` communicates a cooperative request to stop work. Operations must observe it and clean up; cancellation does not forcibly terminate arbitrary code.

### 174. What is a race condition?

A race condition occurs when the outcome depends on the timing or interleaving of concurrent operations. Protect shared state with synchronization or design it to avoid unsafely shared mutation.

### 175. What is an async deadlock?

An async deadlock can occur when code synchronously blocks on asynchronous work that needs a captured context to continue. Prefer `await` end-to-end and avoid `.Result` or `.Wait()` in request paths.

## 11. ASP.NET Core servers and pipeline

### 176. What is ASP.NET Core?

ASP.NET Core is the cross-platform web framework for building HTTP services and web applications on modern .NET. It provides hosting, servers, middleware, routing, APIs, and integration with platform services.

### 177. What is Kestrel's role in web hosting?

Kestrel is ASP.NET Core's default cross-platform HTTP server and can listen directly for application traffic. It can also run behind a reverse proxy or platform ingress.

### 178. What is a reverse proxy?

A reverse proxy accepts client connections and forwards requests to an application server. It may provide TLS termination, load balancing, buffering, or edge controls, depending on configuration.

### 179. How does IIS integrate with ASP.NET Core?

IIS can host an ASP.NET Core app in-process or forward requests to a separate Kestrel process out-of-process. The hosting model affects process boundaries, configuration, and deployment behavior.

### 180. What does the ASP.NET Core Module do?

On IIS, the ASP.NET Core Module starts or forwards to the application process and manages integration with IIS. Its behavior and supported options depend on installed hosting components and deployment mode.

### 181. What is HTTP.sys hosting?

HTTP.sys is a Windows-only HTTP server option built on the Windows kernel HTTP stack. It supports Windows-specific features and is not available as a cross-platform server.

### 182. What is a connection middleware?

Connection middleware operates on server connections before individual HTTP requests are processed. It is useful for connection-level concerns and differs from ordinary request middleware.

### 183. What does `Use` do in an ASP.NET Core pipeline?

`Use` adds middleware that can invoke the next delegate, modify request/response behavior, or short-circuit. Its position determines which requests and responses it can observe.

### 184. What does `Run` do in middleware configuration?

`Run` adds terminal middleware that does not call a next component. It short-circuits the pipeline for requests reaching it.

### 185. What does `Map` do in the HTTP pipeline?

`Map` branches the pipeline based on a path prefix or predicate. Requests on the branch follow its middleware rather than the unbranched continuation.

### 186. What is endpoint routing?

Endpoint routing matches requests to endpoint metadata and handlers, enabling middleware such as authorization to inspect the selected endpoint before execution. Routing and endpoint execution are separate pipeline stages.

### 187. What is endpoint metadata?

Endpoint metadata stores information associated with a route, such as authorization requirements, names, or API descriptions. Middleware and tooling can inspect that metadata without executing the handler.

### 188. Why does middleware ordering matter?

Middleware runs in registration order on the request and generally in reverse order as control returns. Ordering determines whether components see routed endpoints, authenticated users, or transformed responses.

### 189. Where should exception-handling middleware run?

Place central exception handling early enough to observe downstream failures, before components whose exceptions it should handle. It cannot catch failures that occur outside its execution scope.

### 190. What does forwarded-header middleware do?

It processes trusted proxy headers such as the original client address and scheme. Restrict known proxies or networks; accepting arbitrary forwarded headers enables spoofing.

### 191. What is request buffering?

Request buffering stores some or all request body data so it can be read more than once. It consumes memory or disk and should be limited to cases that need replay.

### 192. What is response buffering?

Response buffering delays sending response data, enabling operations such as modification or compression but increasing memory use and latency. Streaming is preferable for large responses where feasible.

### 193. What is response compression?

Response compression reduces response bytes using supported encodings, trading CPU for bandwidth. Avoid compression configurations that expose secrets through length side channels in sensitive contexts.

### 194. What is static-file middleware?

Static-file middleware serves eligible files from configured web roots. Place it and configure content types and caching deliberately; do not expose private or user-uploaded content unintentionally.

### 195. What is a web root?

The web root is the filesystem location intended for public static content, commonly `wwwroot`. Files outside it are not automatically public, but application code may still expose them.

### 196. What is a route constraint?

A route constraint narrows which route values match a route template, such as requiring an integer. It helps disambiguate routes but is not a substitute for validating domain input.

### 197. What is a route group in minimal APIs?

A route group applies a shared prefix and conventions, such as authorization or tags, to a set of endpoints. It helps organize endpoints and centralize consistent metadata.

### 198. What is an endpoint filter?

An endpoint filter wraps minimal API handler execution to inspect arguments, results, or exceptions. Use it for endpoint-scoped cross-cutting behavior rather than general-purpose HTTP pipeline concerns.

### 199. What is an MVC action filter?

An action filter runs around MVC action invocation and can inspect action arguments or results. Its lifecycle and scope are different from middleware and minimal API endpoint filters.

### 200. What is an authorization filter?

An authorization filter participates in MVC authorization processing. For modern endpoint-based policies, use ASP.NET Core authorization services and endpoint metadata where appropriate.

### 201. What is a model binder?

A model binder converts request data into action parameter or model values. Custom binders can handle domain-specific formats, but should report invalid input clearly and avoid trusting client values.

### 202. What is an input formatter?

An input formatter deserializes an HTTP request body into an MVC model, commonly from JSON. Supported media types and formatter settings determine how content is interpreted.

### 203. What is an output formatter?

An output formatter serializes an MVC response object according to content negotiation and configured media types. Explicit API contracts help prevent accidental exposure of internal model fields.

### 204. What is content negotiation?

Content negotiation selects a response representation based on request headers and server capabilities. APIs should define predictable behavior for unsupported media types and formats.

### 205. What is an HTTP middleware short circuit?

A short circuit occurs when a middleware component completes a response without invoking the next component. It can reduce unnecessary work but may intentionally bypass downstream behavior.

### 206. What is a request delegate?

A `RequestDelegate` represents asynchronous handling of an `HttpContext`. Middleware composes delegates to implement the request pipeline.

### 207. What is `HttpContext`?

`HttpContext` represents the current HTTP request and response plus connection and user information. It is request-scoped and should not be retained after request processing.

### 208. Why should request state not be stored in singleton middleware fields?

Middleware instances are usually reused across requests, so mutable per-request fields can race or leak data between callers. Store request-specific data in local variables or request-scoped facilities.

### 209. What is an `IHttpContextAccessor` caveat?

`IHttpContextAccessor` exposes the current context through ambient state, but the context may be absent outside a request and should not be retained. Prefer explicit parameter passing where practical.

### 210. How should large HTTP responses be served?

Stream data incrementally and honor client disconnect cancellation rather than buffering the full payload. Set correct content headers and manage source-resource lifetime through completion.

## 12. Hosting, DI, configuration, and logging

### 211. What is `WebApplication`?

`WebApplication` is the streamlined ASP.NET Core application builder and host introduced in .NET 6. It configures services, middleware, endpoints, and lifetime for common web applications.

### 212. What is `WebApplicationBuilder`?

`WebApplicationBuilder` provides a high-level way to configure the host, web server, configuration, logging, and DI services. It builds the `WebApplication` that runs the pipeline.

### 213. What is the Generic Host used for outside web applications?

The Generic Host provides DI, configuration, logging, lifetime, and hosted-service support for worker and console applications. It gives background processes lifecycle behavior similar to web apps.

### 214. What is `IHostApplicationLifetime`?

It exposes application lifecycle notifications and a stopping token. Use these notifications for coordinated shutdown, not as a replacement for disposing resources or handling cancellation.

### 215. How do you register a hosted background worker?

Register it with hosted-service registration such as `AddHostedService<T>()`. If the worker needs scoped dependencies, create a scope per work unit rather than holding scoped objects indefinitely.

### 216. How does a hosted service get scoped dependencies?

Inject `IServiceScopeFactory`, create an async or regular scope for each unit of work, and resolve scoped services inside it. Dispose the scope promptly, including on cancellation or failure.

### 217. What is dependency injection scope validation?

Scope validation detects invalid lifetime relationships such as resolving scoped services from the root provider or capturing them in singletons. Enable it in development and fix violations rather than suppressing them.

### 218. What does `IServiceProvider` represent?

`IServiceProvider` resolves registered services from a container or scope. Excessive service-locator use hides dependencies; prefer constructor injection for ordinary application services.

### 219. What is `IServiceScope`?

An `IServiceScope` provides a bounded lifetime for scoped services and disposes them when the scope ends. Use a request scope or explicitly created scope appropriate to the unit of work.

### 220. How are disposable DI services released?

The container disposes services it creates when their owning scope or provider is disposed. Do not manually dispose container-owned instances unless ownership was explicitly transferred.

### 221. What is configuration provider precedence?

When multiple providers define a key, the effective value is determined by provider ordering, commonly with later providers overriding earlier ones. Review actual host defaults and custom registration order.

### 222. How are environment variables mapped to configuration keys?

The default configuration system commonly maps double underscores (`__`) in environment-variable names to configuration key separators (`:`). Naming and precedence should be verified for the host setup.

### 223. What are user secrets?

User secrets store development settings outside the project tree for local workflows. They are not encrypted production secret storage and should not be used as a deployment vault.

### 224. What is an options validation failure?

It occurs when configured options violate declared validators or data annotations. Validate at startup where possible so misconfiguration prevents the service from accepting traffic.

### 225. What is named options?

Named options provide separate configured instances of one options type under different names. This is useful when one service consumes multiple configurations of the same kind.

### 226. How does options reload work?

Reload-capable providers can notify options infrastructure of changes, and `IOptionsMonitor<T>` can expose updated values to singleton consumers. Reload behavior depends on provider support and validation.

### 227. What is a logging scope?

A logging scope attaches contextual properties to a group of log events, such as a correlation identifier. Scopes improve correlation but should contain only appropriate, non-sensitive data.

### 228. What is a logging provider?

A logging provider routes `ILogger` events to a destination such as console, debug output, or a telemetry backend. Providers determine formatting, filtering, buffering, and delivery behavior.

### 229. What is log filtering?

Log filtering selects which events are emitted based on category and severity level. Tune filters to retain useful operational signals without creating excessive cost or losing necessary diagnostics.

### 230. What is a logging category?

A logging category identifies the source of log events, commonly the fully qualified type name for `ILogger<T>`. Category filters can target application components or framework namespaces.

## 13. HTTP APIs, data, and serialization

### 231. What is a RESTful API?

A REST-style API models resources and uses HTTP methods and status codes according to their semantics. REST is an architectural style, not a requirement imposed by ASP.NET Core.

### 232. What does idempotent mean for an HTTP method?

An operation is idempotent when repeating the same request has the same intended effect as performing it once. HTTP defines method semantics, but an application's implementation must preserve them.

### 233. What is the difference between PUT and PATCH?

PUT generally replaces a resource representation, while PATCH applies a partial modification using a defined patch format. Define validation, concurrency, and null semantics clearly for either operation.

### 234. What is an ETag?

An entity tag is an opaque validator for a representation. Conditional requests using `If-Match` or `If-None-Match` can prevent lost updates or avoid transferring unchanged content.

### 235. How can optimistic concurrency be implemented in an API?

Use a version token or ETag and require clients to submit the version they observed. Reject stale updates, commonly with a precondition-failed or conflict response.

### 236. What is API versioning?

API versioning allows clients to target distinct contract versions through a defined strategy such as URL, header, or media type. Plan deprecation and compatibility policy rather than versioning every internal change.

### 237. What is OpenAPI?

OpenAPI is a language-neutral description format for HTTP APIs, documenting paths, operations, schemas, and security schemes. Tooling can generate documentation and clients from it.

### 238. What is a minimal API parameter source?

Minimal APIs can bind parameters from route, query, headers, body, services, or custom binding based on type and metadata. Use explicit binding attributes when inference could be ambiguous or surprising.

### 239. What is an API contract?

An API contract defines externally observable request and response shapes, status codes, and behavior. Keep it separate from persistence entities to reduce accidental coupling and exposure.

### 240. What is a DTO?

A data transfer object is a type designed to carry data across a boundary, such as an API request or response. DTOs make validation and public contract evolution explicit.

### 241. What is `System.Text.Json`?

`System.Text.Json` is the .NET JSON serialization library, supporting converters, source generation, and configurable naming and behavior. Defaults and available features vary by target framework version.

### 242. What is a JSON converter?

A `JsonConverter<T>` customizes reading and writing a particular type's JSON representation. Converters should validate input, avoid ambiguous formats, and be compatible with the application's trimming/AOT strategy.

### 243. What is JSON source generation?

JSON source generation creates serialization metadata or optimized serialization code at build time for declared types. It can reduce reflection needs and improve startup or AOT compatibility.

### 244. What is polymorphic JSON serialization?

Polymorphic serialization represents derived types through a declared base contract, often using discriminators. Configure allowed derived types explicitly to avoid ambiguous or unsafe type activation.

### 245. What is a serialization contract?

A serialization contract determines which members, names, types, and converters represent an object on the wire. Treat it as a public compatibility boundary and test representative payloads.

### 246. How should unknown JSON properties be handled?

Choose behavior according to forward-compatibility and validation needs: ignoring unknown fields can tolerate additive changes, while rejecting them can catch unexpected input. Apply the choice consistently to the contract.

### 247. What is a `JsonDocument`?

`JsonDocument` provides a read-only DOM over JSON and owns pooled resources that should be disposed. Clone elements that must outlive the document.

### 248. What is `JsonNode`?

`JsonNode` is a mutable JSON DOM for dynamic document manipulation. It is more convenient for edits than low-level readers but can allocate more than streaming APIs.

### 249. When should `Utf8JsonReader` be used?

Use `Utf8JsonReader` for forward-only, low-allocation processing of UTF-8 JSON. Its state must be preserved correctly when handling segmented input.

### 250. When should JSON be streamed?

Streaming is useful for large payloads because it avoids buffering the entire document in memory. The reader, writer, and HTTP body APIs must be used with correct asynchronous and cancellation handling.

### 251. What is content type validation?

Content type validation checks that a request's declared media type is supported for the endpoint before parsing its body. It should complement size limits and payload validation.

### 252. What is a Problem Details response?

Problem Details is a standard structured error representation for HTTP APIs, defined by RFC 7807 and updated by RFC 9457. ASP.NET Core supports producing these responses through built-in services and middleware.

### 253. What is pagination?

Pagination divides large result sets into bounded responses. Cursor/keyset pagination can remain stable and efficient under changes, while offset pagination is simpler but may become slow or inconsistent.

### 254. What is rate limiting for an API?

Rate limiting restricts request volume according to a partition and policy, helping protect capacity and fairness. Choose keys carefully, return useful retry information, and do not treat it as authentication.

### 255. What is an API idempotency key?

An idempotency key lets a service recognize retries of a logical operation and return a consistent result. Store keys and outcomes with appropriate scope, expiry, and concurrency protection.

### 256. What is a database transaction?

A transaction groups data operations so they commit or roll back according to the data store's guarantees. Keep transaction duration short and avoid assuming a database transaction also covers external services.

### 257. What is a unit of work?

A unit of work groups changes and coordinates persistence for one logical operation. ORM contexts often provide unit-of-work behavior, but explicit boundaries should match business transaction needs.

### 258. What is Entity Framework Core?

Entity Framework Core is a cross-platform object-relational mapper for .NET with LINQ querying, change tracking, and migrations. Provider capabilities and SQL behavior differ by database.

### 259. What is `DbContext` lifetime guidance?

`DbContext` is typically scoped to a unit of work or web request and is not thread-safe. Do not run concurrent operations on the same context instance.

### 260. What is a database migration?

A migration records a schema change that can be applied to a database. Review generated SQL, coordinate application compatibility, and use a controlled deployment process for production schema updates.

### 261. What is the N+1 query problem?

The N+1 problem issues one query for a result set and an additional query for each item. Fix it with an appropriate projection, eager loading, or batched query rather than unbounded per-item access.

### 262. What is connection pooling?

Connection pooling reuses physical database connections across logical opens to reduce setup cost. Dispose logical connections promptly and configure pool limits according to the server and workload.

### 263. What is a database provider?

A provider translates ORM operations to a specific database's capabilities and protocol. Query translation, transactions, types, and migration behavior are provider-specific.

### 264. Why parameterize SQL queries?

Parameters separate data from SQL syntax, preventing injection and enabling database plan reuse. Avoid concatenating untrusted values into command text.

### 265. What is a repository pattern?

A repository abstracts data access behind a domain-oriented interface. It can clarify boundaries, but wrapping an ORM without adding meaningful behavior may duplicate its existing abstraction.

### 266. What is a cache invalidation strategy?

It defines when cached values become stale and how they are refreshed or removed. Use bounded lifetimes, versioning, or event-driven invalidation as appropriate, and plan for stale or missing data.

### 267. What is distributed caching?

A distributed cache is shared across application instances, often through a network service. Account for serialization, network failure, expiration, consistency, and cache stampedes.

### 268. What is cache stampede?

A cache stampede occurs when many callers recompute an expired or missing item at once. Use request coalescing, jitter, stale-while-revalidate, or controlled refresh where suitable.

### 269. What is data serialization versioning?

Serialization versioning evolves payload schemas while preserving defined compatibility with old producers or consumers. Prefer additive, optional fields and test compatibility across supported versions.

### 270. Why should API clients use cancellation?

Cancellation stops unnecessary work when a client disconnects or no longer needs a result. Propagate request cancellation to downstream I/O when safe and avoid treating expected cancellation as an application fault.

## 14. Application and platform security

### 271. What is the principle of least privilege?

Grant each user, service, and process only the permissions needed for its task. Reduce blast radius by using narrow roles, scoped credentials, and separate identities across environments.

### 272. What is the difference between claims and roles?

Claims are name/value assertions about a principal; roles are commonly represented as claims and used for coarse grouping. Policies can evaluate claims and custom requirements for more precise authorization.

### 273. What is policy-based authorization?

Policy-based authorization combines requirements and handlers into named rules evaluated by the authorization system. Policies make access rules reusable and clearer than scattered controller conditionals.

### 274. What is resource-based authorization?

Resource-based authorization evaluates permission against a specific loaded resource, such as whether the caller owns a document. Load the resource and explicitly authorize it before revealing or changing it.

### 275. What is an authorization handler?

An authorization handler evaluates one or more requirements for a user and optional resource. It should implement a focused, testable access rule and avoid unsafe assumptions about request input.

### 276. What is OAuth 2.0?

OAuth 2.0 is an authorization framework for delegated access to protected resources. Use a vetted identity provider and appropriate flow rather than implementing token issuance or protocol details from scratch.

### 277. What is OpenID Connect?

OpenID Connect is an identity layer built on OAuth 2.0 that defines authentication and identity information. Validate issuer, audience, signature, expiry, and other required token properties.

### 278. What is a bearer token?

A bearer token grants access to whoever possesses it, so it must be protected in transit and storage. Use TLS, short appropriate lifetimes, audience restrictions, and secure client storage.

### 279. What is JWT validation?

JWT validation verifies cryptographic signature and required claims such as issuer, audience, and lifetime. Never authorize based only on decoding a token's readable payload.

### 280. What is cookie authentication?

Cookie authentication protects an authentication ticket in a browser cookie and reconstructs the principal on requests. Configure secure transport, HttpOnly, SameSite, expiration, and antiforgery protections as appropriate.

### 281. What is CSRF?

Cross-site request forgery tricks a browser into sending an authenticated request initiated by another site. Use antiforgery defenses for cookie-authenticated state-changing requests and suitable SameSite settings.

### 282. What is an antiforgery token?

An antiforgery token is a value paired with a browser cookie or session context to verify that a state-changing request originated from the expected application flow. It is not a general substitute for authorization.

### 283. What is cross-site scripting (XSS)?

XSS occurs when attacker-controlled content executes as script in a user's browser. Encode output for its context, sanitize allowed rich text, and use Content Security Policy as defense in depth.

### 284. How does Razor reduce XSS risk?

Razor HTML-encodes ordinary expression output by default. Explicitly rendering untrusted content as raw HTML bypasses that protection and requires trusted sanitization.

### 285. What is SQL injection?

SQL injection occurs when untrusted input is interpreted as SQL syntax. Use parameterized queries or safe ORM APIs, and never rely on string filtering as the primary defense.

### 286. What is command injection?

Command injection occurs when untrusted input alters an operating-system command. Avoid shell invocation when possible; otherwise use safe argument APIs, strict validation, and least-privileged execution.

### 287. What is path traversal?

Path traversal uses crafted paths to access files outside an intended directory. Canonicalize and constrain resolved paths to an allowed root, and avoid trusting user-provided filenames.

### 288. What is server-side request forgery (SSRF)?

SSRF causes a server to make attacker-influenced requests to unintended internal or external targets. Restrict destinations, validate resolved addresses, control redirects, and apply network egress policies.

### 289. What is the security risk of unsafe deserialization?

Deserializing attacker-controlled data into types with side effects or permissive polymorphism can enable dangerous behavior or resource exhaustion. Use constrained schemas, explicit type allowlists, and size/depth limits.

### 290. How should passwords be stored?

Store passwords using a vetted password-hashing scheme with per-password salts and suitable work factors, such as the ASP.NET Core Identity hasher. Never store plaintext or reversibly encrypted passwords.

### 291. What is password hashing versus encryption?

Hashing is a one-way transformation suited to password verification; encryption is reversible with a key. Password verification should use a dedicated adaptive password hash, not ordinary fast hashes.

### 292. What is key rotation?

Key rotation replaces cryptographic keys periodically or after compromise while preserving a controlled transition for data still protected by prior keys. Plan retention, access, and compatibility across instances.

### 293. What is ASP.NET Core Data Protection key persistence?

Data Protection keys can be persisted to configured storage and protected at rest. Multi-instance applications that share protected payloads need a shared, secured key ring and consistent application name.

### 294. What is a secret manager?

A secret manager stores and distributes sensitive values with access controls, auditing, and rotation support. Applications should retrieve secrets using managed identity or narrowly scoped credentials where available.

### 295. What is managed identity?

Managed identity is a cloud-provider feature that lets a workload obtain identity credentials without embedding long-lived secrets. Provider setup, token audience, and role assignments must be configured correctly.

### 296. What is certificate validation?

Certificate validation checks that a certificate chains to a trusted authority and is valid for the requested identity and usage. Do not disable validation to work around TLS errors.

### 297. What is TLS termination?

TLS termination is where encrypted client traffic is decrypted, often at a proxy or load balancer. Protect downstream hops as required and configure trusted forwarded metadata correctly.

### 298. What is HSTS preload?

HSTS preload is a browser-maintained list that forces HTTPS for listed domains. Enrollment has operational consequences and should follow the browser program's current requirements.

### 299. Why is CORS not an API access-control mechanism?

CORS restricts browser scripts from reading cross-origin responses; it does not stop direct HTTP clients from reaching the server. Enforce authentication and authorization independently.

### 300. What are security headers?

Security headers communicate browser policies such as content restrictions, framing rules, and transport security. Configure headers based on the application and avoid conflicting settings across proxies and app middleware.

### 301. What is Content Security Policy?

Content Security Policy (CSP) restricts sources and behaviors for browser-loaded content, reducing the impact of some injection attacks. Start with a tested policy and avoid unsafe broad allowances.

### 302. What is a security vulnerability in a NuGet dependency?

A vulnerable dependency contains a known flaw in a specific version or usage. Inventory direct and transitive packages, update to supported fixed versions, and assess exploitability in the application's context.

### 303. What is dependency confusion?

Dependency confusion exploits package-source resolution to substitute a malicious package for an internal dependency. Use source mapping, private package naming controls, and trusted feeds.

### 304. What is software supply-chain security?

Supply-chain security protects source, dependencies, build infrastructure, artifacts, and release processes. Use verified inputs, least-privileged automation, provenance, scanning, and signed artifacts where appropriate.

### 305. What is an SBOM?

A software bill of materials lists components included in a software artifact. It supports vulnerability response and inventory, but must be accurate and tied to the shipped build.

### 306. What is threat modeling?

Threat modeling systematically identifies assets, trust boundaries, likely threats, and mitigations before or during design. Revisit it when architecture, data flows, or attacker capabilities change.

### 307. What is defense in depth?

Defense in depth layers independent controls so failure of one does not expose the entire system. Examples include authorization, network restrictions, secure defaults, monitoring, and tested recovery.

### 308. How should an application handle untrusted input?

Validate input against explicit length, format, and domain rules; use context-safe encoding and parameterized APIs. Validation reduces invalid data but does not replace authorization or output encoding.

### 309. What is output encoding?

Output encoding transforms data so it is treated as data rather than executable syntax in a particular context, such as HTML or JavaScript. The correct encoding depends on the output context.

### 310. What is a denial-of-service risk from request bodies?

Oversized, deeply nested, or computationally expensive input can exhaust memory, CPU, or connections. Enforce body-size, timeout, concurrency, and parsing limits appropriate to the service.

### 311. Why avoid returning exception details to clients?

Exception details can reveal stack traces, paths, configuration, or secrets and aid attackers. Return a stable safe error response and retain diagnostic details in access-controlled telemetry.

### 312. What does secure cookie configuration include?

Use `Secure` over HTTPS, `HttpOnly` where JavaScript need not access the cookie, an appropriate `SameSite` policy, and a narrow domain/path and lifetime. Account for legitimate cross-site authentication flows.

### 313. What is token audience restriction?

Audience restriction ensures a token is accepted only by its intended resource. Validate the expected audience to prevent tokens issued for another service from being misused.

### 314. What is authorization over-fetching?

Authorization over-fetching occurs when an endpoint retrieves or returns data before checking whether the caller may access it. Authorize before disclosure and filter results to the caller's allowed scope.

### 315. How should security logging be designed?

Log security-relevant events with enough context for investigation while excluding passwords, bearer tokens, and unnecessary personal data. Restrict access and retention, and protect logs against tampering.

## 15. Observability, diagnostics, and performance

### 316. What is observability?

Observability is the ability to infer system behavior from emitted telemetry, principally logs, metrics, and traces. Instrument meaningful operations and correlate signals without collecting unnecessary sensitive data.

### 317. What is a metric?

A metric is a numeric measurement aggregated over time, such as request count, duration, or queue depth. Use bounded dimensions to avoid unbounded time-series cardinality.

### 318. What is a distributed trace?

A distributed trace records causally related operations across service boundaries using spans and context propagation. Trace identifiers help connect logs and requests, but sampled traces may not include every event.

### 319. What is OpenTelemetry?

OpenTelemetry is a vendor-neutral set of APIs, SDKs, semantic conventions, and protocols for telemetry. .NET instrumentation can export signals to compatible backends.

### 320. What is `Activity` in .NET?

`System.Diagnostics.Activity` represents an operation span and carries trace context and tags. ASP.NET Core and diagnostic libraries use activities for distributed tracing.

### 321. What is trace context propagation?

Trace context propagation carries trace and span identifiers across process boundaries, commonly through standardized HTTP headers. Validate incoming context and avoid treating trace IDs as trusted identity.

### 322. What is metric cardinality?

Cardinality is the number of unique time series produced by metric dimensions. Using unbounded values such as user IDs or raw URLs can overload telemetry storage and increase cost.

### 323. What are .NET runtime counters?

Runtime counters expose operational measurements such as GC heap size, allocation rate, thread-pool activity, and exception counts. Available counters vary by runtime version.

### 324. What is `dotnet-counters`?

`dotnet-counters` is a diagnostic CLI tool for monitoring .NET runtime and application counters. It can help identify trends such as GC pressure or thread-pool starvation.

### 325. What is `dotnet-trace`?

`dotnet-trace` collects EventPipe traces from .NET processes for analysis of runtime and application events. Trace collection settings affect overhead and data volume.

### 326. What is `dotnet-dump`?

`dotnet-dump` captures or analyzes process dumps for managed diagnostics such as heap inspection and thread state. Dumps can contain secrets and personal data, so protect and dispose of them securely.

### 327. What is EventPipe?

EventPipe is a cross-platform tracing mechanism for .NET runtime and application events. Diagnostic tools use it to collect data from running processes.

### 328. What are EventCounters?

EventCounters are a mechanism for publishing numeric diagnostic measurements through event infrastructure. They can be collected by compatible diagnostic tools and listeners.

### 329. What is a profiler?

A profiler observes execution to analyze time, allocation, contention, or call behavior. Profiling adds overhead, so use representative workloads and interpret results with the profiler's measurement model.

### 330. What is sampling in performance profiling?

Sampling periodically records execution state to estimate where time is spent with relatively low overhead. Short or rare events may be missed, so sampling is not an exact count of every call.

### 331. What is an allocation profile?

An allocation profile identifies code paths and types responsible for managed allocations. Combine it with throughput and GC measurements to distinguish costly allocation from harmless allocation.

### 332. What is a CPU profile?

A CPU profile estimates or records where processor time is spent. It helps identify hot code paths, but should be collected under representative load and interpreted alongside waiting time.

### 333. What is a flame graph?

A flame graph visualizes aggregated call stacks, with width generally representing a sampled metric such as CPU time. It helps locate expensive call paths but does not itself prove causation.

### 334. What is a performance baseline?

A performance baseline records measurements under a repeatable workload and environment for comparison. Include latency percentiles, throughput, resource use, and test conditions.

### 335. Why measure tail latency?

Tail latency, such as p95 or p99 response time, reveals slow experiences hidden by averages. Queuing, retries, dependencies, and pauses can disproportionately affect the tail.

### 336. What is Little's Law in service capacity?

Little's Law relates average concurrency, throughput, and time in a stable system: average items in the system equals arrival rate times average time. It helps reason about queues but does not replace load testing.

### 337. What is backpressure?

Backpressure limits producers when consumers cannot keep up, preventing unbounded queues and resource exhaustion. Use bounded channels or explicit flow-control policies where appropriate.

### 338. What is `System.Threading.Channels`?

Channels provide asynchronous producer-consumer coordination with configurable capacity and full-mode behavior. They can support bounded work queues without manually managing locks and condition variables.

### 339. What is a bounded queue?

A bounded queue caps pending work and defines what happens when full, such as waiting, rejecting, or dropping. It prevents unlimited memory growth and makes overload behavior explicit.

### 340. What is a thread-pool hill-climbing algorithm?

The thread pool adjusts worker-thread capacity based on observed throughput and blocking behavior. It is runtime-managed, but excessive blocking can still cause latency and starvation.

### 341. What is sync-over-async?

Sync-over-async blocks a thread while waiting on an asynchronous operation, for example with `.Result`. It can waste thread-pool capacity and cause deadlocks or latency under load.

### 342. What is benchmarking?

Benchmarking measures a narrowly defined operation under controlled conditions, often with a framework such as BenchmarkDotNet. Microbenchmarks need warm-up, multiple iterations, and realistic interpretation.

### 343. Why can microbenchmarks mislead?

They may omit real I/O, contention, JIT warm-up, deployment settings, or workload variability. Validate important optimizations with end-to-end measurements.

### 344. What is a cold-start cost?

A cold start is the work required before an application can serve its first useful request, including process startup, JIT, initialization, and dependency connection setup. Measure startup separately from steady-state throughput.

### 345. What is a startup health check?

A startup check indicates whether initialization has completed sufficiently for other health checks or traffic handling. It is distinct from ongoing liveness and readiness policies.

### 346. What is a readiness probe?

A readiness probe indicates whether an instance should receive traffic at that time. It should reflect whether the service can handle requests, without unnecessarily restarting a healthy process.

### 347. What is a liveness probe?

A liveness probe indicates whether the process is irrecoverably unhealthy and should be restarted. Avoid including transient dependency failures that could trigger restart loops.

### 348. What is a health-check dependency?

A dependency check reports availability of an external service such as a database. Use it selectively and distinguish readiness impact from process liveness.

### 349. What is a correlation ID?

A correlation ID associates related operations for diagnostics. Generate or validate IDs at trust boundaries and do not use them as authentication or authorization credentials.

### 350. What is a structured trace attribute?

A trace attribute is a key/value property attached to a span to describe its operation, such as a safe database system name. Keep values bounded and avoid secrets or high-cardinality identifiers.

## 16. Deployment, containers, and cloud

### 351. What is a container image?

A container image is a packaged filesystem and metadata used to create containers. It should include the application and required runtime/native dependencies while minimizing unnecessary tools and attack surface.

### 352. What is a multi-stage Docker build?

A multi-stage build uses separate build and runtime stages so the final image contains only deployment artifacts and required runtime components. It can reduce image size and exclude build tools.

### 353. What is the .NET SDK container image?

The SDK image contains tooling to restore and build .NET projects. It is commonly used in a build stage, while a runtime or ASP.NET runtime image serves as a smaller final base.

### 354. What is the ASP.NET runtime container image?

The ASP.NET runtime image includes the shared framework needed by framework-dependent ASP.NET Core apps. Image tags track platform releases; rebuild and redeploy to receive updated base layers.

### 355. What is a chiseled container image?

A chiseled image is a minimized Linux runtime image with unnecessary components removed. Reduced surface can improve security and size, but shell utilities or globalization support may be absent depending on variant.

### 356. What is a distroless-style image?

A distroless image contains an application runtime and essential files without a general-purpose package manager or shell. It can reduce attack surface but requires different debugging and operational practices.

### 357. What is a non-root container?

A non-root container runs the application under an unprivileged user, limiting impact if the process is compromised. Ensure writable paths, ports, and mounted volumes support that identity.

### 358. What is a container health check?

A container health check reports whether the process responds to a configured test. Orchestrators may use it for traffic or restart decisions, so align it with app health semantics.

### 359. What is container image pinning?

Pinning an image to a specific tag or digest makes the base image selection more predictable. Tags can move; digests identify immutable image content, though updates still need deliberate adoption.

### 360. What is a container layer?

A layer is a filesystem change captured in an image. Layer ordering affects cache reuse and download size; copy dependency manifests before frequently changing source where practical.

### 361. What is a Kubernetes deployment?

A Kubernetes Deployment manages a desired number of replicated pods and rollout behavior for a workload. Application probes, resource limits, and shutdown handling must align with deployment configuration.

### 362. What is a Kubernetes service?

A Kubernetes Service provides a stable network abstraction for reaching a set of pods. It is distinct from ASP.NET Core's in-process DI service registration.

### 363. How should an ASP.NET Core app handle SIGTERM in a container?

The host should receive shutdown notification, stop accepting work, and cancel hosted operations within the orchestrator's grace period. Configure termination grace time and application shutdown timeout consistently.

### 364. What is a rolling deployment?

A rolling deployment replaces instances gradually while preserving service capacity. Keep old and new application versions compatible with shared data and protocol during the transition.

### 365. What is blue-green deployment?

Blue-green deployment maintains two environments and switches traffic between them after validation. It enables fast rollback but requires compatible data changes and sufficient duplicate capacity.

### 366. What is a canary deployment?

A canary deployment exposes a new version to a small portion of traffic before broader rollout. Compare health and business metrics, and define automated or manual rollback criteria.

### 367. What is configuration externalization?

Configuration externalization keeps environment-specific settings outside the immutable application artifact. Validate settings on startup and secure sensitive values separately.

### 368. What is twelve-factor configuration?

The twelve-factor approach treats deploy-varying configuration as environment-provided values rather than embedded code constants. Apply it with secret handling and validation appropriate to the platform.

### 369. What is a sidecar?

A sidecar is a companion process or container deployed alongside an application to provide a supporting capability, such as proxying or telemetry. It introduces network and lifecycle failure modes to consider.

### 370. What is a service mesh?

A service mesh provides infrastructure-level service communication features such as identity, routing, and telemetry. Avoid duplicating controls blindly; understand which responsibilities remain in the application.

### 371. What is cloud managed identity integration?

Managed identity integration lets an application authenticate to supported cloud services using its assigned workload identity. Grant only required roles and avoid falling back to embedded credentials.

### 372. What is a cloud secret vault?

A secret vault stores and controls access to sensitive values, often with audit and rotation features. Fetch secrets securely, handle unavailability, and avoid logging or caching them indefinitely.

### 373. What is object storage?

Object storage persists data as objects addressed by keys, often with metadata and access policies. It is not a filesystem or transactional database; design for its consistency and API semantics.

### 374. What is a cloud load balancer?

A load balancer distributes connections or requests across service instances and may perform TLS termination or health-based routing. Configure forwarded headers and health behavior to match its topology.

### 375. What is autoscaling?

Autoscaling changes instance capacity based on configured signals such as load or queue depth. Scaling signals need adequate stabilization and the application must tolerate concurrent instances and cold starts.

### 376. What is a serverless .NET deployment?

A serverless deployment runs code through a managed event-driven platform that handles much of the infrastructure. Cold starts, execution limits, runtime support, and platform-specific hosting affect design.

### 377. What is an immutable deployment artifact?

An immutable artifact is built once and promoted unchanged through environments. It improves provenance and consistency; environment-specific configuration is supplied separately.

### 378. What is a deployment slot?

A deployment slot is a separately addressable environment used by some hosting platforms to stage and swap application versions. Slot-specific settings and warm-up behavior must be managed explicitly.

### 379. What is runtime patch management for self-contained apps?

Self-contained apps include a runtime copy, so they generally need to be republished and redeployed to pick up runtime servicing updates. Track supported versions and rebuild images and artifacts regularly.

### 380. How should a .NET app be deployed to a read-only filesystem?

Place writable state only in explicitly mounted or configured writable locations and avoid assumptions that the application directory can be modified. Test temporary-file and key persistence behavior under the deployment policy.

## 17. Testing and resilience

### 381. What is a unit test?

A unit test verifies a small unit of behavior in isolation, usually quickly and deterministically. Keep external infrastructure outside the test unless integration behavior is the subject.

### 382. What is an integration test?

An integration test verifies interaction across real or representative components, such as an HTTP pipeline and database. Control data, environment, cleanup, and external dependencies.

### 383. What is an end-to-end test?

An end-to-end test exercises a user-visible workflow across the deployed application boundary. Use it for a focused set of critical scenarios because it is slower and more failure-prone than unit tests.

### 384. What is `WebApplicationFactory<TEntryPoint>`?

`WebApplicationFactory<TEntryPoint>` from ASP.NET Core testing support hosts an application in-process for integration tests. It allows test-specific configuration and clients without requiring a deployed server.

### 385. What is `TestServer`?

`TestServer` hosts an ASP.NET Core request pipeline in memory for tests. It is useful for application-level HTTP testing but does not reproduce every socket, proxy, or production-server behavior.

### 386. What is a test double?

A test double substitutes for a dependency in a test; examples include fakes, stubs, and mocks. Use the simplest double that gives reliable evidence about the behavior under test.

### 387. What is contract testing?

Contract testing verifies that a service and its consumers agree on an interface or message schema. It can detect compatibility issues without requiring every full-system combination in each test.

### 388. What is a flaky test?

A flaky test sometimes passes and sometimes fails without a relevant code change, often due to timing, shared state, or external dependency variance. Fix the cause rather than repeatedly rerunning and ignoring it.

### 389. What is test isolation?

Test isolation prevents one test's state or execution from affecting another. Use unique data, controlled dependencies, deterministic clocks where useful, and reliable cleanup.

### 390. What is a resilience policy?

A resilience policy defines how an application responds to transient failure, delay, or overload. It should preserve correctness and have explicit limits rather than retrying indefinitely.

### 391. What is retry with exponential backoff?

It retries eligible transient failures with increasing delays, often bounded and jittered. Retry only operations safe to repeat and avoid multiplying retries across layers.

### 392. What is retry amplification?

Retry amplification occurs when multiple components retry independently, multiplying requests during an outage. Coordinate retry budgets and use bounded attempts with jitter.

### 393. What is a circuit breaker?

A circuit breaker temporarily stops calls to a failing dependency after a threshold, allowing recovery and protecting callers. It needs carefully chosen thresholds and a controlled half-open probe.

### 394. What is a timeout budget?

A timeout budget allocates an overall deadline across an operation and its downstream calls. Nested timeouts should fit within the caller's deadline and leave time for cleanup or response handling.

### 395. What is a bulkhead?

A bulkhead isolates resources or concurrency for different workloads so one failure or overload does not consume all capacity. Separate pools, limits, or queues can provide isolation.

### 396. What is hedging in resilient requests?

Hedging starts an additional request after a delay to reduce tail latency and accepts one successful result. It can amplify load and should be limited to safe, idempotent operations.

### 397. What is a fallback response?

A fallback provides a predefined or degraded result when a dependency fails. It must be semantically safe and clearly distinguish stale or incomplete data where that matters.

### 398. What is a timeout versus cancellation?

A timeout is a policy that limits operation duration; cancellation is the cooperative signal used to stop work. A timeout often triggers cancellation, but code still needs to observe it.

### 399. What is a transient fault?

A transient fault is a temporary failure likely to resolve, such as a brief network interruption. Classification should be specific to the dependency and operation rather than treating every exception as retryable.

### 400. What is idempotency in resilient processing?

Idempotency makes duplicate execution produce an equivalent intended outcome, supporting safe retries and redelivery. Persist deduplication state where needed and account for concurrent duplicates.

### 401. What is the transactional outbox pattern?

The outbox stores a message in the same database transaction as business state, then a separate dispatcher publishes it. It avoids a dual-write gap but requires idempotent delivery and cleanup.

### 402. What is at-least-once message delivery?

At-least-once delivery ensures a message is delivered one or more times, so consumers must tolerate duplicates. It does not guarantee exactly-once side effects by itself.

### 403. What is a poison message?

A poison message repeatedly fails processing and can block progress or consume resources. Use bounded retry, dead-letter handling, diagnostics, and a safe replay process.

### 404. What is graceful degradation?

Graceful degradation preserves essential functionality when optional components fail or capacity is constrained. Define which features can be omitted and how users are informed.

### 405. What is chaos testing?

Chaos testing deliberately injects controlled failures to test system resilience assumptions. Start in safe environments, limit blast radius, and define rollback and success criteria.

### 406. What is a service-level objective (SLO)?

An SLO is a target for a measurable service indicator over a time window, such as request availability or latency. It guides reliability decisions and should reflect user experience.

### 407. What is an error budget?

An error budget is the allowed unreliability implied by an SLO over its evaluation window. Teams can use it to balance feature delivery and reliability work.

### 408. What is load testing?

Load testing measures behavior under expected or elevated traffic, including latency, throughput, and resource limits. Use production-like configuration and realistic request distributions.

### 409. What is soak testing?

Soak testing runs a sustained workload to find issues such as leaks, resource exhaustion, or gradual degradation. Observe the system long enough to reveal trends beyond short load tests.

### 410. How should tests handle time-dependent code?

Inject a clock abstraction or controllable time provider where practical so tests can advance time deterministically. Avoid sleeps that make tests slow and flaky.

## 18. Globalization and networking

### 411. What is globalization in .NET?

Globalization covers culture-sensitive formatting, comparison, calendars, and language-related behavior. Applications should distinguish user-facing culture choices from protocol and storage formats.

### 412. What is the difference between culture and UI culture?

`CurrentCulture` influences formatting and parsing; `CurrentUICulture` influences resource lookup for localized text. They can differ for a user or operation.

### 413. What is invariant culture?

Invariant culture provides culture-independent conventions for operations that should not vary by user locale. Use it for stable machine-readable formats, not for localized user presentation.

### 414. Why use ISO 8601 timestamps in APIs?

ISO 8601 provides a widely interoperable date/time representation, especially when including an explicit offset or UTC marker. Define whether values represent an instant or a local wall-clock time.

### 415. What is `DateTimeOffset`?

`DateTimeOffset` represents a date/time with an offset from UTC and is usually appropriate for recording an instant with its original offset. It does not preserve a time zone's future daylight-saving rules.

### 416. What is the difference between `DateTime` and `DateTimeOffset`?

`DateTime` carries a kind indicator but may be ambiguous without context; `DateTimeOffset` includes a numeric offset. Neither alone represents a named time zone's full rule set.

### 417. What is a time zone identifier?

A time zone identifier selects a rule set for converting between local times and UTC. Identifier conventions and available zones can differ across operating systems and globalization data sources.

### 418. What is ICU in .NET?

ICU is a common globalization library used by modern .NET on many platforms for culture and collation data. Runtime builds and configuration may use other modes, so verify deployment-specific behavior.

### 419. What is globalization-invariant mode?

Globalization-invariant mode reduces dependency on culture data by limiting globalization behavior. It can affect sorting, formatting, and culture-specific operations and must be tested against application requirements.

### 420. Why should identifiers usually use ordinal comparison?

Programmatic identifiers are generally culture-independent, so ordinal comparisons avoid locale-specific casing or sorting surprises. Use linguistic comparison only when the values are natural language.

### 421. What is Unicode normalization?

Unicode normalization maps equivalent text sequences into a selected canonical form. Apply consistently where matching or storage requires it, while respecting the original text when preservation matters.

### 422. What is a grapheme cluster?

A grapheme cluster is a user-perceived text character that may consist of multiple Unicode code points. String length in .NET counts UTF-16 code units, not necessarily characters as a user sees them.

### 423. What is UTF-8?

UTF-8 is a variable-width Unicode encoding commonly used for network and file formats. Modern .NET provides UTF-8-oriented APIs and spans that can reduce transcoding allocations.

### 424. What is a `Rune` in .NET?

`System.Text.Rune` represents a Unicode scalar value, independent of whether it occupies one or two UTF-16 code units. It is useful for correctly iterating Unicode scalar values.

### 425. What is `HttpClient`?

`HttpClient` sends HTTP requests and is designed to reuse underlying connections through handlers. Manage client/handler lifetime carefully to avoid socket exhaustion and stale DNS behavior.

### 426. Why use `IHttpClientFactory`?

`IHttpClientFactory` centralizes `HttpClient` configuration, handler pooling, logging, and named or typed clients. It helps manage handler lifetime and supports resilience integration.

### 427. What is a typed `HttpClient`?

A typed client wraps an `HttpClient` with service-specific operations and configuration. It provides a clear dependency boundary and is commonly registered through `IHttpClientFactory`.

### 428. What is DNS refresh behavior with `HttpClient`?

DNS is typically resolved when connections are established, so long-lived pooled connections can outlive DNS changes. Configure handler lifetime or connection lifetime to balance reuse and endpoint refresh.

### 429. What is `SocketsHttpHandler`?

`SocketsHttpHandler` is the primary managed HTTP transport handler used by modern .NET `HttpClient`. It supports connection pooling, proxy, TLS, and protocol configuration.

### 430. What is HTTP/2 multiplexing?

HTTP/2 can carry multiple concurrent streams over one connection, reducing connection overhead. Stream limits and server behavior still affect concurrency and latency.

### 431. What is HTTP/3?

HTTP/3 maps HTTP semantics onto QUIC over UDP, enabling features such as independent stream delivery. Availability and deployment depend on runtime, OS, server, client, and network support.

### 432. What is gRPC?

gRPC is a high-performance remote procedure call framework using Protocol Buffers and typically HTTP/2. It supports streaming and code generation, with browser and proxy considerations.

### 433. What is gRPC-Web?

gRPC-Web adapts gRPC for browser environments that cannot use the full native gRPC transport directly. It commonly requires compatible server or proxy support.

### 434. What is a WebSocket?

A WebSocket provides a long-lived, bidirectional connection upgraded from HTTP. Design for authentication, idle timeouts, reconnects, message limits, and horizontal scale.

### 435. What is SignalR?

SignalR provides high-level real-time communication abstractions for .NET clients and servers, selecting transports such as WebSockets when available. Multi-instance deployments may need a backplane or managed service.

### 436. What is a network socket timeout?

A socket timeout limits how long an operation may wait for network progress. Distinguish connect, read, write, and overall request deadlines and propagate cancellation appropriately.

### 437. What is TLS certificate pinning?

Certificate pinning restricts accepted server certificates or public keys beyond ordinary trust validation. It can prevent some interception but creates operational risk during certificate rotation; use only with a maintained strategy.

### 438. What is proxy configuration for outbound HTTP?

Proxy configuration determines how outbound HTTP requests are routed through a proxy. Configure proxy credentials and bypass rules securely and account for deployment-specific environment settings.

### 439. What is socket exhaustion?

Socket exhaustion occurs when connections or ephemeral ports are consumed faster than they can be reused or released. Reuse `HttpClient` handlers, dispose responses, and diagnose connection churn.

### 440. What is network egress control?

Egress control restricts outbound destinations from an application or environment. It limits SSRF and data-exfiltration impact but must allow required dependency endpoints.

## 19. Advanced platform scenarios

### 441. What is a custom `SynchronizationContext`?

A `SynchronizationContext` defines how asynchronous continuations are scheduled for an environment. ASP.NET Core normally does not provide the classic request synchronization context used by older ASP.NET.

### 442. What does `ConfigureAwait(false)` do?

It tells an awaiter not to require resuming on the captured synchronization context when one exists. In ASP.NET Core there is normally no custom request context, so its effect is often less important than in classic UI or ASP.NET applications.

### 443. What is an async iterator?

An async iterator yields values over time with `IAsyncEnumerable<T>` and supports asynchronous production and cancellation. Consumers should enumerate with `await foreach` and pass cancellation where supported.

### 444. What is `IAsyncEnumerable<T>` useful for in an API?

It can represent asynchronous streaming results, reducing buffering for long or large responses. Verify serializer and hosting support and define what happens if a stream fails after response headers are sent.

### 445. What is a `Channel<T>` useful for in a worker?

A channel coordinates asynchronous producers and consumers, optionally with bounded capacity. It supports in-process work flow but is not durable across process restarts.

### 446. What is a durable message queue?

A durable queue persists messages outside the process so they can survive restarts and be processed asynchronously. Delivery, ordering, duplication, and retention guarantees depend on the broker.

### 447. What is a distributed lock?

A distributed lock coordinates access among processes using a shared system. Correctness depends on leases, expiry, fencing, and failure semantics; a local `lock` cannot coordinate machines.

### 448. What is a fencing token?

A fencing token is a monotonically increasing value that lets a protected resource reject operations from an expired lock holder. It helps prevent stale owners from acting after lease loss.

### 449. What is a lease?

A lease grants time-limited ownership that must be renewed to remain valid. Systems should assume the lease can expire during pauses or network partitions and enforce ownership at the resource.

### 450. What is eventual consistency?

Eventual consistency means replicas or derived views may temporarily disagree but are expected to converge under stated conditions. Applications must define acceptable staleness and conflict behavior.

### 451. What is a saga?

A saga coordinates a multi-step business transaction across services using local transactions and compensating actions. Compensation is not necessarily a perfect rollback and failures must be recoverable.

### 452. What is a distributed transaction?

A distributed transaction coordinates atomic work across multiple resource managers, often with substantial operational complexity. Many architectures prefer outbox/saga patterns and explicit eventual consistency.

### 453. What is a feature flag?

A feature flag conditionally enables behavior without necessarily deploying new code. Protect flag administration, define defaults and cleanup, and avoid uncontrolled combinations.

### 454. What is a circuit-breaker half-open state?

In half-open state, a breaker allows limited probe calls to test recovery. Successful probes can close it; failures reopen it according to the policy.

### 455. What is graceful degradation in a microservice?

It preserves core operations when optional remote capabilities are unavailable, for example by disabling a noncritical enrichment. Never silently weaken authorization or data correctness.

### 456. What is an idempotent consumer?

An idempotent consumer records or derives processed-message identity so duplicate delivery does not repeat side effects. Ensure deduplication state and business updates are coordinated atomically when possible.

### 457. What is a poison-pill message pattern?

A poison pill is a deliberate message used to signal a worker to stop or exit its processing loop. It differs from an accidental poison message and must be handled by the application's shutdown protocol.

### 458. What is a process-wide unhandled exception?

It is an exception that escapes normal handling boundaries and may terminate the process. Global handlers are useful for last-resort logging, not for safely continuing after arbitrary corruption.

### 459. What is fail-fast behavior?

Fail-fast behavior terminates an application when it cannot operate safely, rather than continuing in an invalid state. A supervisor can restart it, while health checks and logs explain the cause.

### 460. What is an application domain in modern .NET?

Modern .NET uses `AssemblyLoadContext` for many assembly isolation scenarios; it does not support creating multiple unloadable application domains like .NET Framework. Process isolation may be needed for stronger boundaries.

### 461. What is process isolation?

Process isolation separates components into operating-system processes, creating stronger fault and security boundaries than in-process load contexts. It adds IPC, deployment, and operational complexity.

### 462. What is a source-generated regular expression?

The `[GeneratedRegex]` attribute enables compile-time generation of a regex implementation in supported .NET versions. It can improve startup or performance for static patterns and provides build-time diagnostics.

### 463. What is a frozen collection?

A frozen collection is an immutable collection optimized for lookup after construction, available in modern .NET. It suits data built once and queried often, but has construction cost and cannot be mutated.

### 464. What is a `SearchValues<T>` type?

`SearchValues<T>` represents a precomputed set optimized for repeated search operations in supported modern .NET APIs. It can improve repeated character or byte searches when profiling shows the need.

### 465. What are hardware intrinsics?

Hardware intrinsics expose processor-specific instructions through .NET APIs, often for vector or cryptographic operations. They require feature checks or fallbacks for portability across CPUs.

### 466. What is `Vector<T>`?

`Vector<T>` provides SIMD operations over multiple values where supported by the runtime and hardware. The actual vector width is platform-dependent.

### 467. What is a source-generated P/Invoke?

`LibraryImport` can generate interop marshalling code at compile time for supported declarations. It can reduce runtime marshalling overhead and improve AOT compatibility.

### 468. What is custom marshalling?

Custom marshalling converts data between managed representations and native ABI representations. It must define ownership, layout, encoding, and error behavior precisely.

### 469. What is ABI compatibility?

Application binary interface compatibility concerns calling conventions, symbol names, data layout, and binary contracts between compiled components. Managed source compatibility does not guarantee native ABI compatibility.

### 470. What is a `SafeBuffer`?

`SafeBuffer` is a safe-handle abstraction for unmanaged memory regions with bounds-aware access helpers. It helps manage native memory ownership but still requires correct allocation size and lifetime.

### 471. What is memory-mapped file access?

Memory-mapped files expose file contents through virtual memory, enabling efficient access to large files or shared regions. Synchronization, file lifetime, and platform constraints still apply.

### 472. What is `ArrayPool<T>`?

`ArrayPool<T>` rents and returns arrays to reduce repeated allocation pressure. Returned arrays may contain old data, so clear sensitive content when necessary and do not use after returning.

### 473. What is `ObjectPool<T>`?

An object pool reuses objects to reduce construction or allocation cost. Pooling adds reset and concurrency complexity and can retain more memory, so benchmark before adopting it.

### 474. What is a `ref struct`?

A `ref struct` is a stack-constrained type that cannot be boxed or stored in ordinary heap locations. This enables safe APIs over stack-bound memory such as spans.

### 475. What is a ref-like type restriction across `await`?

Stack-bound ref-like values cannot generally live across an asynchronous suspension because their storage lifetime is not guaranteed. Newer language/runtime capabilities evolve these rules, so follow the compiler's target-specific constraints.

### 476. What is a required member in C#?

The `required` modifier requires callers to initialize specified members during object creation at compile time. It is a compile-time contract and does not independently validate deserialized or runtime-created objects.

### 477. What is a record type?

A record provides value-oriented equality and concise data modeling, with record classes or record structs. Its equality and mutability semantics should match the domain's intended identity.

### 478. What is a primary constructor in modern C#?

Primary constructors allow constructor parameters in the type declaration for supported class, struct, and record forms. Language version and target SDK determine availability; parameters are not automatically properties for ordinary classes.

### 479. What is an interceptors feature in C#?

Interceptors are an evolving compiler feature intended to redirect certain call sites at compile time and have version-specific status and constraints. Check the current language documentation before relying on them in production.

### 480. What is a .NET workload?

A workload is an installable set of SDK components for additional development scenarios, such as mobile or WebAssembly. Workload manifests and installation state are managed by the SDK tooling.

### 481. What is a workload manifest?

A workload manifest describes workload packs and versions recognized by the SDK. It enables SDK tooling to resolve the components required for a particular workload.

### 482. What is .NET MAUI?

.NET MAUI is a .NET UI framework for building native apps across supported platforms from shared code. Its platform support, workloads, and lifecycle are version-specific.

### 483. What is Blazor WebAssembly?

Blazor WebAssembly runs .NET code in a browser through WebAssembly and a browser-hosted runtime. Download size, browser sandbox constraints, and client-side trust must be considered.

### 484. What is Blazor Server?

Blazor Server runs component logic on the server and communicates UI updates over a persistent connection, typically SignalR. It requires connection state and careful scaling and latency planning.

### 485. What is WebAssembly support in .NET?

.NET can target WebAssembly for browser and server-side scenarios, with platform capabilities and restrictions that differ from native execution. Validate supported APIs, threading, AOT, and deployment requirements for the chosen runtime.

### 486. What is a .NET worker service?

A worker service is a host-based application commonly used for background processing. It can run as a Windows service, systemd service, container, or cloud worker with appropriate hosting integration.

### 487. What is Windows Service integration for .NET?

Windows Service hosting lets a .NET worker integrate with Windows service lifecycle and notifications. It is Windows-specific and should be configured alongside correct service identity and recovery settings.

### 488. What is systemd integration for .NET?

Systemd integration lets a hosted .NET process participate in Linux service management and lifecycle. Configure the unit's restart, identity, environment, and shutdown behavior deliberately.

### 489. What is a single-file deployment?

A single-file publish bundles application files into a single executable-oriented artifact, subject to extraction and native-library behavior. It is a packaging option, not necessarily a single native binary or Native AOT.

### 490. What is a bundle extraction directory?

Some single-file deployments extract native or other files to disk at runtime. Configure and secure extraction paths appropriately, or choose compatible options that avoid extraction when supported.

### 491. What is composite Native AOT?

Composite Native AOT combines multiple assemblies into a native compilation unit in supported toolchains to affect output size and performance. Support and tradeoffs depend on SDK version and target platform.

### 492. What is a framework-dependent container?

A framework-dependent container relies on a compatible shared runtime in its base image. Keep application and base runtime versions compatible and update the base image for security servicing.

### 493. What is a self-contained container?

A self-contained container includes the app's runtime in its published output, while the image still depends on an operating-system base. Runtime updates require republishing and rebuilding the image.

### 494. What is cross-compilation for .NET?

Cross-compilation builds output for a target different from the build host, subject to SDK, runtime pack, native toolchain, and platform support. Native AOT often requires a target-compatible compiler toolchain.

### 495. What is a runtime pack?

A runtime pack contains runtime-specific files used by SDK restore and publish for a target framework and RID. The SDK selects compatible packs based on project target and deployment options.

### 496. What is a targeting pack?

A targeting pack contains reference assemblies and metadata used to compile against a framework without supplying its runtime implementation. The SDK resolves the pack for the chosen target framework.

### 497. What is API compatibility analysis?

API compatibility analysis compares public API surfaces across versions to identify potential breaking changes. It can be integrated into library builds and release review.

### 498. What is a compatibility switch in .NET?

A compatibility switch or runtime configuration setting can alter selected runtime behavior to ease migration or tune execution. Use only documented settings and test their effect on the exact runtime version.

### 499. How should a .NET platform upgrade be approached?

Review support dates and breaking changes, update the SDK and dependencies, build and test every target, then validate deployment and performance in a representative environment. Roll out gradually with observable rollback criteria.

### 500. How do you choose among JIT, ReadyToRun, and Native AOT?

Choose JIT for broad runtime flexibility, ReadyToRun when startup improvement justifies larger output, and Native AOT for supported workloads that benefit from native startup or deployment characteristics. Benchmark the published application and verify library, reflection, and platform compatibility.
