---
trigger: always_on
description: This repository contains small, independently installable extensions for the [Pi coding agent](https://github.com/earendil-works/pi). The project is public and portable: changes must work without access to a maintainer's machine, private services, or local daemon.
---

# pi-extensions agent and contributor guide

This repository contains small, independently installable extensions for the [Pi coding agent](https://github.com/earendil-works/pi). The project is public and portable: changes must work without access to a maintainer's machine, private services, or local daemon.

## Repository map

```text
pi-extensions/
├── packages/
│   ├── pi-extensions-i18n/      # Shared locale and catalog runtime
│   ├── pi-extensions-tool-display/ # Tool-display host and shared rendering protocol
│   ├── pi-model-request/ # Extension-side model requests (auth + provider session headers)
│   ├── pi-distill/              # Tool-output distillation
│   ├── pi-tool-supervisor/      # Post-edit file review
│   ├── pi-terminal-mux/         # Terminal multiplexer abstraction (muxy/cmux/tmux/zellij/wezterm/herdr/otty/orca + headless fallback)
│   ├── pi-metrics/               # Session metrics (live elapsed spinner, per-turn and total run summaries)
│   ├── pi-models-discovery/     # Dynamic model discovery for providers marked with discoverModels
│   ├── pi-session-tools/        # Bash pipe output cache and session_log/session_squash conversation squashing
│   ├── pi-session-resources/    # Clickable tabbed # picker for session files, browser URLs, and PR/MR links
├── scripts/                     # Repository checks and workspace helpers
├── .github/workflows/           # CI and release automation
├── README.md                    # English project documentation
├── README.zh-CN.md              # Chinese project documentation
├── AGENTS.md                    # This guide
└── package.json                 # Private npm workspace root
```

Each package owns its entrypoint, tests, configuration example, localization resources, and package README. The public package source of truth is this repository; consumers should install the published npm packages instead of copying package source into another project.

## Package boundaries

- `pi-safety-guards` is independently installable; see `packages/pi-safety-guards/README.md` for its configuration, behavior, and tests.
- `pi-nested-skills` is independently installable; see `packages/pi-nested-skills/README.md` for its configuration, behavior, and tests.
- `pi-notifications` is independently installable; see `packages/pi-notifications/README.md` for its configuration, behavior, and tests.
- `pi-naming` owns automatic Pi session titles and manual terminal naming; it uses pi-ai and terminal-mux, not session-tools. Automatic and manual naming share configurable session/workspace/tab targets.

- `pi-distill` discovers active tools with object parameter schemas and observes their results through Pi's native `tool_call` and `tool_result` events. It does not register duplicate tools.
- `pi-tool-supervisor` reviews the actual before/after diff of `edit` and `write` against configured rule files. It reports findings but is not an operating-system sandbox or an edit rollback mechanism.
- `pi-extensions-tool-display` owns the actual Pi tool-display host, built-in tool renderer overrides, and the shared result-rendering middleware protocol. Feature packages register domain-specific panels through it.
- `pi-extensions-i18n` owns locale selection, catalog validation, interpolation, and the `/pi-language` command. Feature packages use it instead of implementing separate locale runtimes.
- `pi-model-request` owns how an extension issues its own model request: resolve auth from the model registry, add the provider session headers (`x-opencode-session`, `x-opencode-client`) that Pi's core adds to its own requests, apply a resolved `baseUrl`, and call the completion. Any package that calls `completeSimple`/`complete` itself must go through it instead of re-deriving those rules.
- `pi-terminal-mux` owns terminal multiplexer detection and pane/surface operations. Extensions that need terminal interaction depend on it instead of re-implementing backend detection.
- `pi-metrics` owns session metrics: the live elapsed spinner and per-turn/total summaries listen to Pi's native `input`, `agent_start`, `turn_start`, `turn_end`, `agent_end`, and `agent_settled` events without registering tools.
- `pi-models-discovery` owns dynamic model discovery: it reads `discoverModels` providers from models.json, fetches `{baseUrl}/models`, persists a startup cache, and exposes `/model-discovery` plus `/model-discovery-refresh` commands.
- `pi-session-tools` owns the bash pipe output cache (`tool_call` rewrites `grep`/`tail`/`head` pipelines with `tee`) and `session_log`/`session_squash` for non-destructive conversation squashing; the main agent writes the handoff summary directly into the `session_squash` call.
- `pi-session-resources` observes successful tool results, rebuilds resources from the active session branch, and exposes file, browser, and PR/MR targets through a clickable, tabbed `#` picker above the editor without adding model-context messages.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maplezzk/pi-extensions](https://github.com/maplezzk/pi-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
