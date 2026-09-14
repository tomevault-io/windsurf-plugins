---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

Zoomies is a self-hosted GitHub Actions runner fleet controller: one Go binary
that watches for queued jobs, starts an ephemeral runner container per job, and
destroys it when the job finishes. SQLite for state, a Svelte 5 UI embedded with
`go:embed`, no Kubernetes and no database server.

`cmd/zoomies` is the only binary. It is the controller (`zoomies controller`),
the agent (`zoomies agent`), the installer (`zoomies init`) and the CLI, chosen
by subcommand. On a single VM the controller runs an agent inside itself, so the
whole system is one process and one SQLite file.

Read [docs/architecture.md](docs/architecture.md) before changing anything
structural — it explains the shape and the reasons for it.

## Commands

```sh
make build-nogui   # build without rebuilding the UI -- the fast inner loop
make build         # build the UI and embed it (needs npm)
make test          # go test -race -count=1 ./...
make lint          # go vet, gofmt check, staticcheck if present, UI lint
make fmt           # gofmt + prettier
make dev           # controller with auth off, on :8080, for UI work
make ui-dev        # Vite dev server proxying /api to that controller
make test-ui       # Playwright against a real built binary
make openapi       # regenerate the UI's TypeScript client from api/openapi.yaml
make screenshots   # recapture docs/screenshots from the real UI, both themes (needs Pillow)
```

`go build ./...` fails on a clean checkout: `internal/api` embeds
`internal/api/webdist`, which is a build product. Run `make build-nogui` once —
it writes a placeholder — before any Go command that compiles that package. CI
does the same thing as its first step.

Go 1.27.1 or later (`go.mod` sets the floor; CI builds with 1.27.1), Node 22 or
later. Node is a build-time dependency only; the shipped binary is
self-contained and static (`CGO_ENABLED=0`, pure-Go SQLite).

Run a single test with `go test -run TestName ./internal/pkg/`. `make test-e2e`
needs real GitHub credentials and skips itself without them.

## Package boundaries

These are load-bearing. Code that crosses them is the main thing to catch in
review.

| Package | Rule |
| --- | --- |
| `internal/store` | The **only** place SQL is written. Domain types, embedded migrations, every query. No other package imports `database/sql`. |
| `internal/scheduler` | **Pure.** `Decide` takes a snapshot and returns a `Plan`. No clock reads, no database, no network — that is what makes scaling behaviour testable, and it is where every decision's operator-facing *reason string* comes from. |
| `internal/api` | Transport only. A handler reads a request, asks the controller / auth / store, and renders the shape `api/openapi.yaml` promises. It has no opinions about the fleet. The resource views themselves (`HostView`, `PoolView`, …) are `internal/controller/views.go` types the handlers alias, because the event stream renders the same JSON and is fed from the controller. |
| `internal/provider` | The infrastructure-provider contract, a fake that obeys it, and `RunContractTests`, the conformance suite every provider passes. It may import `internal/store`'s domain types and `config.Finding` and **nothing else** -- two tests enforce that, and that it names nothing provider-specific. Renting a machine is not placing a runner: this is separate from `internal/backend` on purpose. |
| `internal/provider/proxmox` | The first provider. The Proxmox VE API is hand-rolled `net/http` for the same reason the Docker one is. |
| `internal/controller` | Wiring: the reconcile loop, the machine loop, webhook ingest, the agent task queue, the log relay, and every payload the event stream carries (`views.go`, `derived.go`). The machine loop is **not** part of `Reconcile`: a clone takes minutes and `reconcileMu` is held for a whole scheduling pass. |
| `internal/config` | `zoomies.yaml` + `ZOOMIES_*` overrides, and the validator. |
| `internal/github` | App auth, JIT configs, webhook validation, the fallback poller, and `fake.go`, a fake GitHub used by tests. |
| `internal/backend` | Docker, Podman, bare process. The Docker API is hand-rolled `net/http` against the Engine API on purpose (see below). |
| `internal/agent` | The runner-executing half and its outbound transport. |

Other invariants worth knowing before you edit:

* **One writer.** `store.Store` funnels writes through a single connection
  behind a mutex, with a separate pooled reader in WAL mode. Do not add a second
  writer; `database is locked` is designed out of this codebase.
* **The runner state machine is enforced in the store**, not the caller.
  `provisioning → registering → idle ⇄ busy → draining → removed`, plus
  `failed`. Go through `store.TransitionRunner`; an agent must not be able to
  report a nonsensical state and corrupt fleet accounting.
* **The controller never dials an agent.** Agents connect outbound only (long-poll
  for tasks, POST results), so a host behind NAT needs no inbound rule. That is
  why log streaming is inverted: the controller queues a `stream_logs` task and
  the agent opens a chunked POST that gets relayed to the browser's SSE stream.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eyupio/zoomies](https://github.com/eyupio/zoomies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
