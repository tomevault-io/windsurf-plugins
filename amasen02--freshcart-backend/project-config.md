---
trigger: always_on
description: This page is the single source of truth for "how do we write code here". The defaults are encoded
---

# Coding conventions

This page is the single source of truth for "how do we write code here". The defaults are encoded
in `.editorconfig`, `Directory.Build.props`, `Directory.Packages.props` and the Roslyn analyzer
suite — the prose below explains the *why* so reviewers can hold each other to it.

---

## Naming

| Element | Style | Examples |
|---|---|---|
| Namespaces, classes, records, structs, enums | PascalCase | `OrderingDbContext`, `RefreshToken`, `CanonicalRoles` |
| Interfaces | `I` + PascalCase | `IRefreshTokenService`, `IIdentityAuditLog` |
| Methods | Verb-first PascalCase + `Async` suffix on async | `IssueAsync`, `RecordSuccessfulSignIn` |
| Properties | PascalCase | `CreatedOnUtc`, `MultiFactorEnabled` |
| Parameters, locals | camelCase, full words | `cancellationToken`, `signUpRequest`, `pendingMessages` |
| Private fields | `_` + camelCase | `_options`, `_msSqlContainer` |
| Constants | PascalCase | `MinimumPasswordLength`, `SessionCookieName` |
| Type parameters | Single capital with descriptive prefix | `TUser`, `TResponse`, `TRequest` |

Rules that catch the AI-fingerprint smell:

- **No single-letter locals** outside the tightest of loops. `i`/`j` are tolerated only as direct
  loop indices, and even then the LINQ-style `.Select((item, index) => …)` is preferred.
- **No abbreviation-as-naming.** `repo` is `userRepository`; `ctx` is `httpContext`; `tmp` is
  whatever it actually represents.
- **Methods read like sentences.** `await refreshTokenService.RotateAsync(...)` describes the
  intent; `await refreshTokenService.ProcessAsync(...)` does not.

---

## File layout

- One public type per file, file name matches the type.
- `internal sealed` is the default for handlers and helpers — only widen visibility when a type is
  consumed across project boundaries.
- File-scoped namespaces everywhere (`namespace FreshCart.X.Y;`).

---

## Async

- Every `async` method takes a `CancellationToken` and forwards it to the next call that does I/O.
- `ConfigureAwait(false)` is used inside library code (BuildingBlocks, BuildingBlocks.Messaging,
  ServiceDefaults). It is omitted in ASP.NET Core application code because there is no
  synchronisation context to deadlock against.
- Never call `.Result` / `.GetAwaiter().GetResult()`. The single exception is the EF Core
  `SaveChangesInterceptor.SavingChanges` (non-async overload) — which mirrors the EF base class.
- Method names end with `Async` so the type is obvious at the call site.

---

## LINQ + IQueryable / IEnumerable

- Stay in `IQueryable<T>` as long as possible — projecting / filtering before materialisation lets
  EF compose SQL on the server.
- Materialise with the explicit method: `ToListAsync`, `FirstOrDefaultAsync`, `SingleAsync`,
  `AsAsyncEnumerable`. Never use the non-async overloads in request paths.
- Apply `AsNoTracking()` on every read path. Tracking is opt-in, not opt-out.
- Use `AsSplitQuery()` for `Include(...)` chains wider than one level.
- Never enumerate twice — if a result is consumed twice, materialise it once and operate on the
  list.

---

## Lazy loading

- EF lazy loading is **off** by default. Each include is explicit.
- The single exception is the Backoffice modular monolith where the read-only admin grids enable
  lazy loading inside a tracked-by-default scope, documented in the relevant ADR.

---

## DTOs, commands, queries, entities

- **Records** for DTOs, commands, queries, value objects — they are immutable and equality-by-value
  matches how they are reasoned about.
- **Classes** for entities, aggregates, services — they have identity and mutable state.
- DTOs live in either the endpoint contracts file or the `Common/Models` folder; they never leak
  EF-tracked entities to the wire.

---

## Dependency injection

- Constructor injection is the default.
- `[FromKeyedServices("…")]` for keyed services on .NET 8+ when two implementations of the same
  interface need to coexist.
- Never use the service locator (no `provider.GetService<T>()` inside business code).
- `IServiceCollection` extensions live in a `DependencyInjection.cs` file per project
  (Application, Infrastructure, Api) so registrations are findable.

---

## Comments and docs

- The default is **no comment**. Well-named identifiers are the documentation.
- Comments explain **why** something is the way it is — invariants, surprising trade-offs, links
  to ADRs.
- Comments never explain **what** the next line does (`// loop through products` is rejected at
  review).
- Public APIs carry XML doc comments. Internal members do not unless the *why* is non-obvious.
- Commit messages are imperative, terse, technical. Conventional Commits format.

---

## Error handling

- Throw rich domain or application exceptions — `NotFoundException`, `BadRequestException`,
  `ConflictException`, `ForbiddenException`, `DomainException`, `InternalServerException`.
- `CustomExceptionHandler` (in BuildingBlocks) translates them into RFC 7807 `ProblemDetails`.
  Application code never reaches for HTTP status codes.
- Catch only when you can do something useful. Logging + rethrow is the most common useful
  action.

---

## Validation

- One `AbstractValidator<TCommand>` per command, registered automatically.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amasen02/freshcart-backend](https://github.com/amasen02/freshcart-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
