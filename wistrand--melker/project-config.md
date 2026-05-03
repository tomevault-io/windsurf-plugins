---
trigger: always_on
description: Guidance for Claude Code when working with Melker.
---

# CLAUDE.md

Guidance for Claude Code when working with Melker.

## Project Overview

**Melker** - *Run text with meaning*

Website: https://melker.sh

Melker Engine is a Deno library for creating rich Terminal UI interfaces using an HTML-inspired document model. It renders component trees to ANSI terminals using a dual-buffer system.

## Quick Reference

| What                | Where                                                      |
|---------------------|------------------------------------------------------------|
| Getting started     | [getting-started.md](agent_docs/getting-started.md)        |
| .melker file format | [melker-file-format.md](agent_docs/melker-file-format.md)  |
| First app tutorial  | [tutorial.html](docs/tutorial.html)                        |
| Examples            | [examples/](examples/) (basics, components, layout, canvas) |
| **AI Agent Skill**  | [skills/creating-melker-apps/](skills/creating-melker-apps/) |
| CLI reference       | [cli-reference.md](agent_docs/cli-reference.md)            |
| Internals reference | [internals-reference.md](agent_docs/internals-reference.md) |

## Reading Order

New to Melker? Read in this order:

1. [getting-started.md](agent_docs/getting-started.md) — install, hello world, critical rules
2. [tutorial.html](docs/tutorial.html) — build your first app step by step
3. [melker-file-format.md](agent_docs/melker-file-format.md) — full `.melker` syntax reference
4. [dx-footguns.md](agent_docs/dx-footguns.md) — common mistakes to avoid
5. [script_usage.md](agent_docs/script_usage.md) — `$melker` API and script lifecycle
6. [component-reference.md](agent_docs/component-reference.md) — all 30+ components

## Documentation Index

### For App Developers

| Topic                          | Doc                                                    |
|--------------------------------|--------------------------------------------------------|
| Script context ($melker, $app) | [script_usage.md](agent_docs/script_usage.md)                        |
| State bindings (optional)      | [state-binding-architecture.md](agent_docs/state-binding-architecture.md) |
| Selection binding              | [bind-selection-architecture.md](agent_docs/bind-selection-architecture.md) |
| Selection ID system            | [selection-id-architecture.md](agent_docs/selection-id-architecture.md)     |
| Graphics modes                 | [gfx-modes.md](agent_docs/gfx-modes.md)                              |
| Debugging & logging            | [debugging.md](agent_docs/debugging.md)                              |
| Common mistakes                | [dx-footguns.md](agent_docs/dx-footguns.md)                          |
| AI assistant                   | [ai-accessibility.md](agent_docs/ai-accessibility.md)                |
| I18n (internationalization)    | [i18n-architecture.md](agent_docs/i18n-architecture.md)              |

### Component Reference

| Component                                         | Doc                                                                            |
|---------------------------------------------------|--------------------------------------------------------------------------------|
| Filterable lists (combobox, select, autocomplete) | [filterable-list-architecture.md](agent_docs/filterable-list-architecture.md)  |
| File browser                                      | [file-browser-architecture.md](agent_docs/file-browser-architecture.md)        |
| Data table                                        | [data-table.md](agent_docs/data-table.md)                                      |
| Data bars (charts)                                | [data-bars.md](agent_docs/data-bars.md)                                        |
| Data boxplot                                      | [data-boxplot-architecture.md](agent_docs/data-boxplot-architecture.md)        |
| Data heatmap                                      | [data-heatmap-architecture.md](agent_docs/data-heatmap-architecture.md)        |
| Data tree                                         | [data-tree-architecture.md](agent_docs/data-tree-architecture.md)              |
| Split pane                                        | [split-pane-architecture.md](agent_docs/split-pane-architecture.md)            |
| Spinner                                           | [spinner-architecture.md](agent_docs/spinner-architecture.md)                  |
| Toast notifications                               | [toast-architecture.md](agent_docs/toast-architecture.md)                      |
| Tooltips                                          | [tooltip-architecture.md](agent_docs/tooltip-architecture.md)                  |
| Mermaid diagrams in markdown                      | [mermaid-support.md](agent_docs/mermaid-support.md)                            |
| Tile map                                          | [tile-map-architecture.md](agent_docs/tile-map-architecture.md)                |
| SVG overlay (canvas/img/video/map)                | [svg-overlay-architecture.md](agent_docs/svg-overlay-architecture.md)          |

### For Contributors (Internals)

| Topic               | Doc                                                                  |
|---------------------|----------------------------------------------------------------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wistrand/melker](https://github.com/wistrand/melker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
