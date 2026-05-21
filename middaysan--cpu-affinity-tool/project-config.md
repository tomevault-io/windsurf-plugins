---
trigger: always_on
description: This file records the actual repository structure, platform boundaries, runtime architecture, build and release contract, and the repo-specific workflow rules that must stay true for this project.
---

# AGENTS.md for `cpu-affinity-tool`

## Purpose
This file records the actual repository structure, platform boundaries, runtime architecture, build and release contract, and the repo-specific workflow rules that must stay true for this project.

Keep it truthful. If architecture, CI, release flow, platform scope, or important repository structure changes, update `AGENTS.md` in the same change.

## Repo workflow contract
This repository uses the staged workflow standard with this file as the canonical repo contract.

Workflow facts:
- canonical repo contract: root `AGENTS.md`
- optional local overlay: `.codex/AGENTS.md`
- canonical local stage artifact: `.codex/ROADMAP.md`
- local user-facing roadmap content may be written in Russian
- repo `workflow_mode`: `staged-default`

Overlay rules:
- `.codex/AGENTS.md` may exist only as a local additive overlay
- it may not contradict facts or restrictions from this file
- it may only tighten workflow activation through `workflow_override: inherit | explicit-only`
- it may not weaken repo-shared policy

Roadmap identity rules:
- stages use immutable `stage_id` values such as `S00`, `S01`, `S02`
- display order numbers are convenience only
- once `.codex/ROADMAP.md` records a roadmap mutation, `stage_id` becomes the canonical stage reference
- legacy root `ROADMAP.md` and `ROADMAP_PROMPTS.md` may exist as ignored local convenience files, but they are not canonical workflow artifacts

Freshness rules:
- root `AGENTS.md` is always reread at session start and on `Status`
- if assistant detects drift between this file and repo reality, it must flag the conflicting section and carry the tracked update in the next relevant repo change
- `.codex/ROADMAP.md` owns only stages, statuses, deferred items, residual risks, freshness metadata, and append-only roadmap-change history

## Project and platform status
`cpu-affinity-tool` is a desktop utility for managing CPU affinity and process priority.

Repository binaries:
- `cpu-affinity-tool` - primary Windows binary
- `cpu-affinity-tool-linux` - feature-gated Linux entrypoint

Current platform reality:
- Windows is the primary released and explicitly supported platform
- Linux code exists as a CI build/test/clippy validated desktop beta path from source for `x86_64` `glibc`; desktop sessions on `X11` or `Wayland` are covered by manual beta smoke validation
- Linux also has a separate beta prerelease artifact contract under `linux-beta-v*` tags, but it is not part of the stable release contract
- the project must not be described as a fully cross-platform desktop app

## Repository map
Key directories:
- `src/` - application runtime code and entrypoints
- `src/app/shell/` - the top-level `eframe::App` shell, route enums, transient UI sessions, typed shell events, and presenter module ownership
- `src/app/features/` - bounded feature modules for `rules`, `execution`, `preferences`, `topology`, and `diagnostics`
- `src/app/adapters/` - seams for persisted state loading, OS helpers, and installed-app discovery
- `src/app/runtime/` - thin composition-root state facade kept around `AppState`
- `src/app/models/` - persisted schema, domain and runtime-independent data types, CPU preset and meta helpers, `LogManager`, and running-app tracking structures
- `src/app/models/app_state_storage/` - internal persistence modules for state path resolution, storage I/O, migrations, and schema refresh; `app_state_storage.rs` remains the public storage schema and API entrypoint
- `libs/os_api/` - platform boundary for OS-specific operations; Windows internals are split under `libs/os_api/src/windows/`, while Linux remains a single-file desktop beta backend
- `assets/` - icon, screenshot, `cpu_presets.json`, and social-preview guidance
- `docs/` - release/process documentation and user-facing comparison/rationale references
- `.github/workflows/` - CI and GitHub Release automation
- `changelogs/` - manual release notes

Important root files:
- `Cargo.toml` - package metadata, binaries, features, dependencies
- `LICENSE` - MIT license
- `build.rs` - Windows resource embedding and rebuild hooks
- `app.manifest` - embedded Windows manifest with elevated privilege model
- `Makefile.toml` - local developer automation wrapper
- `README.md` - user-facing project description
- `CHANGELOG.md` - consolidated human-facing release history
- `CONTRIBUTING.md` - contribution workflow and review expectations
- `SECURITY.md` - private security reporting policy
- `SUPPORT.md` - support routing and diagnostics expectations
- `.github/ISSUE_TEMPLATE/` - structured issue intake for bugs and feature requests
- `docs/comparison.md` - comparison with Task Manager, Process Lasso, and CLI workflows
- `docs/why.md` - rationale, limits, and non-goals of affinity management
- `docs/release-checklist.md` - manual checklist for the current Windows-only release contract
- `docs/linux-beta-release-checklist.md` - manual checklist for the Linux beta prerelease contract
- `docs/release-process.md` - current tag-based stable Windows release flow plus Linux beta prerelease flow and release-notes template

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [middaysan/cpu-affinity-tool](https://github.com/middaysan/cpu-affinity-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
