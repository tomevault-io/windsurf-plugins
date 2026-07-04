---
trigger: always_on
description: **License**: Apache-2.0 · **Status**: Pre-1.0 (breaking changes permitted)
---

# RunWisp — Agent Directives

**License**: Apache-2.0 · **Status**: Pre-1.0 (breaking changes permitted)
**Stack**: Go 1.25 daemon, Svelte 5 (runes) + Tailwind UI, Bun workspaces + moon (`bunx moon`), embedded SQLite (`database/sql` + `modernc.org/sqlite`), AsyncAPI-defined optional control-plane protocol.

## 🎯 PRODUCT VISION (read this first — it outranks everything below)

RunWisp replaces **crond + supervisord** with one small Go binary that a single developer can drop on a VPS, a Raspberry Pi, or into a Docker image and immediately see *what ran, when, why it failed, and what it printed*.

**Prime directives** (in priority order; when they conflict, the higher one wins):

1. **Nothing silently fails.** Every run has an exit code, duration, timestamps, and captured output — persisted, browsable, and streamable. If a change makes failures invisible, reject it.
2. **One binary, zero runtime deps.** No Python, Node, external DB, systemd, or sidecars required to run RunWisp. SQLite and the web UI are *embedded*. Do not add runtime deps; prefer a vendored Go lib over a service.
3. **TOML is the sole source of truth.** `runwisp.toml` defines every task. The REST API and Web UI are **read-only + trigger** — they never mutate task definitions. Schema changes are user-visible breaking changes; treat the TOML surface as an API even pre-1.0. Never add a feature that *requires* the UI or API to configure.
4. **Local-first, offline-complete.** The daemon must work fully offline. Any network integration (`internal/cloud/`) is strictly optional — no feature may degrade when it's disabled or unreachable.
5. **Built for the individual and the small team.** Every core capability ships in the binary: scheduling, supervision, observability, web UI, TUI, REST. No artificial limits, no feature flags gating basics.
6. **Boring in prod.** Predictable resource use, graceful shutdown, recoverable state after crash or kill -9. Prefer a simple mechanism that's easy to reason about over a clever one that saves 5%.

## 🚫 NON-GOALS

- **DAGs / workflow orchestration** — that's Dagu/Airflow/Temporal. RunWisp tasks are independent units.
- **Clustering, leader election, HA failover, cross-instance coordination** — one daemon owns its tasks. Anything involving multiple daemons acting in concert is out of scope for the daemon itself; operators who want that can build it on top of the REST API / control-plane protocol.
- **Plugin systems / arbitrary extensibility** — the surface is TOML + shell commands + REST. No JS hooks, no Lua, no WASM.
- **Replacing the user's shell or package manager** — `run:` is a shell command the user already knows how to write. Don't invent a DSL on top of it.
- **Being a log aggregator** — we capture per-run stdout/stderr for visibility. We are not Loki, not ELK.
- **Enterprise identity systems** — CHAP + JWT answers "does this operator control this daemon?". SSO, directory integration, org/team modeling, fine-grained RBAC policies are outside the daemon's scope.
- **Long-horizon analytics / reporting** — retention is per-task and bounded. Anything that needs cross-task, cross-instance, or indefinite history lives outside the daemon.

When in doubt, ask: *"Does this help **one** operator run **their** tasks on **one** machine better?"* If no, it probably doesn't belong in `apps/runwisp`.

## 🧭 INVARIANTS (violating any of these is a bug, regardless of what a test says)

- **Supported platforms**: Linux, macOS, WSL. These are first-class — builds, tests, manual smoke. Native Windows is out of scope.
- **Config reload is explicit, never automatic.** The operator picks up TOML changes with `runwisp reload` (→ `POST /api/reload` over the local socket) or `SIGHUP`; both converge on one reconcile path (`internal/runtime/reconcile.go`). Reload is **validate-first / atomic**: the whole config is re-loaded and re-validated before any live state is touched, and on any error — parse failure, validation failure, or a change to a non-reloadable setting (`[daemon]`, `[scheduler] timezone`, `[storage]`, `[notify]`, the bind host/port) — the reload is rejected and the running task set is left exactly as it was. Reload adds/changes/removes tasks live but is **not** a restart: added tasks get no `run_on_start` and no catch-up, and in-flight runs finish under the definition they started with. No file-watchers, no auto-reload — the daemon only ever reloads when the operator asks.
- **Crash safety**: Killing the daemon (SIGKILL, power loss) must not corrupt state. On restart, any run that was in-flight is marked **interrupted** with a terminal status — it is **not resumed**.
- **Determinism of scheduling**: Given the same TOML + clock, the scheduler produces the same firings. Randomness, wall-clock reads, and FS I/O are injected, never called inline in scheduling logic.
- **No required network**: Daemon startup, task execution, UI serving, and TUI must all work with the NIC unplugged. Any outbound integration attempts happen in the background and never block the hot path.
- **Single writer per task**: Exactly one goroutine/run-manager owns a task's run lifecycle. Any other code observing state does so via `internal/events/` or read-only storage queries.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runwisp/runwisp](https://github.com/runwisp/runwisp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
