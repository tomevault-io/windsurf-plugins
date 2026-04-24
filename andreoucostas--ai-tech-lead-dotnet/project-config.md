---
trigger: always_on
description: > This file is the single source of truth for AI-assisted development in this repository.
---

# [Project Name]

> This file is the single source of truth for AI-assisted development in this repository.
> It is automatically loaded by Claude Code on every session.
> Run `/bootstrap` to populate it from your actual codebase.

---

## Codebase Context

<!-- Populated by /bootstrap — do not fill manually -->

What this application does, who uses it, key domain concepts, and critical user journeys.

---

## Solution Structure

<!-- Populated by /bootstrap — replaces separate CODEMAP.md -->

Project layout, layering strategy, dependency direction between projects, entry points, and where to put new code.

Include a text or mermaid diagram showing project dependencies.

---

## Conventions

<!-- Populated by /bootstrap — replaces separate CONVENTIONS.md -->
<!-- Each convention: the rule, then 1-2 sentence rationale -->

> **DEFAULTS BELOW.** Everything in this section is a starting template. When you run
> `/bootstrap`, these are replaced with conventions observed in your actual codebase.
> If you haven't run `/bootstrap` yet, do not treat these as authoritative.

### .editorconfig & Analysers
<!-- Check for .editorconfig, Directory.Build.props, and Roslyn analyser rules. Reference them here so AI tools respect toolchain-enforced conventions. -->

### Architecture
- Dependency direction: inward only. API → Application → Domain. Never the reverse.
- Domain layer has zero external dependencies.

### Naming
- Classes: PascalCase. Interfaces: `I` prefix. Async methods: `Async` suffix.
- Files match class names exactly. One public class per file.

### Dependency Injection
- Services: scoped. Factories and stateless helpers: transient. Caches and config: singleton.
- Register via extension methods per project, not in Program.cs directly.
- Use `IOptions<T>` for static config, `IOptionsMonitor<T>` for config that can change at runtime, `IOptionsSnapshot<T>` for scoped config refresh.

### Data Access
- EF Core with repository pattern only where it adds value (not wrapping DbContext for the sake of it).
- Queries belong in the application/service layer, not in controllers.
- Always use `.AsNoTracking()` for read-only queries.

### API Design
- Controllers are thin — delegate to services immediately. Minimal APIs are acceptable for simple endpoints if the project uses them.
- Request/response DTOs are separate from domain entities. Never expose domain models in API contracts.
- Use FluentValidation for request validation. No validation logic in controllers.
- Background work uses `BackgroundService` or `IHostedService`. No `Task.Run` fire-and-forget in request handlers.

### Async
- Propagate `CancellationToken` through every async call chain.
- No `async void`. No sync-over-async. No fire-and-forget without explicit justification.

### Null Handling
- Nullable reference types enabled project-wide. No suppression (`!`) without a comment explaining why.
- Guard clauses at public API boundaries. Trust internal code.

### Logging
- Structured logging only (no string interpolation in log messages).
- Use `LoggerMessage` source generators for hot paths.

### Testing
- Every public behavior has a test. Test behavior, not implementation details.
- Unit tests use xUnit + NSubstitute (or project's chosen stack).
- Integration tests use `WebApplicationFactory`.
- Test naming: `MethodName_Scenario_ExpectedResult`.

---

## Architecture Decisions

<!-- Populated by /bootstrap — replaces separate ADR files -->
<!-- Format: Decision → Context → Consequences → Review notes -->

Record significant decisions here. Include accidental decisions that became convention.

---

## Common Tasks

### Add a new API endpoint end-to-end
1. Domain entity/value object (if new)
2. Application service method + interface
3. Request/response DTOs
4. FluentValidation validator for the request
5. Controller action (thin — delegates to service)
6. Unit tests for service logic
7. Integration test via WebApplicationFactory

### Add a new EF Core entity
1. Entity class in domain layer
2. Configuration class (`IEntityTypeConfiguration<T>`)
3. Add `DbSet<T>` to DbContext
4. Generate migration: `dotnet ef migrations add MigrationName`
5. Review generated migration SQL before applying

### Register a new service
1. Create interface + implementation
2. Add registration in the project's DI extension method
3. Inject via constructor — never resolve from `IServiceProvider` directly

---

## Boy Scout Rule

When touching any file, apply these improvements in priority order if they exist:

1. Add missing `CancellationToken` propagation
2. Replace string-interpolated log messages with structured logging
3. Add missing null checks at public boundaries
4. Add missing `.AsNoTracking()` to read-only queries
5. Split fat methods (>30 lines) into focused private methods
6. Add missing unit tests for public methods you're modifying

This is mandatory on every file you modify during normal development.

**When to skip**: hotfixes, time-sensitive production incidents, and proof-of-concept branches. If skipping, add a comment `// TODO: Boy Scout skipped — [reason]` so it's picked up on the next pass. Use `/debt` to clean up later.

---

## Agentic Workflow

When given any task, follow this execution model:

### 1. Classify the intent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andreoucostas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
