---
trigger: always_on
description: Kocoro is the Go CLI/runtime for Shannon AI agents. The main production path is daemon + Kocoro Desktop + Shannon Cloud: the daemon connects to Cloud over WebSocket, receives channel messages, runs the agent loop locally with full tool access, and streams results back. It also supports interactive TUI, one-shot CLI, MCP server mode, and local scheduled tasks.
---

# Kocoro Project Guide

## What This Is

Kocoro is the Go CLI/runtime for Shannon AI agents. The main production path is daemon + Kocoro Desktop + Shannon Cloud: the daemon connects to Cloud over WebSocket, receives channel messages, runs the agent loop locally with full tool access, and streams results back. It also supports interactive TUI, one-shot CLI, MCP server mode, and local scheduled tasks.

## Working Rules

- Use the Go version declared in go.mod as source of truth.
- Prefer existing repo patterns over new abstractions. Keep changes small and directly tied to the task.
- Verify API response bodies, not just status codes.
- Do not create parallel `_enhanced` variants; update existing code in place.
- For risky behavior changes, preserve operator-visible flags, rollback paths, and focused tests.
- When touching dependency or generated-code surfaces, test locally before pushing.
- When adding small hardcoded caps, document the workload, binding symptom, and override path. Prefer config defaults for caps power users may need to lift.

## Module Map

- cmd: CLI entry points for one-shot, daemon, scheduling, update, and MCP serve.
- internal/daemon: primary production path; HTTP API server, WebSocket client, routing, approvals, events, launchd, attachments, session CWD, memory fallback, suggestions, email/password auth (`auth.go` / `auth_handlers.go` / `ws_controller.go`), Desktop RPC reverse-channel (`desktop_rpc/` subpackage — Unix sock listener + length-prefixed JSON codec + DesktopRPCBroker for Calendar RPC v1).
- internal/agent: core loop; tool batching, compaction, spill/budget state, deferred loading, state cache, read tracking, approvals, phase/watchdog, thinking handling, prompt suggestions, forked requests.
- internal/tools: local, gateway, cloud, schedule, publish/upload, image, memory, MCP, and document tools.
- internal/keychain: macOS Keychain wrapper for daemon api_key (Backend interface + osBackend/memBackend; non-darwin returns ErrUnsupportedPlatform).
- internal/client: gateway/SSE/Ollama clients plus AuthClient (`/api/v1/auth/*` REST wrapper).
- internal/session: session persistence, lifecycle, titles, and SQLite FTS index.
- internal/config: config loading, merging, settings, and setup.
- internal/skills: skill registry, bundled skills, marketplace install, provenance, secrets, validation.
- internal/memory: sidecar client/supervisor, bundle puller, tenant safety, audit, service orchestration.
- internal/sync: opt-in session sync; locking, markers, scanning, batching, upload, backoff.
- internal/mcp: MCP client/server and browser profile lifecycle.
- internal/permissions: command permission model and safety checks.
- internal/runstatus, audit, hooks, prompt, instructions, context, schedule, tui, update: supporting runtime surfaces.
- test/e2e contains offline and live E2E coverage.

## Critical Invariants

### Kocoro Skill Docs

The bundled `kocoro` skill is the AI-facing source of truth for daemon HTTP APIs, config fields, and workflows. Any new daemon endpoint, endpoint behavior change, config surface, or user-facing workflow must update the matching bundled skill reference in the same change. Missing references cause agents to invent API workarounds.

### Builtin Skills

Bundled skills are overlaid into the user skill directory on startup and user edits are overwritten. Fork under a new skill name to customize. The hidden generative UI skill emits `html-artifact` blocks for Desktop's sandboxed WKWebView; session-share pages render the same blocks in a sandboxed iframe via `internal/share/artifact.go` (host CSS/CSP/bridge mirrored verbatim from Desktop — see CLAUDE.md). Shared pages strip tool runs (prose + images only).

`internal/daemon/skill_filter.go` maintains a `desktopOnlySkills` registry. Daemon filters these out of the per-request skill list when `req.Source` is a cloud-distributed channel (Feishu / Lark / WeCom / Slack / LINE / Telegram / webhook), keeping the use_skill tool registry, scaffolded listing, and semantic discovery consistent. Filter is applied once on the producer side immediately after `LoadGlobalSkills` so all consumers see the same view. Drift test (`skill_filter_test.go`) walks the `desktopOnlySkills × cloudSourceSet` cross product.

### Agent Names

Agent names must match `^[a-z0-9][a-z0-9_-]{0,63}$`. Validate before path construction.

### Tool Registry

Tool priority is local tools, then MCP tools, then gateway tools. Deduplicate by name. Skill `allowed-tools` is enforced at execution time, not by schema filtering, to keep prompt-cache tool arrays stable.

Skill-exempt tools must be pure infrastructure with no external side effects. Do not exempt side-effecting tools from skill restrictions.

### Tool Concurrency


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kocoro-lab/Kocoro](https://github.com/Kocoro-lab/Kocoro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
