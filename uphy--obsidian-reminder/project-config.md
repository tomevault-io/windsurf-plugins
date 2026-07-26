---
trigger: always_on
description: A plugin that adds reminder functionality to Obsidian's TODO lists. Built with TypeScript + Svelte 4, bundled with esbuild.
---

# obsidian-reminder

A plugin that adds reminder functionality to Obsidian's TODO lists. Built with TypeScript + Svelte 4, bundled with esbuild.

## Language

Write all repository artifacts — code comments, commit messages, documentation — in English.

## Commands

Prefer mise tasks (they just wrap npm scripts under the hood).

| mise | Underlying command (npm script) | Purpose |
| --- | --- | --- |
| `mise run main:init` | `npm install` | Install dependencies |
| `mise run main:build` | `npm run build` (`node esbuild.config.mjs production`) | Run a production-equivalent build once |
| `mise run dev` | `npm run dev` (`node esbuild.config.mjs watch`) | Start a persistent development watch build |
| `mise run main:test` | `npm run test` (`jest`) | Run the full test suite |
| `mise run main:lint:fix` | `npm run lint:fix` | eslint --fix + tsc --noEmit + svelte-check |
| `mise run pre-commit` | `main:lint:fix` → `main:test` | Pre-commit checks |
| `mise run docs` | `npm run dev` in `docs/` | Run the documentation site (VuePress) locally |

`esbuild.config.mjs` uses esbuild's context API: with no arguments it runs a development build once and exits, with `watch` the process stays resident and automatically rebuilds on every file change, and with `production` it runs a production-equivalent build (minified, no sourcemap) once and exits.

To run a single test, call jest directly.
```
npx jest src/model/format/reminder-default.test.ts
npx jest -t "parse - link dates to daily notes"
```

## Architecture

The entry point is `src/main.ts` (`ReminderPlugin`). `src/` is organized into three layers.

- `src/model/` — Domain logic with no dependency on the Obsidian API: `Reminder`/`Reminders` (`model/reminder.ts`), date/time handling via `DateTime`/`Time` (`model/time.ts`), and the reminder-syntax parsers for Markdown (`model/format/`). This is the only layer with direct jest unit tests (because it never imports the `obsidian` module).
- `src/plugin/` — The plugin body, which depends on the Obsidian API. Bundles file watching, notifications, settings, and command registration (`plugin/index.ts` is the export entry point).
- `src/ui/` — Svelte components (`ui/*.svelte`) and the TS that supports them (e.g. calendar calculations). The Obsidian Modal/View code in `src/plugin/ui/` mounts the Svelte components.

Flow from reminder parsing to notification:
1. `src/plugin/filesystem.ts` (`ReminderPluginFileSystem`) watches the vault's modify/delete/rename events.
2. Changed files are passed to `src/model/content.ts` (`Content`), and `getReminders()` calls `parseReminder()` in `model/format/index.ts`.
3. `parseReminder()` goes through `CompositeReminderFormat` (`model/format/reminder-base.ts`), trying each enabled format in turn (the default format `reminder-default.ts`, the Tasks plugin format `reminder-tasks-plugin.ts`, the Kanban plugin format `reminder-kanban-plugin.ts`) to build an array of `Reminder`.
4. The result is stored in `Reminders` in `model/reminder.ts`, and a callback in `main.ts` notifies the UI whenever it changes.
5. `src/plugin/notification-worker.ts` (`NotificationWorker`) runs periodically via `registerInterval`, detects expired reminders with `Reminders.getExpiredReminders()`, and displays them through `ReminderPluginUI.showReminder()` in `plugin/ui/index.ts` → `ReminderModal` in `plugin/ui/reminder.ts` (which internally uses `ui/Reminder.svelte`).
6. The reminder list is shown by a custom View in `plugin/ui/reminder-list.ts` (`ui/ReminderList.svelte`, etc).

Settings are declared by `Settings` in `src/plugin/settings/index.ts` using a builder DSL called `SettingTabModel` (`plugin/settings/helper.ts`), and persisted by `PluginData` in `src/plugin/data.ts` via `loadData`/`saveData`.

## Local development

Symlink the dev vault's `.obsidian/plugins/obsidian-reminder-plugin` to this repository, and install and enable the hot-reload plugin ([pjeby/hot-reload](https://github.com/pjeby/hot-reload)) at `.obsidian/plugins/hot-reload`. hot-reload watches any plugin directory that contains `.git` or `.hotreload`, and automatically reloads just that plugin whenever it detects a change to `main.js`/`styles.css`.

With this set up, running `mise run dev` keeps esbuild's watch build resident, so every time you save a source file it automatically rebuilds and reloads in Obsidian. No manual copying or restarting Obsidian is needed.

Automated tests only cover `src/model/`. After implementing a change that affects runtime behavior in Obsidian (anything under `src/plugin/` or `src/ui/`, or parsing behavior visible in the vault), run the `manual-verify` skill to set up the test vault and walk the user through verification before opening a PR. A Stop hook (`.claude/hooks/manual-verify-reminder.sh`) reminds about this once per change state.

## Release flow

`release.sh` has been removed. Releases are done exclusively via `.github/workflows/release.yml` and `release-beta.yml`, both triggered manually via `workflow_dispatch`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uphy/obsidian-reminder](https://github.com/uphy/obsidian-reminder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
