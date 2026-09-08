---
trigger: always_on
description: Willow is a local-first **super-app**. It bundles several distinct apps behind one
---

# Willow

Willow is a local-first **super-app**. It bundles several distinct apps behind one
shell: **Code**, **Chat**, **Media**, **Agents**, **Spark**, **Design**, and an
unfinished **Figma**-like canvas. Each app lives in its own folder under
`features/` and could plausibly have been built as a standalone product.

New here? Read this file, then the `AGENTS.md` of the folder you are about to
touch. Every package has one.

## Layout

```
apps/studio/        The host shell. Routing, sidebar, settings. The only app.
features/<name>/    One sub-app each. Self-contained; owns its own UI + state.
platform/<name>/    Shared libraries. Used by many features, depends on none.
services/<name>/    Node backends. Separate npm packages, ship independently.
assets/             Images, video, cursors, animations, prompt suggestions.
tools/              Scripts, prototypes, research captures. Not shipped.
backup.cmd          Git checkpoint script: commit + pull --rebase + push.
```

**Where the user's projects get saved** is a swappable choice, and all of it lives
in one folder: `platform/storage/src/adapters/`. `local-disk.ts` (File System
Access API) and `google-drive.ts` implement the same operations, so adding the
local ↔ Drive toggle is a matter of picking an adapter, not rewriting callers. See
`platform/storage/AGENTS.md` before changing anything in there.

## Every package, and where its docs are

| Package | Alias | What it is |
| --- | --- | --- |
| [`apps/studio`](apps/studio/AGENTS.md) | — | The host shell: routing, sidebar, settings, Vite config |
| [`features/code`](features/code/AGENTS.md) | `@willow/code` | The Workbench — Sandpack sandbox, visual editing, and the Agent tool's Codex harness |
| [`features/chat`](features/chat/AGENTS.md) | `@willow/chat` | Standalone chat surface |
| [`features/media`](features/media/AGENTS.md) | `@willow/media` | AI image and video generation |
| [`features/agent-builder`](features/agent-builder/AGENTS.md) | `@willow/agent-builder` | React-Flow workflow canvas (frontend of the Agents app) |
| [`features/spark`](features/spark/AGENTS.md) | `@willow/spark` | Scheduling / background-task agent |
| [`features/design`](features/design/AGENTS.md) | `@willow/design` | Design surface; writes into the workspace's `Design/<project>/` area |
| [`features/projects`](features/projects/AGENTS.md) | `@willow/project-browser` | Project browser **UI** |
| [`features/auth`](features/auth/AGENTS.md) | `@willow/account` | Login / account **UI** |
| [`features/onboarding`](features/onboarding/AGENTS.md) | `@willow/onboarding` | First-run flow |
| [`features/gems`](features/gems/AGENTS.md) | `@willow/gems` | Gem manager. Reference implementation of the synced-folder seam |
| [`features/figma`](features/figma/README.md) | `@willow/figma` | Unfinished canvas prototype. Not routed, not typechecked |
| [`platform/storage`](platform/storage/AGENTS.md) | `@willow/storage` | Persistence, adapters, sync. **Read before touching** |
| [`platform/projects`](platform/projects/AGENTS.md) | `@willow/projects` | Project **data model** and registry |
| [`platform/ai`](platform/ai/AGENTS.md) | `@willow/ai` | Model clients, chat orchestration, computer use |
| [`platform/auth`](platform/auth/AGENTS.md) | `@willow/auth` | Firebase, `useAuth()`, `useUserData()` |
| [`platform/ui`](platform/ui/AGENTS.md) | `@willow/ui` | Shared components |
| [`platform/core`](platform/core/AGENTS.md) | `@willow/core` | Utilities, types, constants |
| [`services/agent-builder`](services/agent-builder/AGENTS.md) | `@agentbuilder` | Workflow-engine backend. Own package, own `node_modules` |
| [`services/local-companion`](services/local-companion/AGENTS.md) | — | Optional loopback daemon: real browser + shell for Spark |
| [`assets`](assets/README.md) | `@willow/assets/*` | Static files, bundled into the app |
| [`tools`](tools/README.md) | — | Scripts, prototypes, research. Not shipped |

Two alias pairs are easy to confuse — both are documented in the feature docs:
`@willow/account` (UI) vs `@willow/auth` (Firebase), and `@willow/project-browser`
(UI) vs `@willow/projects` (data).

## Where new work goes

Start here before creating a file. The rule of thumb: **one sub-app per
`features/` folder, anything shared moves down to `platform/`, anything that runs
on Node lives in `services/`, and anything not shipped lives in `tools/`.**

| You are adding | It goes in | Notes |
| --- | --- | --- |
| A whole new sub-app | `features/<name>/src/` | Needs wiring — see below |
| UI or logic for an existing sub-app | that feature's own `src/` | Group a cluster of modules in a subfolder |
| Something two features both need | `platform/*` | Pick the package from the table below |
| A shared React component | `platform/ui/src/` | `@willow/ui/<module>` |
| A util, type or constant | `platform/core/src/` | The default home for cross-cutting plumbing |
| Model clients / prompt orchestration | `platform/ai/src/` | |
| Persistence, save/load, sync | `platform/storage/src/` | Read its `AGENTS.md` first |
| A Node backend or daemon | `services/<name>/` | Its own `package.json` + `node_modules` |
| A one-off maintenance script | `tools/scripts/` | Typechecked; see caveat below |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YashjitPal/Willow](https://github.com/YashjitPal/Willow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
