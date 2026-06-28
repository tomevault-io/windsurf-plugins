---
trigger: always_on
description: Rules for AI agents working on this codebase. These patterns were established through 60+ refactorings over 2 months. Violating them creates regressions.
---

# AGENTS.md — Stave CLI Codex

Rules for AI agents working on this codebase. These patterns were established through 60+ refactorings over 2 months. Violating them creates regressions.

## Canonical sources

If this file disagrees with one of these, the canonical doc wins — open an issue noting the drift:

- **Architecture:** [`docs/architecture/pkg-stave-facade.md`](docs/architecture/pkg-stave-facade.md) — the facade migration plan. `pkg/stave/` is the stable public API; both `cmd/` and `cmd/mcp/` consume it. `cmd/mcp/architecture_test.go` enforces zero `internal/` imports for the MCP server today; the CLI proper is mid-migration.
- **Build + testing:** [`TESTING.md`](TESTING.md) and the project [`CLAUDE.md`](../CLAUDE.md).
- **Goldens:** the regen-and-triage workflow at the top of `CLAUDE.md` (`make regenerate-goldens` with categorized diff).

## Architecture

Stave uses hexagonal architecture enforced by compile-time tests in `internal/app/architecture_core_isolation_test.go` and `internal/app/architecture_dependency_test.go`.

### Layer Rules

```
cmd/           CLI boundary: Cobra commands, flag parsing, dependency wiring
  |
  v
adapters/      Infrastructure: filesystem, YAML loading, JSON output, git
  |
  v
app/           Application services: orchestration, pipelines, workflows
  |
  v
core/          Domain model: ZERO external imports (no os, no fmt, no net)
```

### Do

- Place domain types in `internal/core/`. Domain types must have zero imports from `app/`, `adapters/`, `cmd/`, or `platform/`.
- Place application orchestration in `internal/app/`. App services import `core/` and `app/contracts` ports.
- Place infrastructure implementations in `internal/adapters/`. Adapters implement port interfaces from `app/contracts`.
- Wire adapters to app services in `cmd/` via `deps.go` files and `cmdutil/compose/infra.go`.
- Use `internal/platform/` for OS-level utilities (crypto, filesystem, logging, shell).

### Internal Encapsulation

- All complex security logic and data structures MUST reside in `internal/`.
- The `cmd/` package is a "Thin Adapter." It handles only flag parsing and calling `app/` or `core/` services.
- If a domain model, evaluation type, or security logic is proposed for `cmd/`, it is a hexagonal violation. Move it to `core/` or `app/`.
- This prevents "Logic Leakage" and keeps the binary's entry point clean and auditable.

### Don't

- Import `adapters/` from `app/` or `core/`. This is the #1 hexagonal violation.
- Import `cobra`, `os.Exit`, `fmt.Printf`, `fmt.Println`, `os.Stderr`, `os.Stdout` from `core/` or `app/`.
- Import `net/http`, `os/exec`, `database/sql`, or `flag` from test files in `core/` or `app/`.
- Add internal-only domain code to `pkg/`. New domain code goes under `internal/`. The `pkg/stave/` facade is the **exception**, not the rule: it re-exports a narrow public API the CLI and MCP server consume. New capability lands in `internal/app/` first, then surfaces through `pkg/stave/` if it belongs in the public contract. See [`docs/architecture/pkg-stave-facade.md`](docs/architecture/pkg-stave-facade.md).
- Add pass-through "workflow" or "bridge" packages between layers. If a package only forwards calls, delete it.
- Move domain models, evaluation types, or security logic into `cmd/`. This is logic leakage.

## Command Structure

Every CLI command follows this file convention:

| File | Purpose |
|------|---------|
| `cmd.go` | Command construction, Cobra registration, `Long` help text |
| `run.go` | Execution logic (thin: construct request, call app service, format output) |
| `options.go` | Flag struct + `Prepare(cmd)` method (resolve defaults, normalize, validate) |
| `output.go` | Rendering (text, JSON, SARIF) |
| `deps.go` | Dependency wiring |

### Do

- Put all flag validation and config resolution in `PreRunE` via `opts.Prepare(cmd)`.
- Keep `RunE` thin — delegate to `app/` or `core/` services.
- Set `SilenceUsage: true` and `SilenceErrors: true` on every command.
- Include purpose, inputs, outputs, exit codes, and examples in the `Long` help string.
- Use stable flag names: `--controls/-i`, `--observations/-o`, `--format/-f`, `--now`, `--quiet`, `--sanitize`.

### Don't

- Parse flags in `RunE`. All flag parsing belongs in `PreRunE`.
- Import domain types in `cmd.go`. The `cmd.go` file constructs the command; `run.go` uses domain types.
- Create new subcommands when a flag mode suffices (e.g., `apply --dry-run` not `plan`).
- Skip `Prepare(cmd)` and put validation inline in RunE.

## Domain Vocabulary

These terms are final. The renames are done. Use the canonical term.

| Canonical Term | Rejected Alternatives (never use) |
|---|---|
| `control` | invariant, rule, check |
| `asset` | resource |
| `finding` | issue, violation, result |
| `exemption` | ignore, suppress, skip |
| `sanitize` | redact, scrub |
| `drift` | delta, change |
| `ExposureWindow` | Episode |
| `ExposureLifecycle` | Timeline |
| `AuditScope` | ScopeFilter |
| `Assessor` | Runner (engine) |
| `PolicyInspector` | PackRunner |
| `GovernanceResolver` | Evaluator |
| `DiagnosticSuite` | CheckSuite |
| `AuditWorkflow` | EvaluateRun |
| `DiagnosticEngine` | Run (diagnose) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sufield/stave](https://github.com/sufield/stave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
