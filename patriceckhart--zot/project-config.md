---
trigger: always_on
description: This file defines how automated coding assistants should work in this repository. Treat it as a practical operating manual, not a substitute for reading the code.
---

# Working Agreement for zot

This file defines how automated coding assistants should work in this repository. Treat it as a practical operating manual, not a substitute for reading the code.

## Product intent

zot is a compact Go coding-agent harness. Changes should preserve its defining properties:

- one portable binary
- a provider-neutral agent engine
- a terminal UI without a framework dependency
- extensions that communicate through subprocess JSON-RPC
- predictable behavior on macOS, Linux, and Windows
- a small dependency and operational footprint

Prefer a narrow, explicit implementation over a generalized subsystem. New abstractions must earn their cost through a real boundary or repeated use.

## Starting a task

Build context before editing:

1. Inspect `git status --short`. Existing modifications may belong to the user or another agent.
2. Locate and read every `AGENTS.md` that governs the target path.
3. Read the owning implementation, nearby tests, and any user-facing documentation for the behavior.
4. Reproduce reported failures when feasible. Record expected behavior separately from observed behavior.
5. For GitHub work, inspect the issue or pull request with `gh` if it is available. Do not change branches merely to review a pull request.

Do not use a search result or one function as a complete model of a feature. Follow calls across package boundaries and inspect persisted or streamed representations when relevant.

## Code ownership map

Put behavior in the package that owns the concern:

| Area | Responsibility |
|---|---|
| `packages/core` | Agent loop, messages, tool contracts, events, confirmation, sessions, compaction |
| `packages/provider` | Provider clients, wire formats, streaming, retries, model metadata and discovery |
| `packages/provider/auth` | Login flows, credential lookup, refresh, and storage |
| `packages/agent` | CLI configuration, model selection, runtime assembly, RPC, and zotfiles |
| `packages/agent/modes` | Interactive, print, JSON, dialog, bot, and Telegram interfaces |
| `packages/agent/extensions` | Extension process lifecycle and host integration |
| `packages/agent/extproto` | Extension wire protocol |
| `packages/agent/ext` | Public extension SDK |
| `packages/agent/skills` | Skill discovery and loading |
| `packages/agent/swarm` | Background-agent state, supervision, and persistence |
| `packages/agent/tools` | Built-in tools, permission checks, and jail behavior |
| `packages/tui` | Terminal input, layout, rendering, markdown, themes, and images |
| `packages/ignore` | Ignore-pattern matching |

Provider quirks must not leak into `packages/core`. Terminal escape handling and visual layout belong in `packages/tui`. Avoid making the interactive mode a catch-all when a focused package can own the behavior.

## Correctness contracts

Some parts of zot require extra care because small mistakes corrupt state or weaken user protections.

### Agent events and sessions

- Keep streamed events and stored transcripts structurally valid.
- Preserve tool-call and tool-result pairing, including cancellation and error paths.
- Maintain compatibility with existing session files unless a migration is explicitly part of the task.
- Validate replay, resume, fork, import, and export paths when changing shared session structures.

### Providers

- Keep request construction, response parsing, streaming state, and retry classification inside `packages/provider`.
- Never assume two OpenAI-compatible services have identical edge behavior.
- When changing model metadata, verify provider ID, model ID, context size, output limit, reasoning support, routing, and pricing independently.
- Tests must use local servers or fixtures. Do not make paid or credentialed API calls.

### Credentials and private data

- Treat API keys, OAuth tokens, session text, extension payloads, and local state as sensitive.
- Never include secrets in logs, fixtures, snapshots, errors, or commits.
- Credential files must retain restrictive permissions where the platform supports them.
- Test authentication with synthetic values and isolated temporary directories.

### Tools and confinement

- Permission checks must fail closed.
- Resolve path and symlink behavior before allowing filesystem access.
- Jail mode is an accident-prevention guardrail, not a security sandbox. Documentation and errors must not claim stronger isolation.
- Preserve confirmation semantics across every mode. A convenient fallback must not silently grant access.

### Extensions and RPC

- Existing JSON-RPC and extension clients are compatibility surfaces.
- Add fields in a backward-compatible way and tolerate peers that do not send newly introduced fields.
- Keep stdout machine-readable where it carries protocol data. Diagnostics belong on stderr or in the established event channel.
- Ensure child processes are cleaned up on normal exit, cancellation, and startup failure.

### Terminal behavior

- Rendering code must account for ANSI sequences, Unicode display width, wrapping, narrow terminals, and partial streamed content.
- Avoid tests that depend on a developer's terminal capabilities.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patriceckhart/zot](https://github.com/patriceckhart/zot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
