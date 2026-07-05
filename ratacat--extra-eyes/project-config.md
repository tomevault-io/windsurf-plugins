---
trigger: always_on
description: **Extra Eyes** — a harness-agnostic CLI that puts extra pairs of eyes on a coding agent. Watcher agents silently observe the working agent's filesystem activity and conversation traffic, then deliver short, targeted messages back. Watchers never edit code.
---

## Project
**Extra Eyes** — a harness-agnostic CLI that puts extra pairs of eyes on a coding agent. Watcher agents silently observe the working agent's filesystem activity and conversation traffic, then deliver short, targeted messages back. Watchers never edit code.

## Vocabulary
- **Working agent** — the primary AI inside a coding harness, doing the work with a human in the loop. v1 first-party harnesses: **Claude Code**, **Codex CLI**, **pi**. Others (Cursor, Aider, Cline, Continue, Goose, …) supported via the universal file fallback.
- **Working context** — the state of the work fed to watchers: file changes, git diffs, and conversation traffic (user prompts + working-agent outputs).
- **Watcher agent** — a secondary AI invoked headlessly by `eyes`, watching the working context and posting messages. Never edits code. Multiple watchers may run in parallel; each receives the full context fan-out.
- **Prompt** — the watcher's standing instructions (what to watch for: security, performance, drift from the plan, hidden assumptions, etc.).
- **Message** — what the watcher sends back to the working agent.
- **Tick** — one observation pass by a watcher.

## Watchers

A **watcher** is a named profile bundling everything needed to spawn a watcher agent:

```toml
# .eyes/watchers/harold.toml
name      = "harold"
default   = true              # at most one profile per project may set this
prompt    = """
You watch a coding agent's work for security mistakes...
"""
harness   = "claude-code"     # claude-code | codex | pi | (raw model)
model     = "claude-sonnet-4-6"
settings  = { thinking = "low", max_tokens_per_tick = 1500 }
```

- Profiles live under `.eyes/watchers/<name>.toml` (project, version-controlled) or `~/.eyes/watchers/<name>.toml` (user, personal). Project profiles win on name conflict.
- `eyes watch` with no argument starts the project's default watcher.
- `eyes watch <name>` starts the named watcher.
- Typical workflow: run `eyes` from the project folder before or after starting the working agent, then give that agent work in the same folder.
- Multiple watchers may run concurrently. Each receives the full context fan-out; each contributes its own messages, labeled with its name.
- A small set of bundled defaults ships with the binary (general code reviewer, security-leaning, plan-drift, etc.) — installable into a project with `eyes init`.

## Architecture

```
        ┌──────────┐    unix socket    ┌──────────┐
watcher │ watcher1 │ ─────────────────▶│          │
agents  │ watcher2 │ ─────────────────▶│  eyesd   │ ◀─── fswatch + git diff
        │ watcherN │ ─────────────────▶│ (daemon) │ ◀─── conversation feed
        └──────────┘                   └────┬─────┘
                                            │
                                            ├──▶ harness hook   (Claude Code, Codex)
                                            ├──▶ MCP tool       (any MCP-capable harness)
                                            └──▶ .eyes/inbox.md (universal fallback)
```

### Working agent → watcher (context fan-out)
- **File changes:** fswatch on the working tree + git diff snapshots. Universal, automatic.
- **Conversation traffic:** captured via harness hooks where available (Claude Code / Codex `UserPromptSubmit` + `Stop`), or via `eyes feed` calls from the working agent on harnesses without hooks.
- All watchers receive the same context. Fan-out happens inside the daemon.

### Multi-session caveat
Extra Eyes is currently project-scoped, not window-scoped. Multiple harness sessions in one repo may share conversation context and watcher message queues, so direct `@eyes` replies, quoted watcher notes, or assistant summaries can crosstalk or recursively bloat the context. Intended default remains one `eyesd` and one `eyes watch` per project; multiple working-agent sessions need explicit session/event routing before this is considered clean.

### Development binary caveat
Installed harness hooks may point at a persistent `eyes` binary such as `~/.cargo/bin/eyes`, while local development often runs a freshly rebuilt `target/debug/eyes` daemon. After changing IPC schemas, hook delivery, or daemon protocol, refresh the installed hook binary after rebuilding, or point hooks at the same rebuilt binary. A stale hook helper can fail before injection, for example by sending an older `FetchMessages` request shape to a newer daemon.

After any change that affects hook behavior, daemon protocol, project scoping, watcher delivery, or the `eyes watch` user path, install the rebuilt binary used by hooks, usually with `cargo install --path . --locked --force`, then restart affected `eyesd` daemons before handoff. Verification should use the persistent `eyes` on `PATH`, not only `target/debug/eyes`.

### Watcher → daemon
- Unix domain socket. Sub-millisecond delivery, large payloads fine.

### Daemon → working agent (delivery surfaces, layered)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ratacat/extra-eyes](https://github.com/ratacat/extra-eyes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
