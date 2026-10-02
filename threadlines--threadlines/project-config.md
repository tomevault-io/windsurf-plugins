---
trigger: always_on
description: Threadlines is a minimal web GUI for using coding agents. A Node WebSocket server wraps provider CLIs (Codex, Claude) and serves web and desktop clients. Codex and Claude are the supported providers.
---

# Threadlines

Threadlines is a minimal web GUI for using coding agents. A Node WebSocket server wraps provider CLIs (Codex, Claude) and serves web and desktop clients. Codex and Claude are the supported providers.

## What Threadlines cares about

1. **Reliability and correctness first.** Behavior stays predictable under load and during failures (session restarts, reconnects, partial streams). If a tradeoff is required, choose correctness and robustness over short-term convenience.
2. **Performance is a close second.** Users drive agents all day and notice a dropped frame, a lying spinner, and a stale label.
3. **Dense, flat design.** Structure comes from typography and spacing, not boxes and shadows. See the Design System section before touching any user-facing surface.
4. **Long-term maintainability.** If you add new functionality, first check if there is shared logic that can be extracted to a separate module. Duplicate logic across multiple files is a code smell. Don't be afraid to change existing code. Don't take shortcuts by just adding local logic to solve a problem.

## A note from Will

Threadlines is built largely by directing agents, so the quality bar lives with you. Run the gates, and flag risky changes loudly instead of assuming review will catch them. Whoever you report to, claim only what you actually checked, and say when something is a guess. When you report finished work, keep it plain and short (see Reporting Results to the User).

I want Threadlines to be impressive, not just tidy. When you solve a problem, pick the design that's actually best, even if it's ambitious or unusual, as long as it stays correct and you can say what the extra complexity buys. Complexity that buys nothing is still a cost. Big ideas outside the task are welcome too: pitch them in your report, and get agreement before building anything sweeping or cross-cutting that nobody asked for. The core architecture (event-sourced orchestration, provider drivers, schema-only contracts) is established.

Treat this file as good defaults, except the five ways to hurt yourself below, which are hard rules. The developer's direct requests override anything here, hard rules included: do what they asked and name the conflict in your report. Direct means they asked for that specific thing, not that you inferred they'd be fine with it. If a rule fights the task and nobody asked for the exception, say so and get sign-off before breaking it.

## A small glossary

Use this language when communicating:

- **you** means the agent reading this file and changing Threadlines.
- **we, us, and maintainers** mean Will and the other people building Threadlines. These are who you are talking to now.
- **developer** means the maintainer directing your current task.
- **user** means a person using Threadlines to direct coding agents.
- **agent** means the coding agent a user runs inside Threadlines. Depending on context, that may also include you.
- **provider** means the agent runtime Threadlines talks to: Codex and Claude (native drivers), plus fx and Cursor (experimental, both ACP descriptors on the shared `apps/server/src/provider/acp/` core). An OpenCode driver exists but is not supported.
- **driver / adapter** means the server code wrapping one provider (`apps/server/src/provider/Drivers/`).
- **client** means the web or desktop UI.
- **environment** means one running Threadlines server and the machine, filesystem, provider credentials, and state it owns.
- **project** means an environment-local workspace record rooted at a directory.
- **thread** means the durable conversation and work history for a project.
- **turn** means one user-to-agent cycle, including follow-up work such as checkpointing.
- **Threadlines home** means the base data directory, `~/.threadlines` by default (`THREADLINES_HOME` overrides it). Live state sits in its `userdata` subfolder; dev-mode state sits in `dev`.

## The five ways to hurt yourself

1. **Killing by pattern.** Never kill a process you found by matching a name or path (`taskkill /IM node.exe`, `Get-Process node | Stop-Process`). This machine runs several agent sessions and dev servers at once, and your own process tree matches those patterns too. Kill only a PID you captured when you spawned the process.
2. **Writing to the live install.** `~/.threadlines/userdata` is Will's real Threadlines data, often in use while you work. Reading and copying from it are fine. Never start a server against it, never write to it, never clean it up.
3. **Running heavy suites in parallel.** Browser tests and full typecheck saturate this machine when two sessions run them at once, and the result is spurious timeouts, not real failures. Before starting one, check for a running vitest/tsc and wait if another session's suite is active.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Threadlines/threadlines](https://github.com/Threadlines/threadlines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
