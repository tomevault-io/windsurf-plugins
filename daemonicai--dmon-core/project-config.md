---
trigger: always_on
description: dmon (pronounced like "demon") is a .NET-native coding agent inspired by [Pi](https://github.com/earendil-works/pi). It is written in **C# on .NET 10**. The agent core runs as a separate process over JSONL/stdio. Two host surfaces are planned: a console/TUI host and an Avalonia desktop host.
---

# dmon — Project Instructions

## What this project is

dmon (pronounced like "demon") is a .NET-native coding agent inspired by [Pi](https://github.com/earendil-works/pi). It is written in **C# on .NET 10**. The agent core runs as a separate process over JSONL/stdio. Two host surfaces are planned: a console/TUI host and an Avalonia desktop host.

See [`coding-agent-brief.md`](./coding-agent-brief.md) for the full vision and architectural rationale.

---

## Tech stack

- **Language:** C# 13 / .NET 10
- **LLM abstraction:** `Microsoft.Extensions.AI` (`IChatClient`) — see ADR-001
- **Extension model:** Two tiers — `.csx` scripts (hot-loaded) and NuGet packages loaded into the Default `AssemblyLoadContext` — see ADR-002, ADR-008
- **RPC protocol:** JSONL over stdio, Pi-compatible shape — see ADR-003
- **Session storage:** Relocatable directory with `messages.jsonl` + `meta.json` + `attachments/` — see ADR-004
- **Provider auth:** API keys via env vars or config file — see ADR-005
- **Permission model:** Tiered prompts (read/write/bash/network), conservative by default — see ADR-006

---

## Architecture Decision Records (ADRs)

ADRs in [`docs/adrs/`](./docs/adrs/) are **binding**. Accepted ADRs must not be contradicted by code or proposals. If new information warrants reconsidering a decision, write a new ADR with status **Supersedes: ADR-NNN** and get it accepted before implementing the change.

Key accepted decisions:

| ADR | Decision |
|-----|----------|
| ADR-001 | Use `IChatClient` (M.E.AI) for LLM abstraction. No MAF dependency. |
| ADR-002 | Extensions expose `AIFunction` via `IDmonExtension`. No wrapper interface. (Loading mechanism superseded by ADR-008.) |
| ADR-003 | JSONL over stdio, Pi-compatible protocol shape. Not strict JSON-RPC 2.0. |
| ADR-004 | Session = relocatable directory. `messages.jsonl` append-only. Large outputs in `attachments/`. |
| ADR-005 | API keys only (env or config). No OAuth for V1. |
| ADR-006 | Conservative permission model. Read within CWD is implicit; all writes require a prompt. |
| ADR-007 | Provider-extension lifecycle: `IsApplicable()` at load, `EnsureRunningAsync()` gated, per-runner `IProviderFactory`. |
| ADR-008 | Extensions load into the **Default** `AssemblyLoadContext` (no collectible per-load contexts); reclaim via process restart. Supersedes ADR-002's loading mechanism. |
| ADR-009 | *(Superseded by ADR-019.)* Config-driven extension loading (`config.yaml` active-extension list, reflection-loaded at startup) — replaced by composition-root hosting. |
| ADR-010 | A scoped single-turn in-process `IChatClient` in a tool extension is in scope; multi-agent orchestration (multiple `dmon-core` processes over stdio/RPC) remains deferred. |
| ADR-011 | Distribution model: granular contract packages on nuget.org; `dmon` (dotnet tool) acquires `dmoncore` at runtime into the global NuGet cache (no bundling); 3-part protocol-keyed version scheme (`Major.Minor` = wire protocol). |
| ADR-012 | Remote access transport: a WebSocket gateway with connection-decoupled, resumable sessions; **Tailscale** is the auth/encryption boundary (single-tenant, home-server); optional shared key for defense-in-depth. |
| ADR-013 | *(Superseded by ADR-022.)* Agent profiles: a named bundle (persona + per-session `assets/` toggle + permission mode) — dissolved into builder verbs and `.cs` selection. |
| ADR-014 | Gateway event replay uses an in-memory per-session `seq` buffer in the live handler, **not** `messages.jsonl` (which holds only conversational turns, written only by the core). Amends ADR-012 Decision 4. |
| ADR-015 | Command results are dedicated typed events correlated by command `id` (`ResultEvent` base; `CommandErrorEvent` for failures); the generic `{type:"response", data}` envelope is retired. Makes the wire contract describable for client generation. Amends ADR-003's response framing. |
| ADR-016 | Conversation persistence: session-storage owns a lossless, dmon-owned **parts** record (memory tiers derive their index from it); no third-party types in the API definition; lenient mapping preserves unmodelled content as render-only opaque `UnknownPart`. Amends ADR-004/ADR-014; supplies ADR-015's deferred `getMessages` DTO. |
| ADR-018 | Gateway auth: a per-device, revocable key set (file-backed `devices.json`, hot-reloaded, fail-closed to last-good) replaces the single shared key; a match tags the connection with its `keyId` and revocation fences live connections. Amends ADR-012 Decisions 6/10/12. |
| ADR-019 | Composition-root hosting: `dmoncore` is a **library**; a .NET 10 file-based program `Dmon.cs` (built, then run with `--no-build` so the build phase stays off the stdio wire) is the core entry point and declares its extensions as compile-time `#:package` deps. Supersedes ADR-009 in full, ADR-011 D2–4, ADR-008's dynamic-load mechanism, and ADR-002's `.csx`/`promote` tier. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daemonicai/dmon-core](https://github.com/daemonicai/dmon-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
