# Entity Framework Interview Questions and Answers

> This guide focuses mainly on Entity Framework Core (EF Core), the modern cross-platform ORM. EF6 is a separate, older product with different capabilities; version-specific behavior should be checked against the version used by the application.

## Contents

1. [ORM and EF fundamentals](#1-orm-and-ef-fundamentals)
2. [Modeling and relationships](#2-modeling-and-relationships)
3. [Queries and loading](#3-queries-and-loading)
4. [Changes and transactions](#4-changes-and-transactions)
5. [Migrations and production use](#5-migrations-and-production-use)
6. [Performance, testing, and reliability](#6-performance-testing-and-reliability)

## 1. ORM and EF fundamentals

### 1. What is Entity Framework?

Entity Framework is an object-relational mapper (ORM) for .NET. It maps .NET entities and relationships to relational database structures and translates supported LINQ queries into database commands. It can reduce repetitive data-access code but does not remove the need to understand SQL, transactions, indexes, or database behavior.

### 2. What is the difference between EF Core and EF6?

EF Core is the actively developed, cross-platform successor with a redesigned codebase, provider ecosystem, and evolving features. EF6 is the earlier framework, historically associated with .NET Framework; EF6 also has a newer cross-platform variant, but its feature set and APIs differ. Do not assume features or behaviors are identical—confirm against the exact version and provider.

### 3. What is a `DbContext`?

`DbContext` represents a unit of work and identity map. It manages entity tracking, query execution, and persistence through `SaveChanges`/`SaveChangesAsync`. It is designed for a short scope, commonly one web request or business operation, and is not thread-safe.

### 4. What is a `DbSet<TEntity>`?

`DbSet<TEntity>` represents a query and persistence entry point for a set of entity instances, typically mapped to a table or view. It supports querying and operations such as `Add` and `Remove`. The actual database command is normally executed when results are materialized or changes are saved.

### 5. What is change tracking?

EF Core tracks entity states and property changes for entities loaded by a tracking query or attached to a context. On save, it generates insert, update, or delete commands based on tracked state. Tracking supports identity resolution and updates but consumes memory and adds overhead.

### 6. What are entity states?

Common states are `Added`, `Unchanged`, `Modified`, and `Deleted`; `Detached` entities are not tracked. `SaveChanges` maps state to database operations. State can be set explicitly, but manually setting it incorrectly may cause unintended writes.

### 7. What is the identity map?

Within a context, EF Core typically tracks one entity instance for a given entity type and key. A repeated tracking query resolves to the already tracked instance, supporting consistent updates. This behavior is scoped to the context and does not guarantee freshness relative to concurrent database changes.

### 8. Why should `DbContext` not be shared across threads?

`DbContext` and its change tracker are not thread-safe. Concurrent use can corrupt tracking state or throw exceptions. Use one context per unit of work and await each operation before reusing it; create separate contexts for genuinely independent concurrent operations.

## 2. Modeling and relationships

### 9. What is a POCO entity?

A POCO (“plain old CLR object”) is a regular .NET class with no required base class. EF Core can map it using conventions, attributes, or Fluent API. Entities should expose domain state and behavior appropriately; public setters are not mandatory for all mappings.

### 10. What is convention-based mapping?

Conventions infer mappings from entity names, property types, key naming patterns, and navigation properties. Conventions reduce configuration but may not express domain-specific choices clearly. Explicit configuration overrides conventions.

### 11. What is the Fluent API?

The Fluent API configures a model through `OnModelCreating` or `IEntityTypeConfiguration<TEntity>`. It can define keys, column names, indexes, constraints, relationships, conversions, and delete behavior. Centralized explicit mapping is often preferable for complex schemas.

### 12. What are data annotations?

Data annotations are attributes such as `[Key]`, `[Required]`, and `[MaxLength]` that express some mapping or validation metadata. They are convenient for simple cases, but do not cover every model configuration. Fluent API takes precedence in many configuration conflicts and can keep persistence concerns outside entity classes.

### 13. How are primary keys configured?

EF Core recognizes common key names such as `Id` or `<EntityName>Id`, or a key can be specified with attributes or Fluent API. Composite keys require explicit configuration in many cases. Keys should be stable identifiers and align with the database schema.

### 14. How do one-to-many relationships work?

A dependent entity holds a foreign key referencing a principal entity, and navigation properties can represent the relationship in the object model:

```csharp
public class Blog
{
    public int Id { get; set; }
    public List<Post> Posts { get; } = [];
}

public class Post
{
    public int Id { get; set; }
    public int BlogId { get; set; }
    public Blog Blog { get; set; } = null!;
}
```

EF Core can infer many relationships by convention, but explicit configuration clarifies optionality and delete behavior.

### 15. What are navigation properties and foreign keys?

A foreign-key property stores the relational key value. A navigation property references the related object or collection in the object model. They may be configured as paired properties; shadow foreign keys can be used when no CLR property exists. Explicit foreign-key properties often make state and serialization behavior easier to understand.

### 16. What is a many-to-many relationship?

A many-to-many relationship associates multiple rows on each side. EF Core can map an implicit join entity/table for suitable cases or use an explicit join entity when the association has its own attributes, behavior, or lifecycle. Schema and provider requirements affect the resulting mapping.

### 17. What is an owned entity type?

An owned entity type represents a value-like object whose lifecycle is tied to its owner. It is configured as part of the owner's model and may map to the same table or a separate table depending on configuration. Owned mapping semantics and capabilities have evolved across EF Core versions; verify the target version.

### 18. What are shadow properties?

Shadow properties exist in the EF model and change tracker but not as CLR properties on the entity class. They can represent foreign keys or store metadata without adding persistence-specific members to the class. Access them through EF APIs and model configuration.

## 3. Queries and loading

### 19. How does LINQ querying work in EF Core?

EF Core receives an expression tree from `IQueryable<T>`, translates supported operations for the configured provider, sends database commands, and materializes results. Not every .NET method can be translated. Translation and SQL differ by provider, so inspect generated SQL and test against the real provider for important queries.

### 20. What is eager loading?

Eager loading retrieves related entities as part of a query using `Include` and `ThenInclude`. Depending on provider and query splitting configuration, this can produce joins or multiple commands. Eager loading can prevent later queries but may return excessive or duplicated data if used indiscriminately.

### 21. What is lazy loading?

Lazy loading fetches a navigation when it is accessed, typically using proxies or another configured mechanism. It can make database access implicit and cause N+1 query problems. It requires compatible entity/proxy configuration and is not enabled by default in many setups. Explicitly inspect query counts and use only when the behavior is understood.

### 22. What is explicit loading?

Explicit loading fetches a navigation deliberately through the context's entry API after an entity has been loaded. It makes the additional database operation visible in code and can be appropriate when related data is conditionally needed.

### 23. What is the N+1 query problem?

N+1 occurs when one query loads a set of parent entities and then a separate query is issued for each parent’s related data. It often results from lazy loading or loops that query per item. Use a suitable projection, eager load, explicit batch query, or redesign to reduce round trips; check the actual generated commands.

### 24. What is a tracking query versus `AsNoTracking`?

A tracking query stores materialized entities in the context change tracker so modifications can be saved. `AsNoTracking` avoids tracking for read-only results, often reducing memory and CPU cost. `AsNoTrackingWithIdentityResolution` can deduplicate entity instances during one query without keeping them tracked afterward; availability and behavior are version-dependent.

### 25. What is client evaluation?

Client evaluation means part of a query is evaluated in application memory rather than translated to SQL. Modern EF Core generally throws when a non-translatable expression occurs in most query portions, while allowing some client-side projection evaluation. Materializing early can cause large data transfers; keep filters and projections translatable where possible.

### 26. Why use projection with `Select`?

Projection selects only required columns and shapes results into a DTO or anonymous type. It can reduce database I/O, network traffic, and materialization overhead compared with loading full entities. Projecting is especially useful for read APIs where change tracking is unnecessary.

### 27. What is split-query behavior?

A split query executes multiple SQL queries for included collections rather than one large join, which can reduce row multiplication and huge result sets. It adds round trips and may observe inconsistent data if concurrent changes occur between commands unless transaction isolation provides the needed consistency. Compare both forms for the actual workload.

### 28. How do you inspect SQL generated by EF Core?

Use logging, diagnostic events, query tags, or `ToQueryString()` where supported. Generated SQL is provider- and version-specific. Inspect parameterization, selected columns, joins, predicates, and execution plans; avoid logging sensitive parameter values in production.

## 4. Changes and transactions

### 29. What does `SaveChanges` do?

`SaveChanges` detects changes (unless configured otherwise), orders and executes database commands for tracked entities, and updates tracking state after success. `SaveChangesAsync` is the asynchronous counterpart. Use async APIs in asynchronous application paths, and treat a failed save as a failed unit of work requiring appropriate error handling.

### 30. Does `SaveChanges` use a transaction?

For relational providers, EF Core generally wraps a single `SaveChanges` call in a transaction when needed so the batch is atomic. Behavior can vary by provider and configuration. Multiple saves or external operations are not automatically one encompassing transaction.

### 31. How do explicit transactions work?

An explicit transaction coordinates multiple database operations:

```csharp
await using var transaction =
    await context.Database.BeginTransactionAsync(cancellationToken);

// Perform related database work.
await context.SaveChangesAsync(cancellationToken);
await transaction.CommitAsync(cancellationToken);
```

Dispose/rollback on failure, keep transactions short, and be careful when combining with retrying execution strategies; use the provider's prescribed execution-strategy pattern.

### 32. What is optimistic concurrency?

Optimistic concurrency assumes conflicts are uncommon and detects them at save time using a concurrency token, such as a row version. If the database row changed since it was read, EF can throw `DbUpdateConcurrencyException`. The application must decide whether to reload, merge, retry, or report a conflict.

### 33. What is a concurrency token?

A concurrency token is a property included in update/delete predicates so EF can detect that the persisted value changed. SQL Server `rowversion` is a common provider-specific token; other databases may use another mechanism or an application-managed token. It is not necessarily a timestamp representing wall-clock time.

### 34. How do you update a disconnected entity?

Disconnected entities arrive outside the context that originally loaded them, such as from an API request. Attach or query the existing entity, validate allowed changes, and mark only intended properties as modified. Blindly calling `Update` on a client-supplied graph may mark many values modified and create security or data integrity issues.

### 35. What is `ExecuteUpdate`/`ExecuteDelete`?

Recent EF Core versions provide set-based operations that update or delete rows directly in the database without loading each entity into the change tracker. They can be efficient for bulk changes but bypass ordinary per-entity tracking and some application-side behavior. Availability, supported expressions, and provider behavior depend on EF Core version.

### 36. How should duplicate inserts or retries be handled?

Design writes to be idempotent where retries are possible, use database constraints to enforce uniqueness, and handle provider-specific unique-constraint failures intentionally. A retry may repeat a command whose first response was lost after commit, so blindly replaying non-idempotent work can duplicate effects.

## 5. Migrations and production use

### 37. What are EF Core migrations?

Migrations are versioned schema changes generated from model differences and applied in sequence. They capture schema evolution, not necessarily every desired data transformation or operational rollout. Review generated migration code before applying it, especially destructive changes and production data migrations.

### 38. How are migrations created and applied?

Typical CLI commands are `dotnet ef migrations add Name` and `dotnet ef database update`; these require the EF tooling and appropriate design-time configuration. In production, many teams generate reviewed SQL scripts or migration bundles and apply them through controlled deployment rather than having every application instance race to change the schema at startup.

### 39. What is a design-time `DbContext` factory?

`IDesignTimeDbContextFactory<TContext>` tells EF tooling how to create a context for design-time operations such as migrations. It can make tooling independent of application startup, but configuration must match intended environments without embedding secrets.

### 40. What is the model snapshot?

The model snapshot is generated metadata describing the model at the last migration. EF uses it to determine differences when creating a new migration. Do not manually edit it except as part of a deliberate, well-understood repair.

### 41. How do you seed data?

Small, stable reference data can be configured through model seeding where appropriate, but seed operations and update behavior depend on EF Core version. Large, environment-specific, or evolving data is usually better handled by explicit deployment scripts or application code. Never seed secrets or assume a model migration is a complete data-management strategy.

### 42. How should `DbContext` be registered in ASP.NET Core?

`AddDbContext<TContext>` commonly registers a scoped context, aligning its lifetime with a web request. Configure provider and connection through secure configuration. Do not inject a scoped context into a singleton, and avoid sharing one context across concurrent operations.

## 6. Performance, testing, and reliability

### 43. How can EF Core query performance be improved?

Project only needed columns, avoid N+1 loading, use no-tracking reads where appropriate, paginate large results, create useful database indexes, and inspect generated SQL and execution plans. Avoid premature optimization; database query design and round trips usually matter more than ORM micro-costs.

### 44. What is context pooling?

Context pooling reuses initialized `DbContext` instances to reduce setup allocations. It is not the same as database connection pooling. A pooled context must not retain request-specific state between uses; configure mutable state carefully and follow EF Core's pooling guidance.

### 45. What is compiled-query support?

Compiled queries can reduce repeated LINQ translation overhead for hot, stable query shapes. They do not make a slow SQL query fast and may add complexity. Profile first, and ensure the query is used enough to justify the optimization.

### 46. How do database indexes affect EF Core?

EF can configure indexes in the model and migrations, but the database engine determines index behavior and query plans. Index columns should match filters, joins, and ordering patterns while considering write costs and storage. Validate with the database's execution-plan tools.

### 47. How should EF Core be tested?

Test domain logic independently where possible. For persistence behavior, prefer integration tests against the same database provider or a close equivalent. EF Core's InMemory provider is not relational and does not reproduce SQL constraints, transactions, or translation behavior; SQLite in-memory can be useful but still differs from production databases.

### 48. Why are repository and unit-of-work wrappers sometimes unnecessary?

`DbContext` already provides unit-of-work and repository-like behavior. An extra generic repository can hide useful query capabilities or duplicate APIs without adding value. Add a domain-specific abstraction when it enforces meaningful boundaries, encapsulates complex queries, or makes the application architecture clearer.

### 49. What are global query filters?

Global query filters add model-level predicates automatically, commonly for soft deletion or tenant scoping. They improve consistency but can hide data from queries unexpectedly. Ensure privileged operations use filters intentionally, and do not rely on a client-controlled tenant identifier for security.

### 50. How do you protect against over-posting?

Do not bind an incoming request directly to a persistence entity with fields the caller should not control. Use request DTOs, validate ownership and authorization, and update an allowlisted set of entity properties. EF's tracking convenience does not replace authorization or input-boundary design.
