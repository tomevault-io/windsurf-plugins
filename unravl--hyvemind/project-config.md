---
trigger: always_on
description: > **Read `PRODUCT.md` first.** It contains the product overview (what Hyvemind is, the three systems, the bee colony agents, key differentiators, current alpha state, brand, business model, roadmap, glossary). This file is the **technical/operational reference** — file paths, IPC surface, debug commands, investigation guides — and deliberately does **not** repeat product context.
---

@PRODUCT.md

# Hyvemind — Technical & Operational Reference

> **Read `PRODUCT.md` first.** It contains the product overview (what Hyvemind is, the three systems, the bee colony agents, key differentiators, current alpha state, brand, business model, roadmap, glossary). This file is the **technical/operational reference** — file paths, IPC surface, debug commands, investigation guides — and deliberately does **not** repeat product context.
>
> **Terminology**: Hyvemind has no "Chat" surface. Wherever this file says `chat.rs`, `chat-event`, `chat-sessions/`, etc., these are internal/historical names for the **Tasks-view conversation**. See `PRODUCT.md §3` for details.

## How to use this document

This file is the technical reference for AI coding agents and humans working on Hyvemind. The major sections you can jump to:

- **[Documentation Maintenance](#documentation-maintenance)** — the rule + pointer to the full trigger tables (`docs/doc-maintenance.md`). Consult it after any change touching IPC / events / env-vars / new files / exported types.
- **[Quick Reference](#quick-reference)** — common dev commands.
- **[Content Security Policy](#content-security-policy-csp)** — the renderer's CSP and why each directive is what it is.
- **[OpenCode runtime](#opencode-runtime)** — how the external OpenCode agent runtime is resolved and managed.
- **[Crash recovery](#crash-recovery)** — what's reconciled on the next launch and what's lost.
- **[Project Layout](#project-layout)** — Rust backend file tree.
- **[Frontend Architecture](#frontend-architecture)** — React 18 + Vite + Tailwind shell, screens, providers, IPC layer, event listeners.
- **[Key Types](#key-types)** — the types most code paths route through.
- **[Diagnosing Hyvemind entities](#diagnosing-hyvemind-entities)** — START HERE for any investigation: the one-call `hyvemind-diagnose` skill.
- **Investigation guides** — manual fallback recipes for [Tasks](#investigating-a-task), [Sessions](#investigating-a-session-id), [Hivemind Reviews](#investigating-a-hivemind-review), and [Swarms](#investigating-a-swarm--progress_logjsonl).
- **[Tauri Commands](#tauri-commands-ipc)** — IPC surface.
- **[Tauri Events](#tauri-events-backend--frontend)** — streaming event channels.
- **[Environment Variables](#environment-variables)** — credentials, debug flags, tunables.
- **[Debug Mode](#debug-mode-checking-logs)** — how to read the structured logs Hyvemind writes when `HYVEMIND_DEBUG=1`.

### Sibling documentation

Don't duplicate any of these in CLAUDE.md — link to them:

| Doc | What it covers |
|-----|----------------|
| `PRODUCT.md` | Product context: vision, three systems, bee-colony agents, brand, roadmap, glossary |
| `CONTRIBUTING.md` | Dev setup, PR checklist, how to add a provider |
| `CHANGELOG.md` | Release notes; `[Unreleased]` accumulates work since the last tag |
| `SECURITY.md` | Threat model, vulnerability reporting process |
| `CODE_OF_CONDUCT.md` | Contributor Covenant v2.1 |
| `app/src/A11Y.md` | Frontend accessibility conventions (aria-live, screen-reader rules) |
| `docs/README.md` | Index of all topical / deep-dive docs + the per-subsystem README map (read this to find a doc; this is the index the now-removed `AGENTS.md` used to provide) |
| `docs/architecture.md` | System component map + sequence-flow diagrams (Mermaid) + crash-recovery + storage layout |
| `docs/ipc-reference.md` | Per-command reference for every Tauri IPC handler (signature / returns / errors / delegation) |
| `docs/frontend-architecture.md` | React deep-dive: provider tree, `taskRuntime.tsx` sub-contexts, IPC wrapper, singleton event stores, recipes |
| `docs/bee-agents.md` | Per-agent deep-dive (Queen / Scout / Worker / Guard / Nurse + scout-review + stability-test pair) |
| `docs/providers.md` | LLM provider abstraction overview (the 4 production impls + mock, dispatch, cost, circuit-breaker integration) |
| `docs/extension-authoring.md` | How to author Rust provider extensions and frontend topbar widgets (+ the OpenCode `submit_*` echo-tool generation) |
| `docs/developer-cookbook.md` | Task-oriented recipes (add a command / tunable / screen / migration, point at a custom opencode binary, enable debug, etc.) |
| `docs/doc-maintenance.md` | Doc-authoring guide: which change requires which CLAUDE.md / PRODUCT.md / README section edit + the drift-audit commands |
| `docs/hivemind-custom-prompts.md` | Ready-to-paste Hivemind reviewer prompts |
| `app/src-tauri/src/core/README.md` | Swarm execution engine deep-dive |
| `app/src-tauri/src/hivemind/README.md` | Multi-model review engine deep-dive |
| `app/src-tauri/src/state/README.md` | Persistence, AppState, secret store, atomic writes |
| `app/src-tauri/src/extensions/README.md` | Provider usage/balance extension framework |
| `app/src-tauri/src/nurse/README.md` | Nurse push-mode engine — bus, detectors, three-tier dispatch pipeline, always-on observability |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Unravl/Hyvemind](https://github.com/Unravl/Hyvemind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
