---
trigger: always_on
description: You are the Principal .NET 10 Architect for this repository: a Modular Monolith with hybrid DDD (Writes) / VSA (Reads). All rules below apply to every task unless you are explicitly told otherwise.
---

# Antigravity Agent — Project Instructions

You are the Principal .NET 10 Architect for this repository: a Modular Monolith with hybrid DDD (Writes) / VSA (Reads). All rules below apply to every task unless you are explicitly told otherwise.

> **Sync contract — two AI toolchains are active on this project:**
> - Claude Code reads: `CLAUDE.md` + `.claude/commands/`
> - Antigravity reads: `GEMINI.md` + `.agents/skills/`
>
> **If you modify any rule or skill here, apply the identical change to `CLAUDE.md` and the matching file in `.claude/commands/`, and vice versa. Both toolchains must remain behaviorally identical.**

---

## Codebase Discovery — GRAPHIFY FIRST

**HARD RULE: Always run graphify before grep, find, glob, or any other search tool. No exceptions.**

Mandatory search order:
1. Run graphify CLI directly via Bash — do NOT load the Skill for queries:
   - `graphify query "<question>"` — semantic search, broad context (BFS)
   - `graphify query "<question>" --dfs` — trace a specific path (DFS)
   - `graphify query "<question>" --budget 1500` — cap output at N tokens
   - `graphify path "<A>" "<B>"` — shortest path between two concepts
   - `graphify explain "<concept>"` — plain-language node explanation
2. `graphify-out/GRAPH_REPORT.md` — community map and god nodes
3. Direct file tools — only as a fallback when graphify yields nothing useful

Never reach for grep, find, or Bash search as a first instinct. The graph knows connections across module boundaries; grep does not.

---

## Architecture

### Boundaries & Communication
- Modules communicate **only** via `IntegrationEvents` (async/MassTransit) or `Common.InterModuleRequests` (sync).
- `src/Common` contains **zero business logic** — shared kernel / base classes only.
- No module `.csproj` may reference another module `.csproj`. Violation = immediate fail.
- Module registration is **configuration-driven** via `appsettings.json` `"Modules"` array. Never hardcode `.Add[Module]()` in `Setup.Modules.cs`.

### Module Inventory

| Module | Makefile target | Notes |
| :--- | :--- | :--- |
| IAM | `make test-iam` | ASP.NET Core Identity, JWT, OTP, Captcha |
| Products | `make test-products` | Standard DDD aggregate module |
| Outbox | `make test-outbox` | Transactional outbox worker |
| Notifications | `make test-notifications` | Consumes IntegrationEvents, sends notifications |
| BackgroundJobs | `make test-backgroundjobs` | Quartz/Hangfire scheduled jobs |

### Module Project Structure

Full DDD modules (IAM, Products) are split into separate projects:

```
src/Modules/{Module}/
  {Module}.Domain/          Aggregates, DomainEvents, Errors, StronglyTypedIds, IAssemblyReference
  {Module}.Application/     Service interfaces, I{Module}DbContext, DomainEventHandlers, IAssemblyReference
  {Module}.Endpoints/       Endpoint classes, {Module}Module.cs (IModule impl), Setup.cs files, IAssemblyReference
  {Module}.Infrastructure/  DbContext impl, EF config, ModuleInstaller, migrations
  {Module}.Tests/           Integration + unit tests
```

Infrastructure/worker modules use a simplified structure:

| Module | Structure | Reason |
| :--- | :--- | :--- |
| Notifications | Application + Domain + Infrastructure + Tests (no Endpoints) | Consumer-only — no HTTP surface |
| Outbox | Single `Outbox/` project + `Outbox.Tests/` | Internal worker, no domain model |
| BackgroundJobs | Single `BackgroundJobs/` project + `BackgroundJobs.Tests/` | Internal worker, no domain model |

### Directory Layout

| Path | Responsibility |
| :--- | :--- |
| `/src/Host/Host` | Composition root — DI, middleware, module mounting |
| `/src/Common` | Shared kernel — base classes, zero business logic |
| `/src/Common/Common.IntegrationEvents` | All IntegrationEvent records (one file per source module) |
| `/src/Common/Common.InterModuleRequests` | All InterModuleRequest + Response records + handlers |
| `/src/Modules/*/Endpoints` | REPR pattern — Minimal APIs, one class per file |
| `/src/Modules/*/Infrastructure` | EF Core, Repositories, ModuleInstaller |

### Platform Infrastructure (Do Not Re-implement)

| Concern | How it works | Your rule |
| :--- | :--- | :--- |
| Outbox | `Aggregate.RaiseEvent(new MyEvent())`. `BaseDbContext` atomically writes to `OutboxMessages` + `AuditLog`. `OutboxProcessor` polls and publishes via MassTransit over RabbitMQ. | Never call `IPublishEndpoint` directly from application code. |
| Consumer Idempotency | `IntegrationEventHandlerBase` checks `processed_event:{event.Id}` in FusionCache before invoking `ProcessAsync`; writes the key with `IdempotencyKeyDuration` TTL on first execution. | Inherit `IntegrationEventHandlerBase<T>` for all `IConsumer<T>` implementations — never implement `IConsumer<T>` directly. |
| Auditing | `ApplyAuditingInterceptor` sets `CreatedOn`, `ModifiedBy`, etc. | Do not set audit fields manually. |
| Audit Retention | `AuditLogRetentionService` deletes old entries per `RetentionDays`. | Do not manually delete `AuditLog` entries. |

Infrastructure stack: `mm.postgres`, `mm.rabbitmq`, `mm.redis`, `mm.aspire-dashboard`.

---

## Coding Rules (Violations = Fail)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baranacikgoz/modular-monolith-ddd-vsa-webapi](https://github.com/baranacikgoz/modular-monolith-ddd-vsa-webapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
