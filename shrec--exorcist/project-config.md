---
trigger: always_on
description: Exorcist is a fast, lightweight, cross-platform Qt 6–based open-source IDE targeting Visual Studio–level capabilities with minimal footprint. Every contribution must preserve modularity, portability, and architectural clarity.
---

# Exorcist IDE — Project Guidelines

Exorcist is a fast, lightweight, cross-platform Qt 6–based open-source IDE targeting Visual Studio–level capabilities with minimal footprint. Every contribution must preserve modularity, portability, and architectural clarity.

## UI/UX Reference (MANDATORY)

All UI work must follow the Visual Studio 2022 dark theme reference. Before implementing any visual component:

1. **Read** `docs/reference/ui-ux-reference.md` — full layout specification, color palette, and UX principles.
2. **View** `docs/reference/vs-ui-reference.png` — the authoritative visual reference screenshot.
3. **Match** the dark theme aesthetic (#1e1e1e backgrounds), panel arrangement (left=explorers, right=tools, bottom=terminals/output), and information density from the reference.
4. **Never** use light/white backgrounds, minimalist sparse layouts, or panel arrangements that deviate from the reference without explicit approval.

## Design Philosophy

The editor must always stay **lightweight**. Users who don't need a module must never pay its memory or startup cost. The modular plugin system gives users maximum freedom to compose exactly the environment they need — enable what they use, disable everything else. No wasted resources, no unnecessary complexity.

Project motto: build a developer-friendly space that stays practical, uses minimal resources, and directly neutralizes real developer pain points.

This motto is enforceable, not decorative:

- If a feature does not reduce real developer friction, question whether it should exist.
- If a feature increases idle memory, startup cost, or background activity for users who did not ask for it, move it behind plugin activation, lazy loading, or shared infrastructure.
- If a workflow pain point appears in multiple plugins, fix it once in a shared base/service/component instead of duplicating the workaround.
- If a design is harder for plugin authors than it needs to be, keep simplifying the SDK and base layers until the standard path is obvious.

### Write-Once Reuse-Everywhere Principle (STRICT)

Shared components (Editor, Output Panel, Terminal, Project Tree, Debugger UI, Test Runner, UART Terminal, Search Panel, Problems Panel) are **finished, self-contained systems with standardized interfaces**. They are written once and reused by every plugin that needs them, parameterized per context.

- **Components DLL**: core UI building blocks — editors, output panels, project trees, search, problems
- **Instruments DLL**: developer tools — terminals, UART terminals, debugger UIs, test runners

A language plugin is a **thin glue layer**: it imports shared components, initializes them with language-specific parameters (LSP config, syntax grammar, file filters), and wires its own narrow language logic to those component interfaces. 100 new languages = 0 component rewrites.

**Plugin-specific UI is the exception.** If a language needs a unique dock panel, it writes that panel inside the plugin and plugs it into the host dock interface. Everything else uses shared components.

### Container-Only Shell Principle (STRICT)

MainWindow is a container. It does not know or care what plugins do internally. It provides:
- 3 dock bar regions (left, right, bottom)
- Menu bar, toolbar, and status bar rails
- Workspace/session lifecycle
- ServiceRegistry and bridge access

Plugins access these surfaces through `IHostServices` — a **direct** reference (not through PluginManager). PluginManager only loads/unloads; once loaded, the plugin talks to IHostServices directly.

### ExoBridge = Global Daemon (not a plugin)

ExoBridge manages machine-global shared resources (git tokens, OAuth, MCP servers, git watchers) across all IDE instances. If a resource depends on which project is open or which window is active, it stays per-IDE-instance, not in ExoBridge.

See [../docs/core-philosophy.md](../docs/core-philosophy.md) for the full Core IDE vs Core Plugins philosophy, activation model, and performance principles.
See [../docs/development-principles.md](../docs/development-principles.md) for the 10 development principles that guide how the project evolves.

## Architecture

- **Shell/container-first design**: The IDE shell is a container and bridge host, not a feature owner. `src/` should expose regions, lifecycle orchestration, contracts, and bridge access, not concrete subsystem workflows.
- **Interface-first design**: Core host surfaces (`IFileSystem`, `IProcess`, `ITerminal`, `INetwork`, `IHostServices`, dock/menu/toolbar/status interfaces) are abstract contracts. Never depend on concrete implementations across boundaries.
- **Plugin-first extension model**: Concrete feature systems belong in plugins. If a behavior has its own workflow, runtime state, commands, menus, toolbars, docks, status UI, or provider lifecycle, the default owner is a plugin.
- **Service registry for loose coupling**: Services are registered/resolved by string key. Plugins must not reference each other directly — communicate through services.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shrec) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
