---
trigger: always_on
description: **Role**: Senior Principal Flutter Engineer & Architect.
---

# Kuron App — GitHub Copilot Instructions

**Role**: Senior Principal Flutter Engineer & Architect.
**Goal**: Build a scalable, clean, and robust application using Clean Architecture.
**App**: Kuron (repo: nhasixapp) — Mobile reading app with privacy features.

## 📝 Project Memory
**CRITICAL**: Read `MEMORY.md` at project root for full context. Update it after every session.

## 🤖 AI Tool Compatibility

### 🚀 RTK - Rust Token Killer (Core Rule)

**Goal**: Maximize token efficiency (60-90% savings) by filtering terminal noise.

1. **Always use `rtk` prefix**: For all terminal commands that generate significant output (git, flutter, npm, ls, etc).
2. **Meta Commands**: Use `rtk gain` to check savings, `rtk proxy <cmd>` for raw output.
3. **Integration**: Follows `~/.gemini/GEMINI.md` global rules.

### Claude Code
- **Primary instruction file**: `CLAUDE.md` (auto-loaded by Claude Code).

### Codex
- **Primary instruction file**: `AGENTS.md`.
- Prefer project-local Codex skills in `.codex/skills/*/SKILL.md`.
- Read `.opencode/skills/*/SKILL.md` as primary local skill source when the task matches a listed skill.

### OpenCode / Aider / Other Agents
- Read `.opencode/skills/*/SKILL.md` as primary skill source.
- Use `.agent/skills/*/SKILL.md` as secondary references.

### General Rules (All AI Tools)
- Reuse existing project rules instead of creating parallel rule sets.
- Keep `.github/copilot-instructions.md` aligned with this file; if guidance overlaps, follow the stricter local project rule.
- Startup sequence: `MEMORY.md` -> active `openspec/changes/` (non-archived) -> `proposal.md` + `tasks.md`.

## ⚡ Core Commands

This project uses **FVM** (Flutter Version Management). Always prefix with `fvm`:

- **Build/Run**: `fvm flutter clean && fvm flutter pub get` | `fvm flutter run --debug` | `fvm flutter build apk --release`
- **Build with flavor**: `fvm flutter build apk --release --flavor prod --dart-define=cronetHttpNoPlay=true` | `--flavor dev` (dev label "Kuron Dev", appId `id.nhasix.app.dev`)
- **Required dart-define**: `--dart-define=cronetHttpNoPlay=true` — Cronet embedded (no Play Services); REQUIRED on AGP 9 for the org.chromium.net namespace conflict. Always pass it. `scripts/build_optimized.sh` includes it.
- **Build Optimized**: `./scripts/build_optimized.sh debug` | `release` (adds flavor + dart-define)
- **Test/Lint**: `fvm flutter test` | `fvm flutter analyze` | `fvm dart run build_runner build`
- **Codegen**: `fvm flutter pub run build_runner build --delete-conflicting-outputs`
- **Format**: `fvm dart format .`
- **Analysis (single file)**: `fvm dart analyze <path>`
- **Release**: `./build_release.sh` (Custom) | `fvm flutter build ipa` (iOS)- **Packages Pub Get**: `./scripts/pub_get_all.sh` | Run `fvm flutter pub get` on all packages

## 📜 Development Scripts

Located in `scripts/` folder. **ALWAYS run after project changes:**

| Script | Command | When to Run |
|---|---|---|
| **Project Status** | `dart scripts/project_status.dart` | After creating issue, moving project phase, or updating progress |
| **Create Feature** | `dart scripts/create_feature.dart [name]` | Scaffold new feature structure |
| **Smart Search** | `./scripts/smart_search.sh <mode> <pattern>` | Code search, audit, violations |
| **Pub Get All** | `./scripts/pub_get_all.sh` | Run `fvm flutter pub get` on all packages at once |

**Note**: `project_status.dart` auto-updates all README dashboards with progress bars and statistics.

## 🧠 Development Workflow (CRITICAL)

We operate with professional discipline. Code is ephemeral; Architecture is permanent.

### The Project Lifecycle
**Never write code without a plan.**

#### 🧭 Active Steering (Automatic Context)
**CRITICAL**: At the start of every session, YOU MUST:
1. Read `MEMORY.md` for cross-session context.
2. Check `openspec/changes/` for active (non-archived) changes.
3. If an active change exists, READ its `proposal.md` and `tasks.md` immediately.
4. **Treat `tasks.md` as the Master Plan**. Do not implement features not listed there.
5. **Update `tasks.md`** automatically as tasks are completed.

#### Phases (OpenSpec Workflow):

1. **Exploration / Analysis**
   - Use `/opsx-explore` or create change via `openspec new change "<name>"`
   - Folder: `openspec/changes/<name>/`
   - Main artifact: `proposal.md`
   - **READ-ONLY**: Document findings only. No code changes.
   - **STOP & WAIT**: Do not implement without explicit user approval.

2. **Execution** (active `openspec/changes/<name>/`)
   - Requires `tasks.md` (generated via `/opsx-apply`)
   - **MUST** work through tasks.md in order. Mark `[x]` on completion.
   - **Use MCP**: `Sequential Thinking`, `Context7`, `Docfork` for complex tasks.
   - **Approval**: Only start after explicit user approval.

3. **Completion** (archived to `openspec/changes/archive/<date>-<name>/`)
   - Run `openspec archive -y` or move folder manually.
   - **Git**: DO NOT run `git add/commit`. User handles source control.

## 🛠 Code Standards

### Architecture
- **Clean Architecture**: `Domain` (Pure Dart) ← `Data` (Impl/API) ← `Presentation` (Flutter).
- **DI**: `get_it` + `injectable` via `core/di/`.
- **Strict Layers**: UI never talks to Data. UI talks to Cubit -> UseCase -> Repository.

### Naming & Style
- **Files**: `snake_case`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shirokun20/kuron-mobile](https://github.com/shirokun20/kuron-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
