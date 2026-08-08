---
trigger: always_on
description: This is a Go CLI/TUI project for `seshagy`, an agent-aware terminal dashboard that supports both tmux and [herdr](https://herdr.dev) as multiplexer backends. The command entry point is `cmd/seshagy/`. Core packages live under `internal/`: `config` for TOML config, `sessionmgr` for the `Multiplexer` backend abstraction (tmux/herdr/noop), tmux sessions and agent pane metadata + detection engine, `integrations` for hook/plugin installs, and `tui` for Bubble Tea UI state/rendering. Tests sit beside 
---

# Repository Guidelines

## Project Structure & Module Organization

This is a Go CLI/TUI project for `seshagy`, an agent-aware terminal dashboard that supports both tmux and [herdr](https://herdr.dev) as multiplexer backends. The command entry point is `cmd/seshagy/`. Core packages live under `internal/`: `config` for TOML config, `sessionmgr` for the `Multiplexer` backend abstraction (tmux/herdr/noop), tmux sessions and agent pane metadata + detection engine, `integrations` for hook/plugin installs, and `tui` for Bubble Tea UI state/rendering. Tests sit beside code as `*_test.go` files. The active backend is auto-detected from the environment (`$HERDR_ENV=1` → herdr wins; else `$TMUX` → tmux; else noop).

### Agent detection subsystem

The agent-state-detection subsystem lives in `internal/sessionmgr/`:

- `agents.go` — pane scan format (`@seshagy_agent_*` fields), `ParseAgents`, `detectAgent`/`detectAgentName` (process-name table + arch-suffix + descendant walk for node-based agents), `NormalizeAgentState`, `isStateFresh`.
- `agents_report.go` — `ReportAgent`/`ReleaseAgent` (seq strict-`>` + per-pane flock + tombstone release), `MarkAgentVisited`/`MarkActiveDoneAgentsIdle` (done→idle-on-visit), `ResolvePaneByCwd` (cwd→pane unique-match for the OpenCode plugin).
- `agents_capture.go` — `CaptureAgentPane`, `ApplyManifestFallback` (the capture-pane screen-rule backstop; Tier A/B authority gate via `integrations.LifecycleAuthorityFor`).
- `manifest.go` — manifest TOML schema, compiler, classifier (`detectManifest`), regex normalization, gate matcher. The manifest classifier itself emits only idle/working/blocked/done; the broader `AgentState` enum adds `unknown` for herdr's `agent_status` wire value (undetected).
- `mux.go` — `Multiplexer` interface, `Terms` (terminology), `BackendKind`, `Detect`/`DetectFromEnv` (env-based backend selection: herdr wins over tmux).
- `tmux_backend.go` / `herdr_backend.go` / `noop_backend.go` — backend implementations. Under herdr, agent-state writes (`ReportAgent`/`ReleaseAgent`/`MarkAgentVisited`/`MarkActiveDoneAgentsIdle`) and the capture-pane manifest backstop are no-ops — herdr owns detection.
- `herdr_parse.go` — JSON parsers for herdr CLI output (workspace/pane/agent payloads); ids are treated as opaque strings.
- `completion.go` — read-only one-call pane/session snapshots used only by shell completion; never route it through full source/agent loading.
- `manifest_regions.go` — region slice helpers (whole_recent, osc_title, bottom_lines(N), bottom_non_empty_lines(N), after_last_prompt_marker, after_last_horizontal_rule, prompt_box_body, osc_progress).
- `manifest_update.go` — launch-time async fetch of manifests from the herdr public catalog; local-override > cached-remote > bundled precedence; version-guarded; HTTPS-only.
- `proctree.go` — process-tree descendant walk (node-agent discovery).
- `flock_{unix,windows}.go` — per-pane flock via `x/sys/unix.Flock` (cgo-free).

Bundled manifests: `internal/sessionmgr/manifests/*.toml` (offline fallback; hot-updated from herdr.dev at runtime).

Integrations live in `internal/integrations/`:

- `install.go` — registry (`Available`: pi, codex, claude, droid, opencode), `Install`/`Uninstall`, shell-hook JSON merge (idempotent, preserves user + herdr entries), `LifecycleAuthority` per integration.
- `assets/seshagy-agent-state.sh` — shared TMUX-gated best-effort hook script (codex/claude/droid).
- `assets/seshagy-agent-state.ts` — Pi TypeScript extension.
- `assets/seshagy-opencode-plugin.ts` — OpenCode TS plugin (session.idle/permission.ask/tool.execute).

## Build, Test, and Development Commands

- `mise run verify`: runs CI checks (`fmt:check`, `lint`, `vet`, `test`, `build`).
- `mise run fmt`: formats Go and YAML files using the configured formatters.
- `mise run vuln`: runs `govulncheck ./...`; GitHub CI runs this as a separate gate.
- `mise run release:check`: validates `.goreleaser.yml` without publishing.
- `make build`: builds the local `./seshagy` binary from `./cmd/seshagy`.
- `go run ./cmd/seshagy`: runs the TUI from the checkout.

Go 1.26 is in `go.mod`. Runtime behavior expects a multiplexer (`tmux` or `herdr`); optional tools include `zoxide`, `fd`, `yazi`, and `eza`.

## Coding Style & Naming Conventions

Use `mise run fmt` before submitting changes. Formatting uses `golangci-lint fmt`, `gofumpt`, `goimports`, `gci`, `golines`, and `yamlfmt`. Prefer small package-local helpers and existing domain terms: sessions, panes, agents, integrations, sources, and launch state. Export identifiers only when used across packages or by command-facing code.

## CLI output


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lmilojevicc/seshagy](https://github.com/lmilojevicc/seshagy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
