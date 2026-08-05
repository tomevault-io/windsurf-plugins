---
trigger: always_on
description: These guidelines describe **how this codebase is already written**, distilled into rules so
---

# Dev Machine Guard — Coding Guidelines

These guidelines describe **how this codebase is already written**, distilled into rules so
future work stays consistent. They are descriptive, not aspirational: every rule below is the
*dominant existing pattern*, with a canonical file you can copy from.

**Audience:** humans and AI agents making changes in this repo.

**How to use this doc.** Before adding a detector, a model field, a CLI flag, a scheduler tweak,
or any platform-specific code, find the matching section and follow the pattern. When in doubt,
open the cited "canonical" file and mirror it. Prefer consistency with the surrounding code over
personal preference.

**Scope.** This documents current conventions. It does **not** propose refactors. Genuinely
additive improvement ideas surfaced during analysis are quarantined in
[§17](#17-additive-improvement-backlog-future--not-part-of-these-guidelines) and are explicitly
*not* things to apply as part of adopting this doc.

> See also: [CONTRIBUTING.md](../CONTRIBUTING.md) · [adding-detections.md](adding-detections.md) ·
> [SCAN_COVERAGE.md](../SCAN_COVERAGE.md)

---

## 0. Prime directives

The ten rules that matter most. The rest of the doc expands on these.

1. **All OS interaction goes through `executor.Executor`.** Never call `os/exec`, and avoid raw
   `os.*` file/env access, in detector/scheduler/business logic. This is the seam that makes
   everything mockable and timeout-bounded. (§2.1, §7.1)
2. **A scan never aborts because one part failed.** Inventory detectors return values, not errors,
   and skip on failure. Only the top-level orchestrator returns an error. (§6.1)
3. **Detection is data-driven.** Add an entry to a spec table; don't add control flow. (§3.1)
4. **Cross-platform code splits by build tag**, not by sprinkling `runtime.GOOS` — *when* the code
   wouldn't compile on the other OS. Otherwise branch at runtime on `model.Platform*` constants. (§2)
5. **Every external command has an explicit timeout** via `RunWithTimeout`/`RunInDir`. Build commands
   from fixed argv; never interpolate untrusted input into a shell string. (§7)
6. **Raw secret values never get serialized.** Tag them `json:"-"`; emit a pre-redacted `Display`
   value plus a `SHA256` fingerprint, and redact in the detector. (§10)
7. **`model` is the single source of truth for every wire/output shape.** Explicit `snake_case`
   JSON tags; optional object → `*T,omitempty`; collection → bare slice (always `[]`). (§9)
8. **Logs go to stderr via `progress.Logger`; results go to stdout.** Pick the level by audience:
   `Warn` for "something expected was skipped", `Debug` for diagnostics. (§6.5)
9. **Concurrency is the exception.** Default to sequential code. When you must parallelize, bound it
   (`min(NumCPU, 8)`) with stdlib `sync` only. (§8)
10. **It must pass the gates:** `gofmt`, `go vet`, `go mod tidy` (no drift), `golangci-lint`,
    `go test -race`, `make smoke`, and `gosec` — on a `CGO_ENABLED=0` cross-compile to
    linux/darwin/windows. (§16)

---

## 1. Project shape & where code goes

- **Module:** `github.com/step-security/dev-machine-guard`, **Go 1.26** (pinned in `go.mod` and
  `.tool-versions`). Dependencies are deliberately minimal — stdlib first.
- **`CGO_ENABLED=0` everywhere.** The binary is pure-Go and cross-compiled. Do not introduce cgo or
  a dependency that requires it. Platform-native behavior comes from build tags + `golang.org/x/sys`,
  never cgo.
- **Two binaries** under `cmd/`:
  - `cmd/stepsecurity-dev-machine-guard` — the real agent/CLI. All product logic.
  - `cmd/stepsecurity-dev-machine-guard-task` — a Windows-only GUI-subsystem launcher
    (`-ldflags "-H windowsgui"`) whose only job is to give Task Scheduler a no-console parent.
    **Never add detection/CLI logic here.**
- **One responsibility per package.** Packages are small and noun-named after the thing they own.
  There is **no `utils`/`helpers` grab-bag** — resist creating one.
- **When to add a package vs a file.** Create a new `internal/` package only when the concern has its
  own OS-abstraction surface worth isolating at compile time (`winproc`, `tcc`, `lock`) or is a
  cohesive sub-domain with multiple files and its own doc (`detector/rules`, `detector/configaudit`).
  **Otherwise add a file to the existing package** — every detected tool is its own file in
  `internal/detector/` (`jetbrains.go`, `nodescan.go`, …), not its own package.
- **`doc.go`** is reserved for (a) a domain root with subpackages, or (b) a non-obvious engine whose
  trust/data-flow model deserves a long comment (`detector/rules/doc.go`). Ordinary leaf packages put
  the package comment atop the primary file.

The package map (own one responsibility each):

| Package | Owns |
|---|---|
| `model` | All shared data/wire types. **Dependency-free.** |
| `executor` | The `Executor` interface — every OS interaction (run cmd, file, env, user, GOOS). |
| `detector` | App/IDE/agent/package-manager/process detection. |
| `detector/rules` | Declarative, content-blind malicious-file scan engine. |
| `detector/configaudit` | Audits package-manager config files (npmrc, pip, yarn, pnpm, bun). |
| `cli` | Hand-rolled flag/verb parser → `cli.Config`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [step-security/dev-machine-guard](https://github.com/step-security/dev-machine-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
