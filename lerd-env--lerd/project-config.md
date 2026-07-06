---
trigger: always_on
description: You are a coding agent working on **lerd**: an open-source, Herd-like local PHP
---

# CLAUDE.md — agent guide for the lerd codebase

You are a coding agent working on **lerd**: an open-source, Herd-like local PHP
development environment for Linux and macOS (Windows via WSL2, beta). It runs
Nginx, PHP-FPM, and services as rootless Podman containers, and ships a built-in
Svelte web UI, a TUI, a CLI, and an MCP server. No Docker, no sudo, no system
pollution.

This file is loaded into every session. Read it before you touch anything, then
follow it exactly. It overrides your defaults. When a rule here conflicts with a
habit, the rule wins.

---

## 1. The three design laws

Almost every mistake an agent makes here is a violation of one of these. Check
your change against all three before writing code.

1. **Store-first, never hardcode.** lerd ships only the default stack. Every
   framework lives in `lerd-frameworks/` and every service in `lerd-services/`,
   as versioned YAML. If a change adds a framework, a service, a worker, an env
   wiring, a doctor check, a custom command, or a proxy, it belongs in a store
   YAML file, **not** in Go. New workers go in the framework store YAML — never
   in hardcoded Go, and never add a merger that backfills them. Copy the closest
   existing YAML; those files are the schema of record.

2. **Framework-agnostic.** No feature may know the name "Laravel" (or Symfony,
   WordPress…) in Go. Behaviour is driven by the YAML definition. If you find
   yourself branching on a framework name in Go, the logic is in the wrong layer
   — move it into the store as declarative data.

3. **Env belongs to sites, not workers.** All environment variables live in the
   site's `.env`. Workers never declare env vars. Service presets inject their
   host/port/credentials into the site `.env` via the framework's `env.services`
   mapping.

---

## 2. Where things live

```
cmd/lerd            CLI + long-running lerd-ui/watcher entrypoints
cmd/lerd-tray       system tray (CGO, libayatana-appindicator)
internal/           all Go application code, one package per concern:
  podman/           quadlet generation, container lifecycle
  nginx/ certs/ dns/  site serving, TLS, .test resolution
  services/ serviceops/  service-preset engine + operations
  store/ registry/  fetch + cache the lerd-frameworks / lerd-services stores
  siteops/ siteinfo/ grouping/  site linking, groups, worktrees
  worker*/ idle/    workers, self-heal, idle-suspend
  sitedoctor/       framework-agnostic health checks
  mcp/              MCP server (tools AI assistants call)
  tui/              terminal dashboard
  ui/web/           Svelte web UI (built + //go:embed'd into the binary)
lerd-frameworks/    local checkout of the framework store for dev; submit to lerd-env/frameworks
lerd-services/      local checkout of the service store for dev; submit to lerd-env/services
docs/               mkdocs site published to lerd.sh; docs/contributing has the human guide
tests/installer/    bats tests for install.sh
```

The web UI is Svelte under `internal/ui/web/`, built to `dist/` and embedded via
`//go:embed`. The Go binary is self-contained; `make build` builds the UI first.

---

## 3. The contribution lifecycle — follow every step, in order

This is the full life cycle of a change. Do not skip steps. Do not reorder them.

### Step 0 — Open an issue first
Before writing code, an issue should exist for the work. Frame it as future work
(what will be done), not as already done. One issue per unit of work. Do not
create GitHub issues, comments, or PRs without explicit approval — draft the text,
show it, and wait. (See §7.)

### Step 1 — Understand before you build
Read the surrounding package and the closest existing example. Match its naming,
comment density, and idioms. Decide which layer the change belongs to using the
three laws in §1. If it's store data, you're editing YAML, not Go.

### Step 2 — Write the test first (TDD)
New functionality **must** include tests, and behaviour changes must update them.
A PR without corresponding test coverage will not merge. Write the failing test,
then make it pass. Keep tests out of `/tmp` — those get wiped; use a repo fixture.

### Step 3 — Implement (DRY, KISS)
Reuse existing patterns and helpers; do not duplicate. In the web UI, extract
shared markup into components from the start — never copy markup between views.
Pick the simplest design that satisfies the issue. Keep code comments minimal:
explain only what isn't self-evident, and keep any comment block to 2-3 lines at most.

### Step 4 — Document it
Update the relevant page under `docs/` for every feature or behaviour change,
**before** committing. If the change touches what AI assistants can do, update the
MCP skills and guidelines alongside it.

### Step 5 — Run the full local gate (see §4)
Build, test, vet, gofmt, UI tests, and installer tests must all pass locally —
not just at release time. Then install the local build and smoke-test the real
app; don't rely on tests alone for anything with a runtime surface.

### Step 6 — Commit only when asked (see §6)
Do not commit, push, or open a PR automatically. Wait for an explicit go-ahead,
and pause at phase boundaries of a staged change so a human can smoke-test in the
browser before the next phase.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lerd-env/lerd](https://github.com/lerd-env/lerd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
