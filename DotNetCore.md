# .NET Core / .NET Interview Questions and Answers

> “.NET Core” refers to the cross-platform .NET Core line through .NET Core 3.1. From .NET 5 onward the platform is named .NET. This guide covers the shared modern platform and calls out version differences where useful.

## Contents

1. [Platform and project structure](#1-platform-and-project-structure)
2. [Runtime and application configuration](#2-runtime-and-application-configuration)
3. [Dependency injection and hosting](#3-dependency-injection-and-hosting)
4. [Web APIs and middleware](#4-web-apis-and-middleware)
5. [Configuration, logging, and security](#5-configuration-logging-and-security)
6. [Performance, deployment, and testing](#6-performance-deployment-and-testing)

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
