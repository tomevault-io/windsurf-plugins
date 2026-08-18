---
trigger: always_on
description: Flow Shuttle / 流梭 is a local-first desktop app. Make incremental, focused changes and keep user data safe.
---

# Flow Shuttle Development Guidelines

Flow Shuttle / 流梭 is a local-first desktop app. Make incremental, focused changes and keep user data safe.

## Read First

Before doing any work, read:

1. This `AGENTS.md`
2. `LOOPS.md` if it exists in the project root
3. The task-related source, docs, or design files
4. The current `git status` and relevant `git diff`

If `LOOPS.md` is present, follow it for non-trivial tasks. If `LOOPS.md` is not present, follow the Default Execution Loop in this file.

Do not rely only on memory. Always re-read current project files before changing anything.

## Default Execution Loop

1. Read the current state.
2. Restate the task contract, including done criteria and out-of-scope work.
3. Make a minimal plan.
4. Change only necessary files.
5. Verify with available commands or clear manual checks.
6. Read logs, traces, and diffs when something fails.
7. Repair only when the cause is clear.
8. Stop and report if the contract is unclear or risk is too high.
9. End non-trivial tasks with a Loop Report.

## Project Context

Flow Shuttle / 流梭 is a local-first Electron desktop app for personal work progress journaling. It uses React, TypeScript, SQLite, `electron-vite`, `electron-builder`, NSIS, and GitHub Releases.

Core areas include:

- Today: daily work page where each work item has one editable daily block.
- Reports: daily, weekly, and monthly reports generated from local records.
- Heatmap: activity calendar based on real recorded work items.
- Projects: project lists, project detail pages, project timelines, and project notes.
- Settings: appearance, language, data directory, version, and update settings.
- AI assistance: user-owned API key for summaries and report assistance.
- Auto update: weak update notification based on electron-builder / electron-updater / GitHub Releases / NSIS.

## Must Preserve

- Keep Flow Shuttle local-first. Do not upload user work data by default.
- Keep user data safe. Be extremely careful with save logic, data directory behavior, migrations, and report generation.
- Keep AI features based on user-owned API keys unless a task explicitly says otherwise.
- Markdown report/export content must remain complete and must not be truncated.
- When changing UI text, update or check i18n entries.
- Do not change SQLite schema, migrations, IPC, preload, main process, save logic, report generation, or data directory behavior unless the task explicitly asks for it.
- Do not rewrite unrelated code or reorganize core directories for cleanup.
- Do not introduce new dependencies unless necessary and explained.
- Do not change license, release, or auto-update behavior unless the task explicitly asks for it.

## Protected Areas

Do not change these areas unless the current task explicitly requires it:

- SQLite schema and migrations
- IPC contracts and preload bridge
- Main process startup, security, and window behavior
- Save logic and data directory behavior
- Report generation and Markdown export
- Attachments and user data paths
- Auto-update release behavior

If a task touches these areas, read the related code and logs first, make the smallest safe change, and run the strongest available checks.

## Auto Update Constraints

When touching auto-update behavior, preserve the weak prompt flow:

- Use electron-builder / electron-updater / GitHub Releases / NSIS.
- Keep `autoDownload=false`.
- Keep `autoInstallOnAppQuit=false`.
- Skip real update checks in development.
- In packaged builds, check in the background after startup delay.
- The sidebar/bottom version area should only provide weak status hints.
- Download and restart/install actions should stay in Settings > Version & Update.
- Do not force download or force install updates.

## Public Release Notes

- Compare the final release candidate with the previous publicly released version.
- Include only capabilities that users of the previous public version will newly receive, and fixes for defects that existed in that public version.
- Do not publish development-only regressions, intermediate implementation bugs, QA/debug repairs, reference-screenshot adjustments, or refinements to never-released behavior as user-facing “fixes” or “improvements”.
- Keep internal troubleshooting and iteration history in task traces or commit history, not in public release notes.

## Working Rules

- Make incremental changes only.
- Do not rewrite unrelated code.
- Do not introduce new dependencies unless necessary and explained.
- Keep user data local-first.
- When changing UI text, check i18n entries.
- Markdown export must remain complete and must not be truncated.
- Before committing, run typecheck/build commands if available.
- For documentation-only tasks, do not modify business code, config, dependencies, lock files, or data files.
- Keep `LOOPS.md` as an optional local workflow file. Do not add it to version control unless the project owner explicitly asks.
- If current diffs include unrelated work, leave it alone and mention it in the report.

## Common Checks

Use the commands that exist in this repository. Do not invent commands.

- `npm run typecheck`
- `npm run build`
- `npm run dist:win`
- `npm run dist`
- `npm run dist:dir`
- `npm run dev`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sunyuanrui915/FlowShuttle](https://github.com/Sunyuanrui915/FlowShuttle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
