---
trigger: always_on
description: **Generated:** 2026-05-09
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-05-09
**Commit:** 7fd9284
**Branch:** master

## OVERVIEW

Obsidian Salt Dashboard is an Obsidian plugin built with React 18, TypeScript, esbuild, and a microkernel-style module registry. Core runtime lives in `src/app`; dashboard features live as self-contained modules under `src/modules`.

## PROJECT WORKFLOW

- Refer to `PRD.md` for feature scope; update it only when feature scope or behavior changes.

## STRUCTURE

```text
obsidian-salt-dashboard/
├── src/app/       # Obsidian plugin lifecycle, settings, layout, registry
├── src/modules/   # Built-in dashboard modules
├── src/shared/    # Internal shared components, utilities, constants
├── src/i18n/      # i18next setup and locale resources
├── examples/      # External single-file JSX/JS module examples
├── docs/          # User/architecture/custom plugin documentation
├── assets/        # README images
└── build/         # Generated plugin bundle artifacts
```

## WHERE TO LOOK

| Task                   | Location                                  | Notes                                                                  |
| ---------------------- | ----------------------------------------- | ---------------------------------------------------------------------- |
| Plugin lifecycle       | `src/app/main.tsx`                        | Registers modules, view, command, settings tab, external plugin loader |
| Obsidian view root     | `src/app/view/HomepageView.tsx`           | Hosts the dashboard view lifecycle                                     |
| React root             | `src/app/App.tsx`                         | Thin wrapper around layout                                             |
| Grid behavior          | `src/app/layout/GridLayout.tsx`           | Responsive `react-grid-layout`, drag handle, resize handles            |
| Module contract        | `src/app/architecture/DashboardModule.ts` | `id`, `settingsKey`, defaults, component, settings renderer            |
| Module registry        | `src/app/registry/ModuleRegistry.ts`      | Merges default settings and notifies registry subscribers              |
| Settings UI            | `src/app/settings/SettingsTab.ts`         | Global settings plus each module's `renderSettings`                    |
| Defaults               | `src/shared/constants.ts`                 | `VIEW_TYPE_HOMEPAGE`, `DEFAULT_SETTINGS`, default layout               |
| Query filters          | `src/shared/utils/SourceParser.ts`        | Dataview-style source parser for files/tags/frontmatter                |
| Built-in modules       | `src/modules/*/index.ts`                  | One `DashboardModule` definition per module                            |
| Todo domain            | `src/modules/todo/`                       | Nested daily/regular/shared module family                              |
| Custom plugin examples | `examples/*.jsx`                          | Self-contained external module reference, not importable API           |

## CODE MAP

| Symbol               | Type           | Location                                                  | Role                                                      |
| -------------------- | -------------- | --------------------------------------------------------- | --------------------------------------------------------- |
| `HomepagePlugin`     | class          | `src/app/main.tsx`                                        | Obsidian plugin entry and runtime coordinator             |
| `DashboardModule`    | interface      | `src/app/architecture/DashboardModule.ts`                 | Module/public extension shape                             |
| `registry`           | singleton      | `src/app/registry/ModuleRegistry.ts`                      | Built-in/external module registry                         |
| `PluginLoader`       | class          | `src/app/services/PluginLoader.ts`                        | Loads `.js/.cjs/.jsx` dashboard modules from vault folder |
| `LayoutManager`      | class          | `src/app/services/LayoutManager.ts`                       | Syncs layout entries with registered modules              |
| `SourceParser`       | class          | `src/shared/utils/SourceParser.ts`                        | Parses source strings for recent/random file filtering    |
| `TodoBaseService`    | abstract class | `src/modules/todo/shared/services/TodoBaseService.ts`     | Shared task parsing/sorting/file mutation base            |
| `DailyTodoService`   | class          | `src/modules/todo/daily/services/DailyTodoService.ts`     | Daily task file + stats JSON coordination                 |
| `RegularTodoService` | class          | `src/modules/todo/regular/services/RegularTodoService.ts` | Markdown checkbox task mutation                           |
| `RecentFilesService` | class          | `src/modules/recent-files/services/RecentFilesService.ts` | Query/filter/sort/pin/create/delete recent files          |

## CONVENTIONS

- Package manager is npm; `package-lock.json`, CI, Release, and scripts all use npm.
- Node version is `>=20.0.0`; CI uses Node 20.
- TypeScript uses `baseUrl: "."`, `allowJs`, `isolatedModules`, `strictNullChecks`, `noImplicitAny`, `jsx: react-jsx`; no `paths` aliases.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kuro96/obsidian-salt-dashboard](https://github.com/Kuro96/obsidian-salt-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
