---
trigger: always_on
description: Local-first Go automation agent. Module: `github.com/yyZe0122/yunmengze-agent`. Go **1.26+**.
---

# AGENTS.md

Local-first Go automation agent. Module: `github.com/yyZe0122/yunmengze-agent`. Go **1.26+**.

## Production shape

Only three production pieces:

- `ymzd` — long-running daemon (composition root: `cmd/ymzd`)
- `ymz` — local CLI client of the gateway (`cmd/ymz`); no-arg and `tui` open the TUI; TUI/`run` ensure a unique daemon via `internal/daemonctl` (`ymz start|stop|restart|status` or `ymz daemon …`)
- `core.db` — single SQLite source of truth (`modernc.org/sqlite`, pure Go; builds use `CGO_ENABLED=0`)

Do **not** restore deleted architecture: Module Runtime/Supervisor, out-of-process Memory/Scheduler/Evolution/Echo, `skills.db` / `scheduler.db`, `/v1/modules`, multi-DB, ORM, container DI, or generic event-bus frameworks.

## Commands

| Goal | Linux/macOS | Windows |
| --- | --- | --- |
| Format | `make format` | `.\scripts\dev.ps1 -Action format` |
| Check (fmt + vet + test [+ systemd unit]) | `make check` | `.\scripts\dev.ps1 -Action check` |
| Package VS Code GUI+TUI VSIX (optional; Node) | `make vscode` | — |
| Build → `bin/` | `make build` | `.\scripts\dev.ps1 -Action build` |
| Install to PATH | `make install` → `~/.local/bin` | `.\scripts\dev.ps1 -Action install` |
| check + build + daemon `--check` | `make all` | `.\scripts\dev.ps1 -Action all` |
| Uninstall from BINDIR | `make uninstall` | `.\scripts\dev.ps1 -Action uninstall` |
| Clean `bin/` `dist/` + go cache | `make clean` | — |

```bash
go test ./... -count=1
go test ./internal/<pkg>/ -count=1
go test ./internal/<pkg>/ -run TestName -count=1
```

Dependency edits: `go mod tidy && go mod verify` and keep `go.mod`/`go.sum` clean (CI fails on drift).

Local release matrix: `goreleaser release --snapshot --clean --parallelism 1`.

**Publish (root only on this host):** batch-commit the dirty tree by feature (never one-shot a multi-feature dump), write **`docs/history/changelog/vX.Y.Z.md`** (this file **is** the GitHub Release body; empty stub fails), reset `unreleased.md`, then  
`sudo -i && cd /home/yyze/projects/AutoZeAgent && ./scripts/publish-release.sh vX.Y.Z --yes`  
(`--commit-paths changelog` only for a leftover notes commit.) Every tag **must** attach `ymz-vscode_{version}.vsix` **and** `ymz-vscode-tui_{version}.vsix` (fail-closed; Node 18+). Full runbook: [`docs/release.md`](docs/release.md). Only `scripts/publish-release.sh` publishes — do not invent parallel scripts or steps.

`make check` on Linux also runs `scripts/check-systemd.sh` (no-ops on non-Linux / missing `systemd-analyze`).

## Layout that matters

| Path | Role |
| --- | --- |
| `cmd/ymzd` | Composition root: `main.go` (`run` + flags/defer) + `wire_{store,tools,chat,gateway}.go` + `adapters.go`. Same package; no DI container |
| `cmd/ymz` | Gateway client only — no tools, no provider, no grants; no-arg/`tui` → `internal/tui` |
| `internal/gateway` | Local HTTP/UDS server only (`api.go` + `handlers_*.go` + LocalRunner) |
| `internal/gatewayclient` | Shared CLI+TUI facade: HTTP/SSE transport + typed helpers (no import of gateway server) |
| `internal/tui` | Charm v2 TUI (ADR-053); Gateway-only; slash `cmds.go` + `cmds_*.go`; Elm `update.go` + `update_*.go`; **bubbles/v2** textarea + **lipgloss/v2** + **glamour/v2**; lipgloss 色块分区（无 Ultraviolet 整页 / lazy list）；header 1 行 + 底细线；status 在 editor 下；IME 硬件光标；slash 补全盖对话底；fold e/E/c；**no mouse grab** |
| `internal/kernel` | Session/Task/Run state machines (`model.go`) + repository (`repository.go` + `repository_{session,task}.go`) |
| `internal/tools` | Tool Broker (`broker.go`) + builtins; FS `fs.go` + `fs_{read,write,search,edit}.go`（含 `fs_remove`）；`process_exec` + `process_shell`; `taskkind.go` + `web.go` + `media.go`; nested `internal/executor` unimportable |
| `internal/architecture` | Import-boundary tests (ADR-022 / G4): pkg/gateway/TUI/CLI walls |
| `internal/chatsession` | Multi-turn chat for agent (build/write) and plan (read-only); workspace grants by mode |
| `internal/tasksubmission` · `taskcontrol` · `corequery` | Submit / pause·resume·cancel / read model (Gateway has no business `*sql.DB`) |
| `internal/providerruntime` | Main provider load, `/model`, hot-reload (ADR-048) |
| `internal/providerconfig` | ConfigDir JSON load + `chat.*` / MCP / role-map structs |
| `internal/modelcatalog` | Embedded models.dev snapshot + keyword lookup + ConfigDir cache refresh |
| `internal/configreload` | ConfigDir fsnotify debounce for provider files |
| `internal/agent` | Provider tool loop (ADR-052 R1–R5: observation; turn/step/next-step inbox) |
| `internal/contextpack` | Provider-view `ContextView.Build` + pack/compact; snapshots (ADR-041/051) |
| `internal/sessiontodo` · `internal/editrev` | Session todos (QE); file-edit checkpoints + rewind (QG) |
| `internal/memory` | In-process MemoryManager + `memory_entries` (ADR-044) |
| `internal/skillcatalog` · `skillmaintain` | File skills + draft/apply/usage (ADR-034/050) |
| `internal/toolpermission` | once/similar/permanent/deny gate (ADR-043) |
| `internal/userquestion` | `ask_user` pending + waiter + decide (ADR-052 R4) |
| `internal/modelresolve` | Job pin → session prefer → main (ADR-045) |
| `internal/injectscan` | Fail-closed scan before memory/skill system inject (H6-min) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yyZe0122/YunmengZe-Agent](https://github.com/yyZe0122/YunmengZe-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
