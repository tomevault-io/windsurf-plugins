---
trigger: always_on
description: A monorepo for Supernote plugin SDK research and plugin development. Contains:
---

# Supernote Plugin Research & Development

## What this repo is

A monorepo for Supernote plugin SDK research and plugin development. Contains:
- **SDK source** (`src/`, `lib/`, `android/`) -- extracted sn-plugin-lib internals for reference
- **Official docs** (`official-docs-extracted.md`) -- full extraction of Ratta's plugin documentation
- **Design docs** (`docs/`) -- architecture analysis and plugin design documents
- **Plugins** (`plugins/`) -- each plugin is a standalone React Native project

## Issue tracking: Jira (SNDEV)

**Features, bugs, and status live in Jira, not in markdown.** As of 2026-07-25 the active work from both plugin trackers was imported there.

- **Project:** `SNDEV` -- "SuperNote Development" at https://alexpnw.atlassian.net
- **Access:** the `atlassian` MCP server (user scope, OAuth). If its tools aren't available, the server needs authenticating via `/mcp`; it is configured in `~/.claude.json` at user scope so it applies in every project.
- **Epics:** `SNDEV-6` = SuperTask, `SNDEV-7` = SuperHub. Every issue is parented to one of them.

### Conventions

| Old tracker | Jira |
|---|---|
| `F-001` feature | issue type **Feature**, label `F-001` |
| `B-001` bug | issue type **Bug**, label `B-001` |
| `T-001` task | issue type **Task**, label `T-001` |
| plugin | label `SuperTask` or `SuperHub` |

**The original IDs are preserved as labels.** Design docs, PROGRESS files, and issue descriptions still cross-reference each other by `F-023` / `B-028`, so those references resolve by label search rather than by Jira key.

### Board statuses

`To Do` -> `In Progress` -> `Testing` -> `Done`

**`Testing` means implemented but NOT yet confirmed on-device.** This is the column that matters here: the plugin cannot be verified without a build-copy-install-test cycle, so work sits in Testing until a real device confirms it. Never move something to Done off a code reading. See "Don't mark bugs fixed before testing" in the development practices below.

### Finding things

Ask via the MCP server in plain language, or use JQL directly:

```
project = SNDEV AND status = Testing                  # the on-device test queue
project = SNDEV AND labels = SuperHub                 # everything for one plugin
project = SNDEV AND labels = "B-028"                  # look up an item by its old tracker ID
project = SNDEV AND issuetype = Bug AND status != Done
```

### What still belongs in markdown

Jira owns **what and why**: scope, priority, status, user feedback. The repo owns **how**:

- **`PROGRESS.md`** -- session handoff state (what happened, what's next, current build)
- **`docs/design-*.md`** -- deep dives on specific features or subsystems
- **`docs/changelog.md`** -- archive of completed/resolved items
- **`docs/tracker.md`** -- **FROZEN as of the 2026-07-25 import.** Kept for historical reference and for working offline. It is NOT maintained; do not update it and do not read status from it. Jira is authoritative for anything to do with state.

When you finish work, update the Jira issue. When you finish a session, update `PROGRESS.md`. When a design decision gets made, put it in the design doc and link the Jira key.

### Per-plugin documentation structure
Each plugin under `plugins/<Name>/` carries its own `PROGRESS.md`, `docs/changelog.md`, `docs/design-*.md`, and the frozen `docs/tracker.md`. Design docs cross-reference each other, and now Jira issues, via their headers.

## Plugin development practices

### Creating a new plugin
- Always scaffold from `template/`, never copy another plugin. Each plugin is its own standalone RN project with its own dependencies.
- Rename all `HelloWorld`/`helloworld` references to the new plugin name in: app.json, package.json, android package dirs + kotlin files, ios dirs + swift/xcodeproj, build.gradle namespace.
- Each plugin gets its own directory under `plugins/` with its own `PROGRESS.md` for session continuity.

### Plugin architecture
- **React Native 0.79.2 + React 19.0.0** -- locked versions, do not upgrade
- **sn-plugin-lib** -- Supernote SDK bridge, the only way to talk to the device. Versions have diverged: SuperTask is on `^0.1.43`, while `template/` and SuperHub are still on `^0.1.19`. Check the plugin's own `package.json` before assuming an API exists.
- **Build output** -- `buildPlugin.sh` produces a `.snplg` file (zip of Hermes bytecode + assets + PluginConfig.json)
- **No native modules needed for pure JS plugins** -- build skips Gradle entirely, runs in under a minute
- **Install on device** -- copy `.snplg` to MyStyle/, then Settings > Apps > Plugins > Install

### Problem-solving protocol: check the SDK source first
When stuck on how to accomplish something on-device (inserting elements, marking strokes, navigating, etc.), **read the SDK TypeScript source** in `src/` before guessing or trying undocumented approaches. The SDK source has JSDoc comments with parameter docs, enum values, style constants, and validation logic that aren't in the official docs. Examples of wins from this:
- `PluginNoteAPI.setLassoTitle({style: 1})` -- discovered by reading `src/sdk/PluginNoteAPI.ts`, not documented elsewhere
- `setLassoStrokeLink` params and link style/type enums -- all in the source

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apclark31/supernote-plugin-research](https://github.com/apclark31/supernote-plugin-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
