---
trigger: always_on
description: Monorepo of Pi-native extension packages that register tools and skills directly
---

# pi-extensions

Monorepo of Pi-native extension packages that register tools and skills directly
into the Pi coding agent, each in its own npm package under `@bacnh85/`.

## Packages

| Package | Version | Description |
|---------|---------|-------------|
| **pi-router** | 1.1.3 | Connect to any OpenAI-compatible AI router (9router, omniroute, …) via its /v1 API; API key via built-in /login, URL in settings.json (`/router-config` panel), models auto-cached in models-store.json. |
| **pi-chatgpt-web** | 0.2.0 | ChatGPT web-tier providers via self-hosted OpenAI-compatible bridges — chat-only `chatgpt-web` (chatgpt2api) + agentic `codex-web` (codex-proxy, tool-capable). No Plus subscription. |
| **pi-commandcode** | 0.2.0 | Connect to Command Code's OpenAI-compatible Provider API; API key via built-in `/login`, base URL in settings.json (`/commandcode-config` panel), models auto-cached. |
| **pi-checkpoint** | 0.1.0 | Git-backed undo/redo — snapshots file state per turn into a dedicated ref namespace so `/undo` rolls back a message AND its file changes. |
| **pi-notify** | 0.1.1 | Desktop notifications and sounds — fires on task completion, errors, and questions; cross-platform (macOS/Linux/Windows + terminal OSC). |
| **pi-references** | 0.1.1 | External context roots — alias sibling dirs or git repos as `@docs`/`@sdk`; auto-clones repos and injects descriptions into agent context. |
| **pi-budget** | 0.1.2 | Spend cap enforcement — `--budget <usd>` aborts the agent at the cap; companion to pi-sub (render vs enforce). |
| **pi-init** | 0.1.0 | Guided AGENTS.md generation — `/init` scans the repo and generates/updates AGENTS.md with build/test/lint commands, architecture, and conventions. |
| **pi-permission** | 0.2.0 | Granular permission system — config-driven allow/ask/deny rules per tool with wildcard patterns, external-directory boundary, and a doom-loop guard. |
| **pi-agy** | 0.3.2 | Google Antigravity CLI bridge for delegated implementation, scaffolding, refactors, and test generation. |
| **pi-fff** | 0.7.9 | FFF-powered fuzzy file and content search for Pi. |
| **pi-kicad** | 0.1.3 | KiCad CAD-design extension — drive schematic capture and PCB layout via the Konnect binary over a local HTTP daemon (no MCP SDK). |
| **pi-model-tools** | 0.6.1 | Unified tool-wrapping, argument repair, reasoning management, DeepSeek V4 guidance + Super Power Mode, defensive leak-cleaning, edit mismatch repair, and a Codex-style apply_patch diff tool. |
| **pi-munin** | 0.5.2 | Munin long-term memory as eight native Pi tools for search, retrieval, storage, listing, deletion, capabilities, and confirmed cross-project sharing. |
| **pi-evolve** | 0.3.2 | Trajectory-based self-learning loop — captures tool-call trajectories, reflects to extract learnings, persists to Munin or local JSONL, injects recent learnings into future sessions. |
| **pi-a2a** | 0.7.0 | A2A Protocol v1.0 bidirectional — Pi distributes tasks to remote agents (Hermes, ADK, LangChain, any A2A peer), exposes itself as an A2A-callable agent, self-declares for local session discovery (file registry + enriched Agent Card + mDNS), registers with **multiple a2a-switchboard gateways** (`discovery.gateways`), shows inbound task activity in the host TUI, and has an interactive config panel. |
| **pi-config-panel** | 0.1.0 | Shared interactive config-panel kernel (library) — arrow-key toggle/edit overlay panels (`PanelRow`/`PanelGroup` + `ConfigPanelModel` TUI shell) via `ctx.ui.custom`; powers `/a2a-config`, `/commandcode-config`, `/router-config`. |
| **pi-hub** | 0.1.3 | Interactive installer CLI — `npx @bacnh85/pi-hub` browses the @bacnh85 catalog, searches npm `keywords:pi-package`, multi-selects, and shells out to `pi install`. |
| **pi-notebooklm** | 0.1.8 | Google NotebookLM — notebooks, sources, chat, research, and Studio artifacts via CLI bridge. |
| **pi-obsidian** | 0.8.14 | Obsidian vault integration for Pi. |
| **pi-advisor** | 0.1.4 | OMP-style automatic advisor — a second model reviews each settled turn and injects severity-routed notes (nit card / concern steer, immune-turn cooldown, emission guard); plus the on-demand advisor consult moved from pi-plan. |
| **pi-plan** | 0.11.1 | Plan mode with read-only gating and plan → implement → verify → review workflow; fallback model chain on overload. |
| **pi-ponytail** | 0.1.11 | Lazy senior dev mode — YAGNI/stdlib-first coding discipline. Fork of [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail). |
| **pi-review** | 0.2.8 | Isolated read-only code review with corrected same-session fallback. |
| **pi-rtk** | 0.1.12 | Bash command token rewriting through RTK. |
| **pi-serena** | 0.9.13 | Serena semantic code tools (find/replace/rename symbols, LSP diagnostics) through a persistent TypeScript worker with Python bridge. |
| **pi-sub** | 0.1.32 | Subscription usage footer for OpenAI Codex, OpenCode Go, and Z.ai. |
| **pi-themes** | 0.1.1 | Ayu-based theme collection (dark, mirage, light) for the Pi TUI — pure-themes package (no extension code). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bacnh85/pi-extensions](https://github.com/bacnh85/pi-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
