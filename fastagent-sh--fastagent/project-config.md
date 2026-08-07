---
trigger: always_on
description: fastagent is "Vibe first. Then FastAgent" for agent directories: it turns a file-defined agent (`persona.md` identity, `skills/`, tools, and existing `AGENTS.md` project context) into a live service inside an app, on GitHub, in Telegram, or behind a custom channel without a new authoring DSL.
---

# fastagent — Agent Guide

## What this is

fastagent is "Vibe first. Then FastAgent" for agent directories: it turns a file-defined agent (`persona.md` identity, `skills/`, tools, and existing `AGENTS.md` project context) into a live service inside an app, on GitHub, in Telegram, or behind a custom channel without a new authoring DSL.

The stable design center is the engine-neutral Agent Handler contract (`docs/SPEC.md`); pi (`@earendil-works/pi-*`) is the reference implementation.

## Source of truth

| Document | Purpose |
|---|---|
| `docs/SPEC.md` | The locked v0.1 Agent Handler contract. Do not change its semantics without an explicit decision. |
| `docs/design/core.md` | The pi reference implementation and current architecture. |
| `docs/design/participant-model.md` | When a chat channel speaks, where it answers, what it remembers. Authority for Feishu/Lark + Slack routing. |
| `docs/overview.md`, `docs/README.md` | Product overview and documentation index. |
| `CONTRIBUTING.md` | The full GitHub workflow (branch model, PR loop, merge strategy, review policy). |

Code truth is `src/`.

## Repo map

```
src/
├── agent.ts                 # the Agent Handler contract (pure types, no engine import)
├── collect.ts               # caller-side stream helpers: collect (buffered consumption) + abortFirstIterator (shared cancellation protocol)
├── core.ts, pi.ts           # lightweight neutral subpath + pi reference-implementation subpath
├── index.ts                 # supported all-in-one public surface (re-exports core + pi)
├── cli.ts                   # the THIN entry (import-free; lazy-loads cli/program.ts)
├── cli/                     # the CLI, built on clig.dev: kernel.ts (CommandSpec-as-data + the commander adapter — commander appears ONLY here; help/suggestions/exit-code policy: 0 ok, 1 runtime, 2 usage), program.ts (the spec registry — the CLI surface's single source of truth; lazy per-command imports), presenters (invoke-stream.ts `invoke` stream → exit code; models-view.ts/auth-view.ts `models`/auth-report output; add-feishu.ts `add feishu|lark` app onboarding), shared.ts/serve.ts (cross-command helpers incl. mountSessionControl — per-boot token + control.json discovery file; process side effects live in the command modules), fail.ts, commands/ (one module per command)
├── telegram.ts, github.ts   # subpath-export shims (@fastagent-sh/fastagent/telegram etc. — the supported surface)
├── bind.ts                  # THE reading of a bind address, as the six DIFFERENT questions it is:
│                           # bindable (isBindAddress) / an address not a name (bindAddress, applied
│                           # where a value enters) / reach (classifyBind) / dialable by the NAME
│                           # localhost (answersLocalhost — NOT the same as reach: 127.0.0.2 is
│                           # loopback and --tunnel still cannot reach it) / how a message names it
│                           # (bindLabel) / what a client dials (clientHost). The flag, http.host
│                           # validation, serveNode, the ready lines, control.json and the deploy
│                           # pre-flight all read one through this — conflating any two of those
│                           # questions produces a silent failure, which is why they are separate.
├── log.ts                   # leveled logging singleton (dev=debug, start=info)
├── session.ts               # engine-neutral session-control contract (SessionControl: state/entries/events + dispatch, error codes)
├── session-remote.ts        # remote clients over /control/*: connectSessionControl (control plane) + connectAgent (data plane)
├── observe.ts               # turn-trace logging around an Agent
├── tunnel.ts                # `--tunnel`: cloudflared + per-channel webhook dispatch
├── dev-supervisor.ts        # `dev` supervisor: restart on code-input edits (definition is live-read per invoke)
├── proxy.ts                 # HTTPS_PROXY wiring
├── env.ts                   # `.env` → process.env loading (missing file is normal; anything else surfaces)
├── runtime.ts               # agent runtime/package-manager detection (node vs bun) + readPackageJson
├── loader.ts                # neutral ESM module discovery/loading for tools/ channels/ schedules/ config
├── paths.ts                # PLACEMENT: resolvePlacement — ONE marker (`fastagent.config.*`, at any
│                           # NAME) and one rule: the workspace is the dir you point at, the agent is the
│                           # single config holder at it or one level inside + the machinery paths that follow
│                           # (.secrets/.state + env overrides), the containment guard, and the neutral
│                           # path helpers the CLI/deploy share (displayPath, exists). Engine-neutral,
│                           # so the scaffold/deploy/watcher/env consume it without touching engines/pi.
├── version.ts              # package version (deploy pins it into the image)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fastagent-sh/fastagent](https://github.com/fastagent-sh/fastagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
