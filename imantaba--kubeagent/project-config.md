---
trigger: always_on
description: generates a completion script from the command tree
---

# kubeagent — Project Notes for Claude

A read-only Kubernetes troubleshooting CLI written in Go. This is **also a
Go-learning project** for a developer who is new to Go (comes from Python, but
prefers Go explained from scratch — see "Learning companion" below).

## Build, test, run

- Go lives at `/usr/local/go/bin` — put it on PATH: `export PATH=$PATH:/usr/local/go/bin`
- Module: `github.com/imantaba/kubeagent` (Go 1.26)
- Build: `go build ./...`  (binary: `go build -o kubeagent .`)
- Test:  `go test ./...`
- Run:   `./kubeagent scan [--kubeconfig path] [--output text|json]`
- Or as a `kubectl` plugin (krew): `kubectl kubeagent scan …` — same binary,
  same flags. `invocationName` in `main.go` reads `argv[0]` so usage and error
  text name whichever spelling the user typed.

## Architecture

One-directional pipeline, one focused package per stage:

```
cluster (connect) → collect (list pods) → diagnose (Detector interface) → report (text/JSON)
```

Full design in [docs/design.md](docs/design.md); task-by-task build plan in
[docs/plan-v1.md](docs/plan-v1.md).

## Invariants (do not break)

- **READ-ONLY by default.** Only `List`/`Get`-style calls, EXCEPT the opt-in
  `--fix` remediation flag, whose writes are guard-railed (fixed allowlist,
  protected namespaces, per-action confirmation, re-verify) and never
  LLM-decided. Without `--fix`, kubeagent never creates, updates, patches, or
  deletes anything.
- **The CLI is a Cobra command tree in `internal/cli`**, one file per command;
  `main.go` holds only the `version` symbol the release workflow stamps with
  `-ldflags "-X main.version=<tag>"`. Flags are declared per command and never
  as persistent flags: `--kubeconfig` appears on eight commands, and three of the
  remaining ones deliberately do not accept it. pflag rejects the single-dash
  long-flag form the standard library accepted, so `internal/cli.Normalize`
  rewrites a leading `-longname` to `--longname` for names the target command
  registers — that shim is why command lines written against v0.72 and earlier
  keep working, and removing it is a breaking change. Every command sets
  `SilenceErrors` and `SilenceUsage`, so errors reach `Main`'s renderer and the
  exit codes stay kubeagent's own; validation lives in `RunE`, not in Cobra's
  `Args`/`MarkFlagsMutuallyExclusive` helpers, which would reword the messages.
- **`scan` runs its independent reads through a bounded worker pool**
  (`internal/parallel`, capped by `KUBEAGENT_SCAN_WORKERS`, 8 by default). The
  v1 "sequential, no goroutines" simplification is retired. Determinism is
  preserved by construction, not by discipline: no read closure touches shared
  state, each writes only its own destination, and a sequential block afterwards
  walks a fixed report order — so the rendered bytes are never a function of
  which read answered first. `internal/parallel` must never import
  `internal/remediate` or `internal/explain`. `internal/watch` is no longer the
  only documented long-lived-process exception: the `watch` daemon
  runs informers, a heartbeat ticker, and an HTTP server concurrently, and
  `kubeagent mcp` (`internal/mcp`) is a second long-lived server, serving MCP
  tool calls over stdio for as long as the client stays connected. Both remain
  **strictly read-only toward the cluster** (get/list/watch only; no writes)
  and make **no LLM calls**. `internal/mcp` must never import
  `internal/remediate` or `internal/explain` — there is no code path from the
  MCP server into a write or into a model call. One deliberate carve-out:
  `kubeagent mcp`'s eager startup connection check exits with an error naming
  the kubeconfig path and context on stderr — the operator's channel, read
  before the process ever starts serving — while the protocol stream and
  every tool result stay free of **kubeconfig paths and context names**. That
  is the whole of the promise: a tool result also carries API text — an event
  message, a container waiting message — and API text can contain a filesystem
  path the kubelet chose (typically under `/var/lib/kubelet/`), which is the
  cluster's own layout rather than the operator's workstation. kubeagent
  normalises that text through `internal/safetext.Line`; it does not filter
  paths out of it (see
  [website/docs/features/mcp.md](website/docs/features/mcp.md)). `kubeagent
  gate` (`internal/gate`, `internal/findings`, `internal/sarif`,
  `internal/rolloutwait`) is a third case, though it is not long-lived: it
  runs once and exits. It too is **read-only toward the cluster** (`get`/`list`
  only), makes **no LLM calls**, and must never import `internal/remediate` or
  `internal/explain` (see
  [website/docs/features/ci-gate.md](website/docs/features/ci-gate.md)).
  `kubeagent tui` (`internal/tui`) is a fourth case, a long-lived interactive
  process alongside the watch daemon and the MCP server, not a one-shot run
  like `gate`. It is **strictly read-only toward the cluster** (`get`/`list`
  only, not even `watch`), makes **no LLM calls**, and must never import
  `internal/remediate`, `internal/explain`, `internal/investigate`, or
  `internal/report` (see [website/docs/features/tui.md](website/docs/features/tui.md)).
  `kubeagent rbac` (`internal/rbacprofile`) is a fifth case: a one-shot, read-only command

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imantaba/kubeagent](https://github.com/imantaba/kubeagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
