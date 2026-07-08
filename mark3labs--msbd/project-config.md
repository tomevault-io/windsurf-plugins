---
trigger: always_on
description: **msbd** is a small Go HTTP server that wraps the [microsandbox](https://github.com/superradcompany/microsandbox) Go SDK (`github.com/superradcompany/microsandbox/sdk/go`) and exposes a REST API for managing fast, local microVMs. It exists so that long-running applications can drive microsandbox without linking libkrun / cgo themselves: msbd quarantines all of that to one binary on one KVM-equipped host, and everything else talks plain HTTP.
---

# AGENTS.md

## Project overview

**msbd** is a small Go HTTP server that wraps the [microsandbox](https://github.com/superradcompany/microsandbox) Go SDK (`github.com/superradcompany/microsandbox/sdk/go`) and exposes a REST API for managing fast, local microVMs. It exists so that long-running applications can drive microsandbox without linking libkrun / cgo themselves: msbd quarantines all of that to one binary on one KVM-equipped host, and everything else talks plain HTTP.

Module path: `github.com/mark3labs/msbd`.

## How it's wired up

```
cmd/msbd/main.go         entrypoint: fang/cobra CLI → serve cmd → loadConfig →
                         EnsureInstalled → core.NewService → svc.Reconcile →
                         api.NewServer → ListenAndServe (graceful drain on signal)

internal/core/           SDK-facing business logic. EVERY call to the
                         microsandbox SDK happens here (and only here).
                         The api/ package never imports the SDK.

internal/api/            HTTP surface. Routes, middleware (bearer auth,
                         panic recover, request log), DTOs that mirror
                         the value types in core/.

openapi.yaml             the wire contract. Source of truth for client
                         generators and reviewers. Embedded into the binary
                         via assets.go (//go:embed) and served at /openapi.yaml
                         + /docs (Swagger UI).
```

The two-package split (`api` ↔ `core`) is the boundary that keeps DTO churn from leaking into business logic and vice versa.

## Layout

- **`cmd/msbd/main.go`** — cobra CLI styled with `charmbracelet/fang`. The root command defaults to (and also exposes) a `serve` subcommand whose flags mirror the `MSBD_*` env vars (flag › env › default). `serve` does `msb.EnsureInstalled` (downloads `msb` + `libkrunfw` into `~/.microsandbox/` on first run), startup reconcile, then HTTP serve with graceful shutdown on Ctrl-C / SIGTERM. Also defines the `/readyz` probe (FFI loaded + `/dev/kvm` openable r/w).
- **`assets.go`** (module root) — `//go:embed openapi.yaml` into `OpenAPISpec`. Lives at the root because `go:embed` can't reference a parent directory from `internal/api`. `main.go` hands the bytes to `Server.SetOpenAPI`.
- **`internal/core/service.go`** — `Service` is the single owner of all SDK calls: lifecycle (`Create`/`Get`/`Inspect`/`List`/`Stop`/`Start`/`Delete`), exec (`Exec`/`Run`), jobs (`Launch`/`Poll` + `WriteJobStdin`/`CloseJobStdin`/`SignalJob`), file IO (`ReadFile`/`WriteFile`). Provider-neutral input/output types (`CreateParams`, `Instance`, `ExecParams`, `ExecResult`).
- **`internal/core/terminal.go`** — interactive terminal sessions (`OpenTerminal`). Returns a transport-agnostic `Session` interface (`Output`/`Write`/`Resize`/`Signal`/`Close`/`Wait`); goes through `resolve()` like `Run`, then hands off to the agent-PTY backend. In-memory only.
- **`internal/core/terminal_agent.go`** — the **real kernel-PTY** backend. Drives the microsandbox agent protocol directly (`ConnectAgentSandbox` + `AgentClient.Stream`/`Send`/`Next`) with hand-rolled CBOR frames (`fxamacker/cbor`), replicating what the SDK's `Attach` does but sourcing stdin from the WebSocket instead of a local TTY. Sends `core.exec.request{tty:true,rows,cols}` and relays `core.exec.stdin`/`resize`/`signal` ↔ `core.exec.stdout`/`stderr`/`exited`. **The wire schema (protocol v5) is reverse-engineered from upstream Rust, NOT a public SDK API** — a microsandbox protocol bump can break this file. Constants in this file mirror `crates/protocol/lib`.
- **`internal/core/fs.go`** — extended filesystem ops over `sb.FS()`: `ListDir`/`Stat`/`Exists`/`Mkdir`/`Remove`/`Copy`/`Rename` plus host transfer (`CopyFromHost`/`CopyToHost`). All route through `resolve()`.
- **`internal/core/metrics.go`** — `Metrics(id)` and `AllMetrics()` point-in-time resource snapshots.
- **`internal/core/logs.go`** — `Logs(id, LogQuery)` reads persisted stdout/stderr/output/system logs with tail + source filters.
- **`internal/core/volume.go`** — named persistent volumes (`CreateVolume`/`ListVolumes`/`GetVolume`/`RemoveVolume`) and volume file IO. Volumes are independent of sandboxes (not cached in `Registry`); mount them at create via `CreateParams.Mounts`.
- **`internal/core/image.go`** — cached OCI image inventory (`ListImages`/`InspectImage`/`RemoveImage`/`PruneImages`) over the SDK `msb.Image` factory.
- **`internal/core/snapshot.go`** — sandbox rootfs snapshots over the `msb.Snapshot` factory (`Create`/`List`/`Get`/`Verify`/`Remove`/`Export`/`Import`/`Reindex`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mark3labs/msbd](https://github.com/mark3labs/msbd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
