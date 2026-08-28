---
trigger: always_on
description: Build Kinevo as a maintainable, testable, explainable, offline-capable,
---

# Kinevo — AI Coding Agent Operating Contract

### Mission
Build Kinevo as a maintainable, testable, explainable, offline-capable,
open-source modular monolith while preserving the invariants defined by
`docs/SRS.md`, the architecture, and the test contracts. Never silently redefine
product requirements; never overwrite completed implementation merely to match
documentation.

### Normative source hierarchy
When documents disagree, use this order unless an approved ADR explicitly states
otherwise:
1. `docs/SRS.md`
2. `docs/architecture.md`
3. `docs/domain-model.md`
4. `docs/scheduling-engine.md`
5. `docs/knowledge-layer.md`
6. `docs/offline-sync.md`
7. `docs/ai-architecture.md`
8. `docs/api/openapi.yaml`
9. `database/migrations/`
10. `docs/design.md`
11. `docs/test-strategy.md`
12. implementation source
13. comments / temporary notes

`TASK.md` controls execution order/status only. It never overrides requirements.
Lower-level documents and existing code cannot redefine higher-level
requirements. If a conflict exists between implementation and requirements,
report it and determine the smallest safe correction — do not silently choose.

### Repository governance documents
Contributors and agents MUST respect the open-source governance files:
- `CONTRIBUTING.md` — contributor guide (branch/commit/PR rules, SRS/ADR/migration/API processes).
- `SECURITY.md` — vulnerability disclosure policy. Security issues are never
  reported in public issues.
- `docs/third-party/licenses.md` — license/provenance ledger. Check before
  copying external source.
- `.github/` — issue/PR templates, CI, Dependabot, CODEOWNERS.

### Mandatory agent behavior — before modifying code
The agent MUST:
- identify the relevant SRS requirement IDs (FR-xx / NFR-xx);
- inspect the affected sections of `docs/SRS.md`, `docs/domain-model.md`, and
  `docs/api/openapi.yaml`;
- locate the current domain/application/infrastructure implementation;
- inspect related tests (`server/tests/`);
- inspect relevant local `AGENTS.md` files if present;
- determine whether a migration or API/schema change is required;
- determine whether documentation must change;
- explain the intended impact internally and keep the patch bounded;
- avoid unrelated refactoring unless required for correctness.

### Mandatory agent behavior — after modifying code
The agent MUST:
- run the relevant tests (`composer test` in `server/`, or `make test`);
- run static analysis (`composer analyse`) and formatting/lint (`composer lint`);
- update API/schema contracts when they changed (`docs/api/openapi.yaml`,
  `database/migrations/`);
- update documentation when required;
- update `TASK.md` only when a task is actually completed or materially
  progressed;
- preserve migration safety;
- report unresolved assumptions or failures honestly.

### Pre-Commit Mandatory Verification Protocol
Before executing `git commit`, the agent MUST run and ensure ALL of the
following commands pass 100% without error:

1. `npm audit` — dependency security check.
2. `npm run typecheck` and `npm run build` — frontend typecheck and build.
3. `./vendor/bin/phpstan analyse` and `composer test` (or `make test`) — PHP
   static analysis and test suite.

It is FORBIDDEN to commit if any of the steps above results in a failing
status. If a step fails, fix the underlying issue first and re-run the full
protocol until everything is green before committing.

### Agent modes
#### Exploration mode
Read, search, map dependencies, identify invariants. No broad edits.

#### Implementation mode
One bounded vertical slice. Code + tests + required contracts.

#### Verification mode
Prove behavior. Never weaken a test merely to get green.

#### Refactoring mode
No intentional behavior change. Establish passing baseline first.

#### Architecture-change mode
Requires explicit impact analysis and an ADR (`docs/adr/`) unless already
covered by an existing approved decision.

### Forbidden shortcuts
- Do not create business logic in Vue components.
- Do not make controllers contain scheduling algorithms.
- Do not let Eloquent models become the entire domain model.
- Do not let the browser become authoritative for schedule state.
- Do not allow LLM output to bypass domain validation.
- Do not introduce a new dependency when an existing repository abstraction
  already solves the problem.
- Do not copy external application source code merely because it is convenient;
  check `docs/third-party/licenses.md` and `CONTRIBUTING.md` first.
- Do not expose secrets, note contents, AI prompts, or private document content
  in logs.
- Do not silently introduce a second source of truth.
- Do not silently drift the schema; any schema change requires a migration.
- Do not make undocumented API or architecture changes.
- Do not commit secrets, tokens, or private data.
- Do not change tests merely to make them pass.
- Do not declare a task done without evidence (test output, CI, commit).

### Domain implementation rule
```text
HTTP / UI
   ↓
Controller / Inertia endpoint
   ↓
Application use case
   ↓
Domain service / domain entity
   ↓
Repository / infrastructure
   ↓
PostgreSQL / storage
```

### Scheduling rule
Scheduling MUST be deterministic for the same inputs. AI may propose semantic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sedam-or/Kinevo](https://github.com/sedam-or/Kinevo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
