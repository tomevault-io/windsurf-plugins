---
trigger: always_on
description: This file contains instructions and context for any AI agents (like yourself) working on the `the-pair` repository.
---

# AI Agent Guidelines for "The Pair"

This file contains instructions and context for any AI agents (like yourself) working on the `the-pair` repository.

## Project Identity

"The Pair" is a Desktop application (v1.2.3) built with Tauri 2.x, React 19, and TypeScript. It orchestrates two local AI agents — a **Mentor** (planner/reviewer, read-only) and an **Executor** (code writer/command runner) — that cross-check each other's work. The app manages their lifecycle, monitors resources, tracks git changes, and provides a polished UI for humans to observe and intervene.

## Tech Stack

- **Framework:** Tauri 2.0
- **Backend:** Rust
- **Frontend:** React 19, TypeScript
- **Styling:** Tailwind CSS v4, `clsx`, `tailwind-merge`
- **Icons:** `lucide-react`
- **State Management:** `zustand` (`usePairStore`, `useUpdateStore`, `useThemeStore`)
- **Animations:** Framer Motion

## Architecture Rules

1. **Rust Backend vs. React Frontend:**
   - Never import Node.js built-ins directly into the Renderer (`src/renderer/src/`).
   - Use Tauri commands in `src-tauri/src/` to expose strict APIs to the frontend.
   - Frontend calls backend via `invoke()` from `@tauri-apps/api/core`.
2. **Styling:**
   - Always use Tailwind CSS classes. Custom dark/light mode palette is defined in `src/renderer/src/assets/main.css`.
   - Use the `cn()` utility from `src/renderer/src/lib/utils.ts` for conditional class merging.
3. **Agent Interactions:**
   - The application spawns processes. Always implement an iteration limit (`maxIterations`) before pausing for human intervention.
   - Use the handoff guard (`src/renderer/src/lib/handoffGuard.ts`) to prevent race conditions when pairs finish.
4. **Error Handling:**
   - Handle permissions and locked files defensively in the Rust backend.
   - Propagate errors cleanly to the frontend via Tauri commands to display in the `ErrorDetailPanel`.

## Rust Backend Modules (`src-tauri/src/`)

| Module              | Responsibility                                                                                        |
| ------------------- | ----------------------------------------------------------------------------------------------------- |
| `pair_manager`      | Pair lifecycle: create, list, delete, pause, resume, assign task, update models                       |
| `message_broker`    | State machine for agent turn coordination and event routing                                           |
| `process_spawner`   | Spawns opencode/claude/codex/gemini CLI processes, parses JSON event streams                          |
| `provider_adapter`  | Abstracts provider differences (input/output transport, session/permission/cwd strategies)            |
| `provider_registry` | Detects installed providers (opencode, codex, claude, gemini), reads their configs and model catalogs |
| `model_catalog`     | Static model metadata (display names, billing kind, recommended roles)                                |
| `session_snapshot`  | Persists and restores full pair state; supports session recovery after crash/restart                  |
| `skill_discovery`   | Scans project dirs for `.md` skill files with YAML frontmatter                                        |
| `resource_monitor`  | Per-agent CPU/memory polling (1s interval)                                                            |
| `git_tracker`       | Detects modified/added/deleted files relative to a baseline commit                                    |
| `file_cache`        | Lists files and parses `@mention` references in task specs                                            |
| `path_env`          | Refreshes `$PATH` from login shell so CLI tools are discoverable                                      |
| `config_paths`      | Resolves platform-specific config file locations                                                      |
| `stubs`             | Placeholder Tauri commands not yet fully implemented                                                  |

## Frontend Components (`src/renderer/src/components/`)

| Component                                       | Purpose                                                                                            |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `App.tsx`                                       | Root: routing between dashboard and pair detail views                                              |
| `AppChrome.tsx`                                 | Window chrome, title bar, theme toggle                                                             |
| `OnboardingWizard.tsx`                          | First-run guided setup (model config, directory selection)                                         |
| `CreatePairModal.tsx`                           | New pair creation form                                                                             |
| `PairSettingsModal.tsx`                         | Edit existing pair settings                                                                        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timwuhaotian/the-pair](https://github.com/timwuhaotian/the-pair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
