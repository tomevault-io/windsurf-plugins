---
trigger: always_on
description: > Applies to all Go services under `app-*/**` (the `cmd/` and `internal/` tree of every Go microservice in the Kentang Tech stack). Any AI coding agent (Claude Code, Cursor, Copilot, etc.) operating on a Go file in this repository MUST read this file before generating, editing, or reviewing code. A change that cannot satisfy these rules must be split or redesigned — do not bypass them.
---

# AGENT.md — Mandatory Rules for AI Coding Agents (Go)

> Applies to all Go services under `app-*/**` (the `cmd/` and `internal/` tree of every Go microservice in the Kentang Tech stack). Any AI coding agent (Claude Code, Cursor, Copilot, etc.) operating on a Go file in this repository MUST read this file before generating, editing, or reviewing code. A change that cannot satisfy these rules must be split or redesigned — do not bypass them.

**Stack:** Go (Golang) · Redis · PostgreSQL

**Version:** 1.0 · **Last Updated:** 2026-08-09 · **Status:** Active

---

## Before You Write Any Code

1. Check `WORKSPACE` (Graphify) for the target service's dependency graph.
2. Read `SYSTEM_MAP.md` — SSoT for topology, domain boundaries, data flows, async queues.
3. Read this file (`AGENT.md`) — coding standards for this codebase.

If the change alters domain boundaries, data structures, or service interactions, `SYSTEM_MAP.md` **must** be updated in the same PR. "I'll update the map later" is not acceptable.

---

## Enterprise Requirements

### 1.1 Source File Line Limit

Every new or modified `.go` source file under `app-*/internal/**` and `app-*/cmd/**` must stay under **250 lines**.

- [ ] Check line count before editing a file.
- [ ] Stop adding logic at ~200 lines — split before you're forced to.
- [ ] CI gate fails if any modified source file exceeds 250 lines.
- [ ] **Exception:** generated code (`*_gen.go`, `*.pb.go`, and any tool-generated output) is exempt — it isn't hand-authored and splitting it isn't meaningful. Never hand-edit generated files to dodge this rule.

```bash
find app-*/internal app-*/cmd -name '*.go' \
  -not -name '*_gen.go' -not -name '*.pb.go' -not -path '*/vendor/*' \
  -print0 | xargs -0 wc -l | sort -nr | head -50
```

**Rationale:** a file creeping toward 250 lines signals mixed concerns — schema, business logic, and I/O tangled together — not just "long code." Splitting before the limit is hit prevents last-minute refactors under deadline pressure.

---

### 1.2 Required Header Format

Every new/modified source file must carry this header (values illustrative):

```go
// Package settingsservice provides workspace settings backed by PostgreSQL.
//
// @file      internal/console/settings/service.go
// @for       Console workspace settings service backed by database rows.
// @uses      PostgreSQL connection pool, internal/console/settings/repository
// @reason    Provides scoped settings data to app-console via app-gateway.
// @author    Dodi Rusmana <rusmanadodi@kentangtech.com>
// @layer     service
// @stability stable
// @since     2026-08-09
package settingsservice
```

This is a per-file header rather than Go's typical package-level `doc.go` convention — a deliberate project convention (not idiomatic Go by default) that keeps authorship, purpose, and layer explicit and grep-able across every service.

- [ ] `@file` matches the actual relative path under `app-*/`.
- [ ] `@for` — one sentence, responsibility only, no implementation detail.
- [ ] `@uses` — major dependencies (external services, shared internal packages, infra primitives), not every import.
- [ ] `@reason` — why the file exists, not what it does.
- [ ] `@author` — stays `Dodi Rusmana <rusmanadodi@kentangtech.com>` across all files.
- [ ] `@layer` — one of: `schema | domain | repository | service | handler | router | worker | job | util | config`.
- [ ] `@stability` — `experimental | stable | deprecated`. `deprecated` requires `@deprecated-reason` + `@deprecated-since`.
- [ ] `@since` — ISO date first introduced; don't touch on later edits, git blame owns that.
- [ ] CI gate fails on any modified/new file missing a required tag.
- [ ] Exempt: generated code (same exception as 1.1).

---

### 1.3 Error Message Language

All error messages returned to clients MUST be in **English**.

- [ ] No hardcoded Indonesian (or any non-English) in client-facing error payloads, HTTP status text, or validation messages.
- [ ] Internal structured logs may stay in Indonesian for team communication — never leak into the client JSON.
- [ ] Errors returned over the wire must be structured, not free text:

```go
// apperror.Error is the only shape a handler may write to a client response body.
type Error struct {
	Code    string `json:"code"`    // machine-parseable, e.g. "WALLET_INSUFFICIENT_BALANCE"
	Message string `json:"message"` // English, human-readable
}

func (e *Error) Error() string { return e.Message }
```

Internally, wrap with `fmt.Errorf("charging wallet %s: %w", walletID, err)` for stack context — the wrapped chain never reaches the client directly; the `handler` layer maps it to an `apperror.Error` before writing the response.

---

### 1.4 Type Safety & Validation (Zero Tolerance)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rusmanadodi2598/pannelAI](https://github.com/rusmanadodi2598/pannelAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
