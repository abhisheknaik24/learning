# Entity Framework Interview Questions and Answers

> This guide focuses mainly on Entity Framework Core (EF Core), the modern cross-platform ORM. EF6 is a separate, older product with different capabilities; version-specific behavior should be checked against the version used by the application.

## Contents

1. [ORM and EF fundamentals](#1-orm-and-ef-fundamentals)
2. [Modeling and relationships](#2-modeling-and-relationships)
3. [Queries and loading](#3-queries-and-loading)
4. [Changes and transactions](#4-changes-and-transactions)
5. [Migrations and production use](#5-migrations-and-production-use)
6. [Performance, testing, and reliability](#6-performance-testing-and-reliability)
7. [Provider, version, and configuration details](#7-provider-version-and-configuration-details)
8. [Advanced model design](#8-advanced-model-design)
9. [Relationships and inheritance](#9-relationships-and-inheritance)
10. [Query composition and translation](#10-query-composition-and-translation)
11. [Materialization and tracking details](#11-materialization-and-tracking-details)
12. [Persistence, transactions, and reliability](#12-persistence-transactions-and-reliability)
13. [Schema evolution and deployment](#13-schema-evolution-and-deployment)
14. [Performance, diagnostics, and testing](#14-performance-diagnostics-and-testing)
15. [Security, tenancy, and advanced scenarios](#15-security-tenancy-and-advanced-scenarios)

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

## 7. Provider, version, and configuration details

### 51. How is EF Core versioned relative to .NET releases?

EF Core ships on its own cadence; verify compatibility among the runtime, provider, tooling, and target framework.

### 52. What determines the EF Core version an application runs?

The referenced EF Core packages determine runtime APIs; resolved transitive versions and provider compatibility also matter.

### 53. Why should EF Core packages use aligned versions?

Mismatched runtime, provider, and design packages can cause missing APIs or runtime incompatibility.

### 54. How does EF6 differ from EF Core for feature planning?

They are separate products with distinct APIs, providers, and capabilities; check product-specific documentation before choosing.

### 55. What is an EF Core database provider?

A provider implements database-specific query translation, type mapping, SQL generation, and connection integration.

### 56. Why can the same LINQ query behave differently across providers?

Providers support different SQL features, functions, types, and translations, so portability must be tested explicitly.

### 57. How do you select a database provider?

Install its EF provider package and configure `UseSqlServer`, `UseNpgsql`, `UseSqlite`, or its equivalent.

### 58. What is the EF Core design package used for?

`Microsoft.EntityFrameworkCore.Design` supplies services used by tools at design time, such as migration scaffolding.

### 59. What is the `dotnet-ef` tool?

It is a CLI tool for migration and database tasks; install or manifest-pin a compatible tool version.

### 60. How can a repository pin EF CLI tooling?

Use a local .NET tool manifest so contributors and builds invoke a consistent tool version.

### 61. What does `OnConfiguring` do?

It supplies context options, often as a fallback; application-level dependency-injection configuration is generally clearer.

### 62. What does `DbContextOptions<TContext>` contain?

Provider selection and options such as logging, retry behavior, and connection configuration.

### 63. What is `AddDbContextFactory` useful for?

It creates contexts on demand, including from background services that cannot hold scoped contexts.

### 64. When is `IDbContextFactory<TContext>` preferable to a scoped context?

Use it for short independent operations, background work, or UI components with lifetimes beyond a request scope.

### 65. How can a relational command timeout be configured?

Set it through provider options or `Database.SetCommandTimeout`, choosing a value appropriate to the operation.

### 66. How does context pooling differ from connection pooling?

Context pooling reuses EF context objects; connection pooling is generally managed by the underlying database driver.

### 67. How should connection strings be supplied?

Load them from secure environment-specific configuration or a secret store, not source control.

### 68. What is a provider capability check?

It verifies that the chosen provider and version support a feature before the application depends on it.

### 69. How do you verify a provider's SQL translation support?

Consult provider documentation and integration-test the query against that database.

### 70. What does `UseLazyLoadingProxies` enable?

It configures proxy-based lazy loading when the proxy package and entity requirements are satisfied.

### 71. What is a keyless entity type?

A query model without a primary key, often mapped to a view or SQL result; it is not tracked for updates.

### 72. How does EF Core support raw SQL queries?

Use APIs such as `FromSql` for entity queries and supported SQL-query APIs for scalar or unmapped results.

### 73. How does EF Core resolve provider services?

The provider configured in context options supplies services for model building, translation, and database commands.

### 74. What is a model cache key?

EF caches a model by context type and relevant options; legitimate model variation may require a custom cache key.

### 75. When is `IModelCacheKeyFactory` needed?

Use it when model shape differs by context instance, such as schema-per-tenant mapping.

### 76. Why is changing model shape per request risky?

It defeats model caching and can create correctness or memory issues; prefer stable model shape with parameterized queries.

### 77. What is compiled model support?

Generated model metadata can reduce startup model-building cost for large models in supported EF Core versions.

### 78. When should a compiled model be considered?

Profile model initialization first; it helps startup-heavy applications with sufficiently large models.

### 79. What does `EnsureCreated` do?

It creates a schema directly from the current model without migration history or incremental schema evolution.

### 80. Why should `EnsureCreated` not be mixed casually with migrations?

Its schema lifecycle bypasses migration history and can prevent normal migration-based upgrades.

### 81. What is a provider-specific extension method?

An API available only when a provider package is referenced, such as provider-specific configuration or functions.

### 82. How should EF provider upgrades be planned?

Review release notes, compatibility, generated SQL, migration changes, and regression tests.

### 83. What is `ConfigureWarnings` for?

It changes handling of selected EF diagnostics, including promoting warnings to errors or suppressing known noise.

### 84. Why should EF warnings not be suppressed indiscriminately?

Warnings may reveal unsafe query patterns, model mistakes, or behavior changes that need correction.

### 85. What is an EF Core interceptor?

A hook that observes or modifies selected database operations, commands, connections, or save events.

### 86. When are interceptors appropriate?

Use them for cross-cutting database behavior; avoid hiding essential business logic inside interception.

### 87. How do interceptors differ from logging providers?

Interceptors can participate in or alter operations; logging reports events without being the same interception contract.

### 88. Why accept `DbContextOptions<TContext>` in a context constructor?

It lets dependency injection configure the provider and options while keeping the context reusable.

### 89. Can one context type support multiple providers?

It can be configured with different providers, but migrations and model behavior need deliberate separation.

### 90. What does `UseModel` configure?

It supplies a prebuilt model, commonly a generated compiled model, instead of building metadata at runtime.

### 91. What does `EnableDetailedErrors` do?

It improves diagnostics for certain materialization errors, with potential overhead and no substitute for validation.

### 92. What does `EnableSensitiveDataLogging` expose?

It may include parameter values in logs; enable only in controlled development environments due to data exposure risk.

### 93. What is EF Core's execution strategy?

A provider-configured strategy can retry transient database failures for supported operations.

### 94. Why must retries be considered with explicit transactions?

The entire transactional unit may need execution-strategy coordination so retries replay it consistently.

### 95. How can EF Core API availability be checked?

Check API documentation and release notes for the exact package version targeted by the application.

### 96. What is a relational type mapping?

It maps a CLR property type to a provider-specific store type, conversion, and database facets.

### 97. How can provider package versions be inspected?

Review the project dependency graph or lock file to confirm resolved and transitive versions.

### 98. What is a context lifetime boundary?

It scopes identity resolution, tracked changes, and transaction coordination to one intended unit of work.

### 99. How can EF Core be used in a console application?

Build options explicitly or use dependency injection, then create and dispose a context for each unit of work.

### 100. How is a default database schema configured?

Use provider-supported model configuration such as `HasDefaultSchema`; confirm the resulting schema in migrations.

## 8. Advanced model design

### 101. What is a value converter?

A converter transforms a model property value to and from a provider value, such as an enum and string.

### 102. When should a value converter be used?

Use it when the domain representation differs from the database representation and the conversion is deterministic.

### 103. What are value comparers?

Comparers define equality, hashing, and snapshots for values whose change detection cannot use ordinary CLR equality.

### 104. When is a custom value comparer necessary?

It may be needed for mutable converted values, such as arrays or collections, so EF can detect in-place changes.

### 105. What does a backing field enable?

It lets EF read or write persisted state without requiring a public property setter.

### 106. How can a navigation use a backing field?

Configure field access so EF can populate the navigation while the domain exposes a controlled collection interface.

### 107. What is field-only property mapping?

EF maps a field without a corresponding CLR property, allowing persistence of state hidden from the public API.

### 108. What is a shadow property commonly used for?

It can hold a foreign key, discriminator, or other mapped state without adding a CLR member.

### 109. What is a property access mode?

It tells EF whether to use a property, backing field, or a preference between them for reads and writes.

### 110. What does `HasDefaultValue` configure?

It declares a database default for a column when an insert omits that value.

### 111. How does a database default differ from a CLR initializer?

A CLR initializer runs in application code; a database default runs when an insert omits the column.

### 112. What is a computed column?

It is a database expression whose value is generated by the database rather than supplied as an ordinary insert value.

### 113. What does `ValueGeneratedOnAddOrUpdate` indicate?

It tells EF that the store may generate or update a value; exact generation is provider- and mapping-dependent.

### 114. What are precision and scale?

For numeric store types, precision is total digits and scale is digits to the right of the decimal point.

### 115. Why configure string column length explicitly?

It constrains stored data and may affect schema, indexes, and provider-specific type selection.

### 116. What does Unicode configuration affect?

It controls whether a string column uses a Unicode-capable store type where the provider distinguishes the choice.

### 117. What is a check constraint?

It is a database rule that rejects rows whose values fail a Boolean expression.

### 118. Why use a unique index instead of application-only validation?

The database enforces uniqueness under concurrency, even when multiple application instances race to insert.

### 119. What is a filtered index?

It indexes only rows satisfying a predicate, where the database provider supports that feature.

### 120. What is an alternate key?

It is a unique model key other than the primary key and can serve as a relationship target.

### 121. When should a composite key be configured?

Use one when the domain's stable identity is naturally formed from multiple columns and the schema supports it.

### 122. What is a surrogate key?

It is a generated identifier with no business meaning, used as a stable database identity.

### 123. What is a natural key?

It is an identity derived from meaningful domain attributes, such as a globally unique code.

### 124. What is a table-splitting mapping?

Multiple entity types share one database row and table, typically linked by a shared key and required relationship.

### 125. What is entity splitting?

A single entity's properties are mapped across multiple tables joined by the entity key.

### 126. What is a table-per-hierarchy inheritance mapping?

All types in a hierarchy share one table, with a discriminator identifying each concrete type.

### 127. What is table-per-type inheritance mapping?

Each type has a table, and derived rows join to base rows by key; query cost and support vary by provider.

### 128. What is table-per-concrete-type mapping?

Each concrete type has its own table containing inherited properties, potentially duplicating schema columns.

### 129. How should inheritance strategy be selected?

Compare query patterns, constraints, schema clarity, provider support, and generated SQL for the target database.

### 130. What is a discriminator column?

It identifies the concrete entity type represented by a row in a shared-table inheritance mapping.

### 131. Can a discriminator be incomplete?

Configuration can allow rows with discriminator values not currently mapped to a CLR type, subject to EF version and mapping.

### 132. What is a complex type in EF Core?

It models structured, value-like properties without an independent entity identity; availability and capabilities depend on EF version.

### 133. How does a complex type differ from an owned entity?

Complex types have no entity identity or separate lifecycle, while owned types are entity types tied to an owner.

### 134. What is a JSON column mapping?

It stores structured values as JSON, with querying and update support dependent on EF Core version and provider.

### 135. Why should JSON mapping be tested against the target provider?

JSON operators, indexing, path translation, and partial updates differ across databases and provider versions.

### 136. What is a table-valued function mapping?

It maps a database function returning rows into a queryable model shape where provider support permits.

### 137. What is a scalar function mapping?

It maps a CLR method to a database function expression for translation in supported queries.

### 138. Why map a database function instead of invoking it in client code?

Mapping allows the server to evaluate it as part of a translatable query.

### 139. What is a query filter parameter?

It is a context-dependent value used by a global filter, such as a current tenant identifier.

### 140. How can tenant state reach a global query filter?

Expose the tenant value on the context instance and reference it in the configured filter expression.

### 141. What is a model convention?

A convention applies mapping configuration automatically based on discovered CLR types and model metadata.

### 142. When would you add a custom convention?

Use one to apply consistent model configuration across many properties or entity types without repetitive mappings.

### 143. What is a pre-convention model configuration?

It configures defaults for discovered properties before entity-specific mapping, where supported by the EF version.

### 144. What is `IEntityTypeConfiguration<TEntity>`?

It packages Fluent API configuration for one entity type and can be applied from the context model builder.

### 145. Why separate entity configurations from `OnModelCreating`?

It keeps a large model organized and makes each entity's persistence mapping easier to review.

### 146. What is a shared-type entity?

It is a model entity type represented by a shared CLR type, often a dictionary, and requires a distinct model name.

### 147. When might a property bag entity be useful?

It can represent dynamic or join data when a dedicated CLR class is unnecessary, with reduced compile-time safety.

### 148. What is an unmapped type query?

It materializes query results into a CLR type not registered as an entity, using APIs supported by the EF version/provider.

### 149. What is model validation?

EF validates mapping consistency while building the model and reports unsupported or contradictory configuration.

### 150. Why inspect EF's finalized model?

It confirms the effective mapping after conventions and configuration have been combined.

## 9. Relationships and inheritance

### 151. What is a principal entity in a relationship?

It is the referenced side whose key is targeted by a dependent's foreign key.

### 152. What is a dependent entity?

It is the side that contains the foreign key and depends on a principal for that relationship.

### 153. How is an optional relationship represented?

The dependent foreign key may be null, subject to CLR nullability and model configuration.

### 154. How is a required relationship represented?

The foreign key must have a value, and the database commonly enforces it with a non-nullable column.

### 155. What is a navigation fix-up?

EF synchronizes tracked references and collections when related entities with matching foreign keys are loaded or attached.

### 156. Does navigation fix-up load data from the database?

No; it connects entities already tracked by the same context and does not itself issue queries.

### 157. What is relationship change detection?

EF detects changes to foreign keys or navigations and updates relationship state before saving.

### 158. What is a skip navigation?

It is a navigation across a many-to-many relationship whose join entity is not exposed in the CLR model.

### 159. When should a many-to-many join entity be explicit?

Use one when the association has payload, its own behavior, or needs direct querying and lifecycle control.

### 160. How is a one-to-one relationship identified?

The dependent foreign key is unique, so at most one dependent row can reference a principal.

### 161. What is a shared primary-key relationship?

The dependent's primary key also serves as its foreign key to the principal.

### 162. What is a self-referencing relationship?

It relates rows of one entity type, such as an employee and manager or a node and parent node.

### 163. How can a self-referencing relationship be configured?

Configure the reference navigation, inverse collection, and foreign key explicitly to avoid ambiguity.

### 164. What is cascade delete?

Deleting a principal causes dependent rows to be deleted by EF or the database according to configured behavior.

### 165. What does `DeleteBehavior.Restrict` express?

It prevents deletion of a principal while related dependents exist, with actual database semantics provider-dependent.

### 166. What does `DeleteBehavior.ClientSetNull` mean?

For tracked dependents, EF sets nullable foreign keys to null on principal deletion; database behavior is separately configured.

### 167. What is an orphaned dependent?

A dependent no longer associated with its principal, which may be deleted or have its foreign key nulled by configured behavior.

### 168. What is required dependent behavior?

It specifies how a required dependent is handled when its principal is removed from the relationship.

### 169. What can cause multiple cascade path errors?

Some databases reject schemas where multiple cascading routes can delete the same row.

### 170. How can cascade path conflicts be addressed?

Choose deliberate non-cascading relationships or perform deletions in an explicit, database-safe order.

### 171. How does EF handle a foreign-key change?

It updates relationship snapshots and issues the dependent row update when changes are saved.

### 172. Why can setting only a navigation be sufficient?

EF relationship fix-up can synchronize the foreign key when the entity is tracked and the relationship is detected.

### 173. Why keep both FK and navigation properties?

The FK is useful for querying and disconnected updates; the navigation expresses object relationships.

### 174. What is a shadow foreign key?

It is a mapped foreign-key property stored only in the EF model and tracker, not as a CLR member.

### 175. How do you query a shadow property?

Use `EF.Property<T>` in a LINQ expression or access it through EF metadata APIs.

### 176. What is a backing-field navigation useful for?

It lets the domain restrict collection mutation while EF manages relationship contents.

### 177. How can a collection navigation avoid exposing mutation?

Expose a read-only view over a private collection and configure EF to use the backing field.

### 178. What is a collection navigation initialization requirement?

Initialize it to a usable collection so relationship fix-up can add related entities.

### 179. What is a principal key?

It is the primary or alternate key referenced by a relationship's foreign key.

### 180. What is a composite foreign key?

It references a principal key made up of multiple values, with matching dependent properties.

### 181. Why configure relationship pairs explicitly?

Explicit pairing avoids convention ambiguity when an entity has multiple navigations between the same types.

### 182. How are multiple relationships between the same types disambiguated?

Configure each relationship with `HasOne`, `WithMany` or `WithOne`, and its specific foreign key.

### 183. What is a unidirectional relationship?

It has a navigation on only one side, though the foreign key can still be modeled.

### 184. Can a relationship exist without navigation properties?

Yes. Configure it with foreign-key properties and Fluent API, even if the object model has no navigation.

### 185. What is a relationship constraint in the database?

A foreign-key constraint enforces referential integrity for stored rows.

### 186. How do nullable reference types affect relationship conventions?

EF may use nullability annotations to infer requiredness, but explicit mapping should clarify important schema intent.

### 187. What is a many-to-many join table's composite key for?

It commonly prevents duplicate associations by making the pair of foreign keys unique.

### 188. How can duplicate many-to-many associations be prevented?

Use a composite or unique key on the join columns, enforced by the database.

### 189. What is a join entity payload?

It is additional association data, such as creation time or role, stored alongside the join foreign keys.

### 190. How can a relationship use a non-primary key?

Configure an alternate principal key and point the dependent foreign key to it.

### 191. What is a navigation collection's inverse?

It is the corresponding navigation on the related entity that represents the same relationship.

### 192. What happens when inverse navigations are omitted?

The relationship may still be configured from one side, but explicit configuration improves clarity.

### 193. What is a disconnected relationship update?

It changes foreign keys or navigations for entities attached from outside the current context.

### 194. Why is replacing a navigation collection risky?

It may imply removals or orphaning for existing dependents; reconcile changes intentionally.

### 195. How can relationship changes be reconciled safely?

Load or attach the aggregate deliberately, compare requested membership, and apply only authorized additions and removals.

### 196. What does `WithMany()` without a navigation mean?

It configures a collection relationship where the principal has no CLR collection navigation.

### 197. What does `WithOne()` without a navigation mean?

It configures a one-to-one relationship without a navigation on that side.

### 198. What is an owned collection?

It is a collection of owner-dependent value-like entity instances whose lifecycle is tied to the owner.

### 199. How can an owned entity be mapped to a separate table?

Configure its table mapping explicitly; supported details vary by EF Core version.

### 200. What should be verified after configuring relationships?

Inspect keys, nullability, constraints, delete behavior, and generated migration or database schema.

## 10. Query composition and translation

### 201. What is the difference between `IQueryable<T>` and `IEnumerable<T>`?

`IQueryable<T>` composes provider-translated expressions; `IEnumerable<T>` enumerates in .NET after the query has become local.

### 202. What does `AsEnumerable` do in an EF query?

It switches subsequent operators to LINQ-to-Objects, so later filtering may run on the client.

### 203. What does `ToListAsync` do?

It executes and asynchronously materializes a query into a list; it should be awaited before the context is reused.

### 204. Does calling `Where` execute a query?

No. It builds the expression; execution occurs when results are enumerated or materialized.

### 205. What is deferred execution?

The query is built first and executed later when an operation requests results.

### 206. What is query composition?

It is incrementally building an expression tree from filters, ordering, projections, and other operators before execution.

### 207. Why should filtering usually occur before materialization?

It lets the database reduce rows before sending results, lowering transfer and application memory costs.

### 208. Why can multiple enumeration be expensive?

Each enumeration may execute the database query again and observe a different database state.

### 209. What is a parameterized query?

It sends values as parameters rather than embedding them into SQL text, improving safety and plan reuse.

### 210. How does EF Core parameterize LINQ values?

It extracts many captured values into database parameters; verify generated SQL for unusual expressions.

### 211. What is query-plan reuse?

The database may reuse a compiled execution plan for structurally similar parameterized commands.

### 212. What is a query tag?

`TagWith` adds a descriptive SQL comment to help correlate a query with its application call site.

### 213. What does `EF.Functions.Like` represent?

It expresses a SQL `LIKE` pattern match in a query; wildcard and collation behavior follow the database.

### 214. Why may `ToQueryString` not show the exact executed command?

Execution can add parameters, commands, or provider details that differ from the displayed query representation.

### 215. What is a correlated subquery?

It is a nested query whose predicate or projection refers to a row from the outer query.

### 216. How can `Any` improve query intent?

It expresses existence and commonly translates to an efficient SQL `EXISTS` predicate.

### 217. How does `Contains` translate in a query?

It may become an `IN` predicate or provider-specific equivalent; large collections can affect SQL size and performance.

### 218. What is an `EXISTS` query useful for?

It tests whether related rows exist without requiring them to be projected into the result.

### 219. How can a left join be expressed in LINQ?

Use a group join followed by `DefaultIfEmpty`, or a supported navigation projection that translates to an outer join.

### 220. What is a group join?

It pairs each outer row with a grouped sequence of matching inner rows.

### 221. How does `GroupBy` translation vary?

Simple aggregates often translate well; projecting arbitrary grouped objects may require client shaping or fail translation.

### 222. Why should grouping SQL be inspected?

LINQ grouping semantics and relational grouping differ, and translation may change with provider or EF version.

### 223. What does `Distinct` do in a relational query?

It asks the database to eliminate duplicate projected rows according to database comparison semantics.

### 224. Why can `Distinct` on entities be costly or ineffective?

Entity identity and projection shape matter; it can add sorting or hashing work and may not remove join duplication as intended.

### 225. How does ordering affect pagination?

Stable pagination needs deterministic ordering, usually including a unique tie-breaker.

### 226. What is offset pagination?

It uses `Skip` and `Take`; large offsets can become expensive and rows may shift between requests.

### 227. What is keyset pagination?

It filters by the last-seen sort key and takes the next page, often scaling better for sequential navigation.

### 228. What makes keyset pagination ordering safe?

Use a stable, unique ordering key and encode all sort values needed to resume accurately.

### 229. What is a projection into a DTO?

It selects required columns into a transport shape without exposing or tracking full entity graphs.

### 230. Can a DTO projection still track entities?

Yes, if the projection contains entity instances in a tracking query; scalar-only projections do not create tracked entities.

### 231. How does null semantics affect translated comparisons?

SQL uses three-valued logic; EF may add predicates to approximate C# null semantics depending on configuration.

### 232. What does `UseRelationalNulls` change?

It opts into database null comparison semantics rather than EF's default compensation for C# comparisons.

### 233. Why can string comparison results vary by database?

Collation controls case, accent, and ordering behavior, and defaults differ between database systems.

### 234. What is a database collation?

It defines rules for comparing and sorting textual values.

### 235. How can a query use a specific collation?

Use provider-supported collation configuration or query functions, while considering index compatibility.

### 236. Why can `ToLower` harm a string search?

Applying a function to a column can prevent ordinary index use; prefer an appropriate collation or indexed search strategy.

### 237. How should date comparisons be written?

Use typed date parameters and explicit ranges, accounting for timezone, precision, and provider-specific date types.

### 238. What is sargability?

A predicate is sargable when the database can use an index efficiently to locate qualifying rows.

### 239. What is a non-sargable predicate example?

Applying a function to an indexed column in a filter can prevent a seek, depending on the database optimizer.

### 240. How do you call a database function in LINQ?

Use a mapped function or provider API that EF can translate, then validate SQL against the target provider.

### 241. What happens when a LINQ method cannot be translated?

EF generally throws outside the top-level client projection rather than silently retrieving all rows for client evaluation.

### 242. What is top-level client projection evaluation?

EF can sometimes evaluate non-translatable projection code after fetching required columns, while the rest remains server-side.

### 243. Why avoid premature `AsEnumerable`?

It can move filters, joins, or ordering to memory and transfer far more data than necessary.

### 244. How does `SelectMany` usually translate?

It flattens nested sequences and commonly becomes a join or correlated query.

### 245. What does `DefaultIfEmpty` represent in a join?

It preserves outer rows with no match, commonly producing a left outer join.

### 246. What is a query expression tree?

It is structured code data that an `IQueryable` provider can inspect and translate.

### 247. Why can arbitrary helper methods fail in a query?

The provider cannot translate their method bodies unless they are mapped or evaluated in an allowed client projection.

### 248. How can reusable query logic stay composable?

Return `IQueryable<T>` from carefully bounded query-building functions and avoid executing it prematurely.

### 249. What is the risk of returning `IQueryable` across layers?

Callers can alter execution, add expensive operators, or depend on a context that has already been disposed.

### 250. How can raw SQL be parameterized safely?

Use interpolated EF SQL APIs that parameterize values; never concatenate untrusted text into SQL.

## 11. Materialization and tracking details

### 251. What is entity materialization?

It is constructing CLR entity instances from database result values.

### 252. What is EF Core's state manager?

It maintains tracked entity entries, identity resolution, and relationship state for a context.

### 253. Can results from `AsNoTrackingWithIdentityResolution` be saved directly?

No; attach the intended instances to a context and mark the allowed changes before saving.

### 254. What is a value buffer during materialization?

It is an internal representation of values read from a database row before EF assigns them to a result object.

### 255. What is snapshot change tracking?

EF stores original values and compares them with current values during change detection.

### 256. What is notification-based change tracking?

Entities notify EF about changes through supported interfaces or proxies, potentially reducing snapshot comparisons.

### 257. What does `DetectChanges` do?

It compares tracked state and relationship snapshots to find modifications; save operations usually invoke it automatically.

### 258. When can `AutoDetectChangesEnabled` be disabled?

Only during carefully controlled bulk in-memory modifications where explicit detection is performed before persistence.

### 259. What does `ChangeTracker.Entries()` return?

It exposes tracked entity entries and their current state and property metadata.

### 260. How can an entity's state be inspected?

Use `context.Entry(entity).State` or enumerate `ChangeTracker.Entries()`.

### 261. What is `ChangeTracker.DebugView` useful for?

It provides a diagnostic view of tracked entities, states, keys, and relationships.

### 262. What does `ChangeTracker.Clear()` do?

It detaches all currently tracked entities without issuing database operations.

### 263. When can clearing a change tracker help?

It can bound memory in long-running batch work after a unit is complete; it is not a context-lifetime substitute.

### 264. What is identity conflict when attaching entities?

It occurs when a context already tracks a different instance with the same entity type and key.

### 265. How can an identity conflict be avoided?

Query and update the tracked instance, or ensure only one instance per key is attached to the context.

### 266. How does `Attach` treat an entity with an unset generated key?

Graph state assignment can mark it `Added` when the generated key is unset; verify state for the configured key strategy.

### 267. What does `IsKeySet` tell you about a detached entity?

It indicates whether EF considers its key assigned, which can help classify new versus existing instances.

### 268. What does `Add` do to a graph?

It tracks reachable untracked entities as new, subject to graph traversal and existing tracked instances.

### 269. Why can graph attachment be surprising?

EF assigns states across reachable entities based on keys and operation, potentially inserting or modifying more than expected.

### 270. What does `TrackGraph` provide?

It allows application-defined state assignment while traversing a disconnected entity graph.

### 271. What is a property-level modification?

It marks an individual tracked property as modified so the update can target only intended columns.

### 272. How can original values be inspected?

Read the `OriginalValue` from a tracked property entry.

### 273. What does `GetDatabaseValues` do?

It queries current store values for a tracked entity, commonly for concurrency conflict resolution.

### 274. What is `CurrentValues.SetValues` useful for?

It copies matching scalar values from a source object to an entity, but authorization and allowed fields remain the caller's responsibility.

### 275. What does `Reload` do?

It refreshes a tracked entity's values from the store and resets its tracked state.

### 276. Why can reloading discard user changes?

Store values replace current tracked values, so save or preserve intended edits before reloading.

### 277. What is a tracking query's navigation fix-up effect?

Previously tracked related entities may populate navigations even when they were not included by the current query.

### 278. What does `AutoInclude` configure?

It marks a navigation for automatic eager loading in queries for that entity type, where supported.

### 279. What is filtered include?

It limits an included collection using supported operators such as filtering, ordering, and paging.

### 280. Why can filtered include be surprising in a tracking context?

Navigation fix-up may add previously tracked related entities that do not satisfy the include filter.

### 281. What is a navigation's loaded flag?

It indicates whether EF considers that navigation loaded; it does not guarantee every possible related row is present after filtering.

### 282. How can a navigation be explicitly marked loaded?

Set `Entry(entity).Collection(...).IsLoaded` or the reference equivalent when application logic has established that state.

### 283. What is `Find` optimized for?

It checks the context's identity map first, then queries by primary key if the entity is not already tracked.

### 284. How does `FindAsync` differ from a normal LINQ query?

It can return an already tracked primary-key match without issuing a database query.

### 285. Do no-tracking queries include unsaved local changes?

No. They query the database and do not use the context's tracked instances or pending modifications.

### 286. Why does projecting to scalar values reduce tracking work?

No entity instances are materialized to track when the projection contains only scalar or DTO data.

### 287. What is identity resolution's lifetime in a no-tracking query?

For `AsNoTrackingWithIdentityResolution`, the temporary identity map exists only while materializing that query result.

### 288. What is a materialization interceptor?

It is an interception hook around entity creation, useful for cross-cutting materialization concerns.

### 289. Why should materialization hooks be kept simple?

They run as rows are materialized and can obscure query behavior or add substantial per-row overhead.

### 290. What is a keyless query result's tracking behavior?

Keyless entity types are not tracked as regular entities.

### 291. How does EF handle database nulls for non-nullable CLR properties?

It can fail during materialization when database null cannot be assigned to the CLR member.

### 292. What is a value converter's materialization role?

It converts the provider value read from the store into the model property's CLR representation.

### 293. What is a constructor-bound entity?

EF uses a suitable constructor to initialize mapped properties when materializing an entity.

### 294. Can navigations generally be constructor-bound?

Navigation binding has constraints and should not be assumed; configure and test supported constructor patterns for the version.

### 295. What is a service property?

It is a non-persisted property through which EF can provide certain framework services to an entity.

### 296. Why should entity constructors avoid database work?

Materialization invokes constructors; I/O there causes hidden queries, poor performance, and difficult lifecycle behavior.

### 297. What is a query's result cardinality?

It describes whether the result is a sequence, zero-or-one, or exactly-one value.

### 298. How does `SingleAsync` differ from `FirstAsync`?

`SingleAsync` requires exactly one result; `FirstAsync` requires at least one and returns the first.

### 299. What happens when `FirstOrDefaultAsync` finds no row?

It returns the default value for the result type, commonly null for reference types.

### 300. When do aggregate query operators execute?

Operators such as `CountAsync`, `MaxAsync`, and `SumAsync` execute a database query when awaited.

## 12. Persistence, transactions, and reliability

### 301. What is a unit of work in EF Core?

It is a bounded set of tracked changes committed together, commonly represented by one context and save operation.

### 302. Are multiple `SaveChanges` calls automatically one atomic operation?

No. Use an explicit transaction when several saves must succeed or fail as one unit.

### 303. Should synchronous EF calls be wrapped in `Task.Run`?

Usually not; use EF's asynchronous I/O APIs rather than consuming a worker thread for blocking database calls.

### 304. What is `SaveChanges(acceptAllChangesOnSuccess: false)` for?

It saves without marking changes accepted, allowing controlled coordination before calling `AcceptAllChanges`.

### 305. What does `AcceptAllChanges` do?

It marks successfully persisted tracked entries as unchanged and updates their original-value snapshots.

### 306. Can one save persist changes from multiple entity types?

Yes. A context can track multiple entity types and persist their changes in one save operation.

### 307. Does one save always become one SQL statement?

No. It may issue multiple commands, though a relational provider generally wraps the save atomically in a transaction.

### 308. What is a save interceptor?

It observes or changes save operations around `SaveChanges`, such as applying auditing conventions.

### 309. What is a `SaveChanges` override useful for?

It can implement context-wide persistence behavior, but cross-cutting concerns may be clearer in an interceptor or domain layer.

### 310. How can audit fields be populated on save?

Set them from trusted application identity and time sources in a deliberate save pipeline or interceptor.

### 311. What is a deferred database constraint?

Where supported, it is checked at transaction commit rather than immediately after each statement.

### 312. What is a database-generated key?

It is a key value assigned by the database or provider during insertion.

### 313. How does EF retrieve an identity key after insert?

The provider emits appropriate SQL and reads the generated value, then updates the tracked entity.

### 314. What is a client-generated key?

It is assigned by application code before insertion, such as a GUID generated by the application.

### 315. Why might client-generated keys help disconnected graphs?

They let related entities refer to stable keys before database insertion.

### 316. What is a store-generated concurrency token?

It is a value such as a row version that the database changes on update and EF checks during writes.

### 317. What does the integer returned by `SaveChanges` represent?

It reports the number of state entries written, not necessarily the number of affected database rows.

### 318. How can concurrency conflicts be resolved?

Compare current, original, and database values, then retry, merge, reload, or return a conflict to the caller.

### 319. What is write skew?

It is a concurrency anomaly where individually valid writes together violate an invariant based on a shared observation.

### 320. How can serializable isolation address write skew?

It may prevent conflicting interleavings, but exact guarantees and costs depend on the database implementation.

### 321. What is pessimistic concurrency?

It coordinates writers using database locks or isolation behavior rather than detecting conflicts only at save time.

### 322. Does EF Core provide a universal row-lock API?

No; locking syntax and semantics are database-specific and typically require provider-specific SQL or transaction configuration.

### 323. What is a transaction isolation level?

It defines visibility and locking behavior for concurrent transactions, with semantics that vary by database.

### 324. What is a savepoint?

It marks a point within a transaction to which a database can roll back without aborting the whole transaction.

### 325. When might EF create a savepoint?

Relational providers may create one around `SaveChanges` inside an existing transaction when supported and configured.

### 326. What can disable savepoint behavior on SQL Server?

Features such as Multiple Active Result Sets can affect savepoint support; consult provider documentation for the configuration.

### 327. How can two contexts share a transaction?

Relational contexts can share a connection and transaction through provider-supported APIs and explicit coordination.

### 328. Why share a transaction across contexts?

It can coordinate database changes across separate context units when a single atomic transaction is required.

### 329. Can a transaction include an external HTTP request atomically?

Not through a normal database transaction; use patterns such as an outbox and compensating workflow.

### 330. What is the transactional outbox pattern?

It stores messages in the same database transaction as business changes, then publishes them asynchronously.

### 331. What is a transient database failure?

It is a temporary fault, such as a brief network disruption or failover, that may succeed on retry.

### 332. What is an idempotent operation?

Repeating it produces the same intended effect as performing it once.

### 333. Why should retryable writes be idempotent?

A client may not know whether a failed response occurred before or after the database committed.

### 334. What is a check-constraint violation?

It occurs when a written row fails a database rule that validates its values.

### 335. What is a foreign-key constraint violation?

It occurs when a dependent row references a principal key that does not exist or cannot be referenced.

### 336. What transaction behavior do set-based operations have?

Each command follows provider and database transaction behavior; combine commands in an explicit transaction if atomicity is needed.

### 337. Can an `ExecuteUpdate` predicate use a navigation?

It is translated as a set-based query where supported; validate the expression and SQL with the target provider.

### 338. Can `ExecuteDelete` invoke entity cascade behavior in memory?

It executes set-based database deletion and bypasses per-entity change tracking; database constraints determine dependent effects.

### 339. What is the purpose of batching commands?

It reduces database round trips by grouping multiple commands, subject to provider limits and configured batch size.

### 340. What can limit a batch size?

Provider parameter limits, command size, latency, and database resource constraints.

### 341. What does `MaxBatchSize` configure?

It caps the number of statements sent in a relational command batch for supported providers.

### 342. Why can a failed save leave tracked entities modified?

The context may retain its in-memory states after failure; inspect and recover deliberately rather than assuming it reset.

### 343. Should a context be reused after an unexpected save failure?

It depends on the failure; for uncertain transactional state, discard the context and start a fresh unit of work.

### 344. What is a transaction scope?

It is an ambient transaction abstraction that can coordinate enlistment, but behavior depends on provider and environment.

### 345. What is the risk of distributed transactions?

They may require infrastructure support, increase operational complexity, and are not uniformly available across platforms/providers.

### 346. How can a transaction be kept reliable under retries?

Execute the complete transaction delegate through the configured execution strategy and make its work replay-safe.

### 347. What is a cancellation token for in EF async APIs?

It requests cancellation of pending asynchronous database work where the provider supports cancellation.

### 348. Does cancellation guarantee a database command never ran?

No. Cancellation can race with execution or commit; the caller may need to resolve outcome safely.

### 349. How should transaction duration be minimized?

Perform validation and external work beforehand, then keep only required database operations inside the transaction.

### 350. What does atomicity mean for a save?

Either all database changes in the transaction commit or none do, subject to provider transaction guarantees.

## 13. Schema evolution and deployment

### 351. What is a migration operation?

It is a schema change represented by an `Up` operation and its corresponding `Down` operation where reversal is possible.

### 352. What does the `Up` method contain?

It applies the forward schema change when the migration is run.

### 353. What does the `Down` method contain?

It attempts to reverse a migration; some data transformations cannot be safely undone.

### 354. Why review generated migration code?

Scaffolding can infer incorrect or destructive changes when model intent is ambiguous.

### 355. What is a migration identifier?

It is the ordered migration name and timestamp recorded by EF tooling and stored in migration history.

### 356. What is the migrations history table?

It records which migrations have been applied to a database.

### 357. Can an existing database be brought under migrations?

Yes, but establish a correct baseline and history state carefully rather than blindly applying the initial schema.

### 358. What is a baseline migration?

It represents the existing schema as a starting point so later changes can be tracked incrementally.

### 359. What is a migration bundle?

It is a packaged executable for applying migrations without requiring the EF CLI on the target machine.

### 360. What is an idempotent migration script?

It checks migration history and applies only migrations not already recorded as applied.

### 361. Why generate SQL scripts for production?

Scripts can be reviewed, approved, audited, and applied by a controlled deployment process.

### 362. What is a migration dry run?

It is inspecting generated SQL or applying changes to a disposable environment before production rollout.

### 363. Why avoid applying migrations at every application startup?

Concurrent instances, elevated runtime permissions, startup delays, and unreviewed DDL create operational risk.

### 364. What is an expand-and-contract migration?

It introduces a backward-compatible schema, deploys compatible application versions, migrates data, then removes obsolete schema.

### 365. Why use backward-compatible schema changes?

They allow old and new application versions to coexist during rolling or staged deployment.

### 366. How can a column rename be deployed safely?

Use staged add-copy-switch-remove steps when application versions overlap; a direct rename may break older instances.

### 367. What is a data migration?

It transforms stored rows as part of schema evolution or a separately managed data operation.

### 368. Why may large data updates belong outside a schema migration?

They can hold locks or run too long; separately batched and monitored jobs may be safer.

### 369. What is a migration lock?

It coordinates migration execution to avoid concurrent schema changes, with support and behavior depending on EF version/provider.

### 370. How should seed data changes be reviewed?

Review generated inserts, updates, and deletes to ensure changing model seed data will not overwrite operational data.

### 371. What does `dotnet ef migrations remove` do?

It removes the most recent migration from source when it has not been applied to the target database.

### 372. What happens when a migration snapshot is stale?

New migration generation can produce incorrect or incomplete differences; restore consistency with migrations and model.

### 373. How do multiple `DbContext` types manage migrations?

Configure separate migration assemblies or histories as needed and select the intended context explicitly in tooling.

### 374. What is a migrations assembly?

It is the assembly containing migration classes and the model snapshot for a context.

### 375. How can migration history be stored separately?

Configure the history table name and schema using provider-supported options.

### 376. Why can changing a database schema name affect migrations?

Schema defaults, object qualification, permissions, and generated migration operations may all change.

### 377. What is a migration's target model?

It is the model metadata associated with that migration and used by EF tooling and runtime migration services.

### 378. How can a migration be applied to a particular environment?

Use the deployment pipeline's environment-specific connection and controlled migration artifact.

### 379. Why should migration credentials differ from app credentials?

Schema changes need elevated rights; least-privileged runtime credentials should not ordinarily own DDL privileges.

### 380. What is schema drift?

The actual database schema differs from the schema represented by migrations and the intended model.

### 381. How can schema drift be detected?

Compare migration state and database metadata, and validate schema in deployment checks or monitoring.

### 382. Can migrations fully manage stored procedures and triggers?

They can include custom SQL or operations, but those objects often need explicit lifecycle management.

### 383. How can custom SQL be added to a migration?

Use `migrationBuilder.Sql` with carefully reviewed, provider-appropriate commands.

### 384. Why is raw SQL in migrations provider-sensitive?

DDL syntax, data types, transactional support, and procedural features differ among databases.

### 385. How should a migration handle a destructive column removal?

Confirm data retention and application rollout, back up as required, and separate removal from code that still reads the column.

### 386. What is a migration squashing or re-baselining?

It consolidates historical migrations into a new baseline, requiring careful coordination across every deployed database.

### 387. Why is migration history rewriting dangerous?

Databases already deployed may not match the rewritten migration sequence or baseline assumptions.

### 388. How can an initial migration be regenerated safely?

Only before it is deployed, or through a coordinated re-baseline that preserves existing database schema and history.

### 389. What does `dotnet ef migrations list` show?

It lists migrations known to the selected context and can indicate applied state when connected to a database.

### 390. What is a pending model change?

It is a model difference not yet represented by a migration; tooling can detect it in supported versions.

### 391. What does `dotnet ef migrations has-pending-model-changes` check?

It compares the current model with the snapshot in supported EF tooling versions.

### 392. Why specify startup and target projects to EF tools?

The startup project supplies runtime configuration while the target project may contain the context and migrations.

### 393. What is an EF tooling startup project?

It is the project built and used to resolve design-time services and context configuration.

### 394. How can a context factory simplify migration tooling?

It creates the context independently of application host startup and external service initialization.

### 395. What should a design-time factory avoid?

Hard-coded secrets, production-only side effects, and configuration that silently targets an unintended database.

### 396. What is a migration deployment artifact?

It is a reviewed script, bundle, or other reproducible package produced by the build pipeline.

### 397. How can migration execution be audited?

Record the artifact version, target database, operator or pipeline identity, outcome, and migration history.

### 398. What is a blue-green database deployment concern?

Both application versions may use the same schema, so schema changes must remain compatible during traffic switching.

### 399. Why test migrations from an empty database?

It verifies the complete migration chain can construct the expected schema.

### 400. Why test migration upgrades from prior releases?

It verifies real upgrade paths and catches failures hidden by only testing the newest schema.

## 14. Performance, diagnostics, and testing

### 401. What is the database query plan?

It is the database optimizer's chosen operations for executing a query.

### 402. Why inspect an actual execution plan?

It reveals scans, joins, estimates, and resource use that generated SQL alone cannot show.

### 403. What is parameter sniffing?

Some databases optimize a plan using parameter values from an execution, which can perform poorly for different value distributions.

### 404. Why is a large object graph expensive to load?

It increases transferred rows, materialization work, memory, and relationship fix-up.

### 405. What is cartesian explosion in eager loading?

Joining multiple collections can multiply result rows and repeat parent data.

### 406. How can cartesian explosion be mitigated?

Project selectively, split collection loading when appropriate, or load related sets separately.

### 407. What is query compilation in EF Core?

It translates a LINQ expression tree into a provider query representation and caches reusable results.

### 408. What are compiled-query limitations?

They add complexity and support a constrained query shape; they do not improve database execution itself.

### 409. What is query compilation caching?

EF caches translation work for equivalent query shapes, reducing repeated expression-processing overhead.

### 410. How can dynamically built queries affect caching?

Changing expression structure unnecessarily can create many query shapes and reduce cache reuse.

### 411. What is a parameterized compiled model?

It is not a standard term; model shape is normally cached, while query values should be parameters.

### 412. How can pagination protect an API from oversized reads?

Enforce bounded page sizes and deterministic ordering on server-side queries.

### 413. Why avoid returning unbounded entity sets?

They can exhaust database, network, and application resources and expose more data than needed.

### 414. What is connection pool exhaustion?

Requests wait because all available database connections are busy or leaked.

### 415. How can connection pool pressure be reduced?

Dispose contexts promptly, avoid long-running readers and transactions, and tune concurrency based on database capacity.

### 416. Does disposing a context close a pooled database connection permanently?

Usually the driver returns the connection to its pool; exact behavior depends on provider and connection configuration.

### 417. What is a query-count regression?

A code change unexpectedly increases the number of database round trips.

### 418. How can query counts be tested?

Capture EF diagnostics or use a test interceptor to assert expected command counts for critical paths.

### 419. What is EF Core command logging?

It records database commands and execution details through the configured .NET logging system.

### 420. Why should production SQL logging be carefully configured?

Verbose logs may be costly and can expose query structure or sensitive parameter values.

### 421. What are EF Core diagnostic events?

They are structured notifications about context, query, and database operations that can support instrumentation.

### 422. What is `DiagnosticListener` used for?

It allows observers to subscribe to diagnostic events emitted by EF and other .NET components.

### 423. What can OpenTelemetry instrumentation add?

It can correlate database operations with traces and metrics through compatible instrumentation packages.

### 424. What is a trace span for a database operation?

It records a timed operation and correlation metadata that can connect application work to database activity.

### 425. What is a database command interceptor useful for in tests?

It can capture commands or inject controlled behavior to verify persistence interactions.

### 426. Why prefer integration tests for query translation?

They execute against a real provider and reveal translation, collation, constraints, and SQL behavior.

### 427. Why use disposable database containers in integration tests?

They provide isolated, repeatable instances of the actual database engine and provider.

### 428. What is a migration test matrix?

It tests fresh creation and upgrades from relevant previous schema versions using the production provider.

### 429. How can SQLite in-memory keep a test database alive?

Keep the underlying connection open for the test lifetime when using a shared in-memory database.

### 430. What is a test database fixture?

It manages setup, cleanup, and reuse of a database environment across persistence tests.

### 431. When should tests use a disposable database instance?

Use one for isolated integration tests that need production-provider fidelity and reliable cleanup.

### 432. How can test data isolation be achieved?

Use separate databases or schemas, transactions where suitable, unique test data, and deterministic teardown.

### 433. Why are transaction-wrapped tests not always sufficient?

Some operations, providers, background work, or separate contexts may escape the test transaction.

### 434. What is a migration integration test?

It applies migrations to a fresh or prior-version database and verifies resulting schema and application behavior.

### 435. How can test execution speed be improved without changing semantics?

Reuse expensive infrastructure carefully, parallelize isolated tests, and avoid substituting an inaccurate provider for critical behavior.

### 436. What is an EF benchmark measuring?

It may measure query translation, database execution, materialization, allocations, or end-to-end latency; define the boundary.

### 437. Why separate database and ORM costs in benchmarks?

Network and database execution often dominate, so isolated measurements help identify the actual bottleneck.

### 438. What is a projection benchmark benefit?

It can quantify reduced columns, allocations, and materialization compared with full entity loading.

### 439. How can batch size affect bulk writes?

Larger batches reduce round trips but can exceed parameter limits or increase lock duration and memory.

### 440. What is a bulk operation library?

It is a third-party or provider-specific mechanism for high-volume set operations beyond ordinary tracked writes.

### 441. What should be checked before adopting bulk extensions?

Verify provider/version support, transaction semantics, generated SQL, tracking behavior, and license or maintenance status.

### 442. How can large exports avoid high memory use?

Select only needed columns and stream results where supported, while controlling context and reader lifetime.

### 443. What is `AsAsyncEnumerable` useful for?

It asynchronously streams query results for incremental processing instead of buffering the full result with `ToListAsync`.

### 444. What is a streaming query tradeoff?

It reduces buffering but holds database resources longer and complicates concurrent use of the same context.

### 445. Why should a streaming query not share its context concurrently?

An active data reader or context operation can conflict with another command on the same context.

### 446. What is a performance counter for EF applications?

Track query duration, command count, connection waits, allocations, and database resource metrics together.

### 447. How can slow queries be identified in production?

Use database query monitoring and application traces with query tags, while protecting sensitive data.

### 448. What is a cardinality estimate?

It is the optimizer's estimate of rows flowing through a query plan, which affects operator choices.

### 449. Why can stale statistics slow an EF query?

The optimizer may choose a poor plan when database statistics misrepresent current data distribution.

### 450. When is result caching appropriate for database reads?

Use it for repeatable reads with a defined invalidation and freshness policy; do not cache authorization decisions blindly.

## 15. Security, tenancy, and advanced scenarios

### 451. Does EF Core automatically prevent SQL injection?

LINQ values are generally parameterized, but unsafe raw SQL concatenation can still create injection vulnerabilities.

### 452. How should raw SQL include user-supplied values?

Pass them as parameters through EF's parameterized APIs rather than concatenating them into SQL text.

### 453. Can SQL identifiers be parameterized?

Ordinary SQL parameters represent values, not identifiers; validate and allowlist dynamic table or column names.

### 454. How can dynamic sorting be secured?

Map requested sort tokens to known entity properties or prebuilt expressions instead of accepting arbitrary SQL text.

### 455. What is a tenant escape defect?

It is an access path that lets a request read or change another tenant's data.

### 456. What defense-in-depth controls can protect tenant data?

Combine trusted tenant resolution, authorization, scoped queries, database constraints or row policies, and isolation tests.

### 457. What can bypass a global query filter?

Explicitly ignoring filters, raw SQL paths, or other access paths may bypass it; audit each operation.

### 458. How should soft-deleted rows be restored?

Use an authorized operation that explicitly updates deletion state and observes uniqueness and relationship constraints.

### 459. How can soft deletion affect uniqueness?

A unique constraint may still include deleted rows; use a provider-supported filtered index or a deliberate restore policy.

### 460. How can tenant data isolation be reinforced in the schema?

Include tenant identifiers in keys or constraints where appropriate and enforce access through database policies when available.

### 461. What is row-level security?

It is a database feature that restricts visible or writable rows based on configured security predicates.

### 462. Why must row-level security be tested with the application identity?

Admin or owner connections may bypass policies and give misleading test results.

### 463. How can least privilege be applied to EF applications?

Grant runtime identities only the database permissions required for normal reads and writes.

### 464. What should a migration deployment pipeline enforce?

It should review and version the artifact, target the intended database, restrict permissions, and record the result.

### 465. What is encryption in transit for database access?

It protects the connection channel using TLS when correctly configured and validated by the provider/driver.

### 466. What is encryption at rest?

It protects database storage through database, disk, or managed-service encryption features.

### 467. How should audit data be protected?

Restrict access, minimize collected fields, and define retention and deletion policies for audit records.

### 468. How can secrets be kept out of EF migrations?

Use parameterized configuration and deployment secret stores; migrations should define schema, not embed credentials.

### 469. What is a tenant-aware context?

It is a context that receives trusted tenant state and applies it consistently to tenant-scoped operations.

### 470. How should tenant state be supplied to each unit of work?

Resolve it from trusted request or job context and pass it explicitly when constructing or using the context.

### 471. What is schema-per-tenant architecture?

Each tenant's tables reside in a separate schema, increasing isolation but complicating model caching and migrations.

### 472. What is database-per-tenant architecture?

Each tenant uses a separate database, improving isolation but increasing connection, migration, and operational complexity.

### 473. What is shared-database tenancy?

Tenant rows share tables and are distinguished by tenant keys and access controls.

### 474. How should tenant isolation strategy be selected?

Compare regulatory needs, scale, operational capacity, migration process, and acceptable failure boundaries.

### 475. What is the outbox pattern's delivery guarantee?

It commonly provides at-least-once delivery, so consumers should be idempotent or deduplicate messages.

### 476. What is a saga?

It coordinates a long-running distributed business process through local transactions and compensating actions.

### 477. Why are sagas different from database transactions?

They span independent services and cannot generally provide one ACID transaction across all participants.

### 478. What is a CQRS read model?

It is a query-optimized representation that may be separate from the write model and updated asynchronously.

### 479. Can EF Core support CQRS?

Yes. EF can implement command persistence, read projections, or both, but architecture should match actual complexity.

### 480. What is a table-valued parameter?

It is a database-specific parameter representing rows, useful for bulk input when supported by the provider.

### 481. How can provider-specific database functions be used safely?

Isolate them behind provider-specific query code and test translation and semantics against that provider.

### 482. What is a temporal table?

It is a database-managed table that preserves row versions over time; EF support depends on provider and version.

### 483. What is temporal querying?

It retrieves historical rows as of a specified time using database temporal features where supported.

### 484. What is a database sequence?

It is a database object that generates numeric values independently of inserting a particular row.

### 485. When use a sequence instead of identity generation?

Use it when values are needed before insert or shared across tables, if the provider supports the required behavior.

### 486. What is a database generated concurrency token on PostgreSQL?

Provider-specific system columns or application-managed tokens may be used; choose and configure the mechanism supported by the provider.

### 487. Why are database-specific types important?

Types such as arrays, ranges, spatial values, and JSON may provide capabilities with provider-specific mapping and query semantics.

### 488. What is spatial data support in EF Core?

Providers can map spatial types and translate spatial operations, commonly using a geometry library and database extensions.

### 489. How should spatial queries be validated?

Test coordinate reference systems, units, boundary semantics, translation, and indexes against the actual database.

### 490. Does mapping an EF type to a view create the view?

No. Configure the mapping and manage view creation separately, for example with reviewed migration SQL.

### 491. How can EF coordinate with hand-written SQL?

Use the same connection and transaction when required, and refresh or clear tracked state after external updates.

### 492. What should code consider after issuing direct SQL through EF?

The command may bypass change tracking and entity-level behavior; synchronize affected state before using tracked instances.

### 493. How can raw SQL participate in an EF transaction?

Use the context's connection and transaction APIs to coordinate it with other commands.

### 494. What is an application-managed concurrency token?

It is a value the application changes on each update and configures EF to compare during writes.

### 495. What is a soft delete interceptor tradeoff?

It can centralize conversion of deletes to updates, but may obscure semantics and needs careful handling of cascades and bulk APIs.

### 496. How should auditing handle actor identity?

Obtain it from trusted request or service context and persist it with appropriate privacy and retention controls.

### 497. What is data minimization in EF queries?

Select and return only fields needed for the operation, reducing exposure and resource use.

### 498. How can an EF application support database portability?

Keep core queries within common translations, isolate provider-specific features, and run integration tests per supported provider.

### 499. How should a migration rollback plan be prepared?

Document reversible steps, data-loss limits, backups, and a forward-fix option; not every migration can be safely undone.

### 500. What is the best advanced EF Core design principle?

Treat EF as a database abstraction, not a database replacement: understand the provider, SQL, consistency, and operational behavior.
