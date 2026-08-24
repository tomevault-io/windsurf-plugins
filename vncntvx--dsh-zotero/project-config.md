---
trigger: always_on
description: - You work in `.` This repo is a standalone sibling to `../deepseek-harness`. You keep it beside the harness for local dev. Treat this as temp local layout, hard-coded for your machine.
---

# AGENTS.md

## Workspace and source of truth

- You work in `.` This repo is a standalone sibling to `../deepseek-harness`. You keep it beside the harness for local dev. Treat this as temp local layout, hard-coded for your machine.
- You find the dsh source in `../deepseek-harness`. Read `../deepseek-harness/docs/AGENTS.md`, `../deepseek-harness/docs/architecture.md`, `../deepseek-harness/docs/user/develop/`, `../deepseek-harness/docs/subsystems/`, `../deepseek-harness/docs/cookbook/`, and `../deepseek-harness/packages/*/README.md` when you touch harness contracts.
- You touch slots, services, the web shell, the client-module graph, or Typert, so you check harness source first. Harness source outranks this file.

## Git commit conventions

Follow [Conventional Commits](https://www.conventionalcommits.org/): `<type>(<scope>): <subject>`, types lowercase (`feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `style`, `build`, `ci`, `revert`), subject in imperative mood, header under 72 chars. Optionally place an emoji matching the type right after the colon, before the subject. Body: blank line, bullet points only, each item wraps at 72, what and why.

## Commands

```sh
npm install                  # sibling at ../deepseek-harness; add --no-workspaces only for a nested copy
npm run typecheck            # tsc --noEmit for the node, test, and client projects
npm test                     # vitest unit tests against the mock Zotero server
npm run test:coverage        # 100% coverage gate on src/; src/index.ts and src/types.ts excluded (pure re-export / types-only)
npm run build                # tsc emits the node half into lib/; esbuild emits the browser half lib/client.js (with a loader-handoff self-check)
npm run build:client         # rebuild the browser half only
npm run dev                  # tsc --watch
npm run dev:client           # esbuild --watch for the browser half
npm run format               # prettier --write across the repo
npm run format:check         # verify formatting; run before committing
npm run test:integration     # live Zotero at 127.0.0.1:23119; skipped unless ZOTERO_INTEGRATION=1
```

## Plugin form

The loader mounts the default export (`ZoteroService`) with the row's validated config. `src/index.ts` stays a pure re-export entry.

- `ZoteroService extends Service` with `static inject = ['tools', 'systemPrompt']` and `static Config = ConfigSchema`. You declare it as `ctx.zotero` ([plugin forms](../deepseek-harness/docs/user/develop/basic/index.md), [services](../deepseek-harness/docs/user/develop/framework/service.md)).
- You install the `zotero` settings section in the constructor with `installSettingsSection` (composition entry as the base layer). The `config` getter reads a live source. Every committed section rebuilds the HTTP client and the `local` provider through `rebuild()`. Tools read `service.config` per request so validation limits follow edits. The namespace constant lives in `src/settings-namespace.ts` and you share it with the browser half.
- One package owns all three capability roles: definition (`ZoteroService` + the `ZoteroProvider` interface in `src/types.ts`), provider (`LocalApiProvider`), consumers (`src/tools/`). Split only when roles must evolve apart ([three-role design](../deepseek-harness/docs/user/develop/practice/index.md)).
- You register providers through `registerProvider()`. The call is effect-scoped. Duplicate ids throw. The service selects a provider by the `provider` config id and gates every domain call on its declared `capabilities`. No fallback across providers.

## Registrations are effects

You register everything in the constructor. The fiber unwinds it when you edit config or when HMR replaces the instance ([lifecycle](../deepseek-harness/docs/user/develop/framework/index.md)):

- Tools: you call `ctx.tools.register(defineTool(...))` from `@deepseek-ai/dsh-tools` ([tool tutorial](../deepseek-harness/docs/user/develop/basic/tool.md)).
- Prompt: you call `ctx.systemPrompt.section({ name, order, text })`.
- Command: you call `ctx.inject(['commands'], ...)`. The optional-dependency form keeps the plugin loadable in headless compositions without `commands`.
- Provider: you call `ctx.effect()`.
- Settings section: you call `installSettingsSection` (optional dependency, absent settings services leave the plugin on its entry config).
- Browser surface (`src/client/`, esbuild emits `lib/client.js` in `__ModuleLoader__.load` handoff format): you ship one configuration card over the `zotero` namespace. You read and write through the harness `settingsScope` binder. You do not import values across plugins. The card registers into the keyed `settings.plugin.item` slot under the namespace key. The Plugins config tab dispatches it. You render the staged form in the section's native disclosure chrome. You mirror `PluginCard` tokens, you do not value-import `PluginCard`. The Typert Remote namespace carries only the `zotero/status` connectivity probe for the conversation tab. The tab reads `webEnabled` live. You subscribe it to the same scope and it shows or hides as the flag changes.

## Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vncntvx/dsh-zotero](https://github.com/Vncntvx/dsh-zotero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
