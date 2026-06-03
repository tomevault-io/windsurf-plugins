---
trigger: always_on
description: > A Grafana app plugin for queryless exploration of profiling data stored in Pyroscope.
---

# Grafana Profiles Drilldown

> A Grafana app plugin for queryless exploration of profiling data stored in Pyroscope.

## How these files fit together

| File                                 | What it's for                                                                                                                   |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| **`AGENTS.md`** (this file)          | **Entry point.** Pyroscope & profiling workflow, expected vs bug, Scenes patterns, security. Points to every other doc below.   |
| **`.config/AGENTS/instructions.md`** | **Plugin tooling only** — webpack, `plugin.json`, E2E, rules about `.config`.                                                   |
| **`docs/project-intent.md`**         | **Why** we built the app — philosophy, principles. Use when reasoning about tradeoffs or scope.                                 |
| **`docs/application-structure.md`**  | **How the product is organized** — user journeys, views, exploration types, links in/out. Use when changing UI or URL behavior. |
| **`docs/sources/`**                  | **Shipped user docs** (get started, concepts, choose a view, investigate). Use when updating customer-facing copy.              |

**Code anchors:** Explore shell: `src/pages/ProfilesExplorerView/ProfilesExplorerView.tsx` → `SceneProfilesExplorer`. Root scene: `src/pages/ProfilesExplorerView/components/SceneProfilesExplorer/SceneProfilesExplorer.tsx`. Exploration views: `SceneExploreAllServices`, `SceneExploreServiceProfileTypes`, `SceneExploreServiceLabels`, `SceneExploreServiceFlameGraph`, `SceneExploreDiffFlameGraph`, `SceneExploreFavorites` (same `components/` tree). Function details / flame graph: `SceneExploreServiceFlameGraph/SceneFlameGraph.tsx`, `SceneFunctionDetailsPanel`. Shared utils: `src/shared/`, `src/pages/ProfilesExplorerView/domain/`, `src/pages/ProfilesExplorerView/helpers/`. Use these to open the right file first; avoid broad repo search for small changes. Use grep or codebase search before opening large files (e.g. 400+ lines) unless you need full context.

## When to read which doc (shallow vs full)

**Shallow** = Use the table above; only open a doc if the task clearly needs it. **Full** = Read the whole doc before making changes.

| Task type                                                                        | Read fully                                            | Shallow or skip                                     |
| -------------------------------------------------------------------------------- | ----------------------------------------------------- | --------------------------------------------------- |
| **Tiny edit** — typo, single component, rename, lint                             | This file (table + Scenes bullets if touching scenes) | project-intent, application-structure, instructions |
| **UI / URL / views** — new exploration type, URL param, flame graph or diff view | `docs/application-structure.md`                       | project-intent unless scope changes                 |
| **Scope or principles** — "should we add X?", new feature                        | `docs/project-intent.md`                              | application-structure unless UI changes             |
| **Build / plugin** — plugin.json, webpack, .config, E2E                          | `.config/AGENTS/instructions.md`                      | project-intent, application-structure               |
| **Shipped user docs** — get-started, concepts, structure                         | Relevant file in `docs/sources/` + `docs/README.md`   | Others unless aligning to UI                        |
| **Bug in profiles / Pyroscope / data**                                           | This file (Pyroscope links) + code                    | application-structure only if UI or URL involved    |

**Default:** Stay shallow; open another doc only when the task clearly fits. For renames, lint, or single-file UI tweaks, skip application-structure and other deep docs.

**`docs/application-structure.md` by topic** (open the section you need instead of the whole doc when possible):

- [Entry points & extension links](docs/application-structure.md#entry-points) · [Direct URLs / URL state](docs/application-structure.md#direct-urls)
- [Main exploration layout](docs/application-structure.md#main-exploration-layout) (header, filters, exploration types)
- [Exploration types / views](docs/application-structure.md#exploration-types-views)
- [Flame graph & function details](docs/application-structure.md#flame-graph-and-function-details)
- [Exit & handoffs](docs/application-structure.md#exit-points-and-handoffs)

## Before investigating issues or bugs

**Read relevant documentation before making code changes or proposing fixes.**

### Pyroscope and profiling documentation

[Pyroscope docs](https://grafana.com/docs/grafana/latest/datasources/pyroscope/) · [Grafana profiling](https://grafana.com/docs/grafana/latest/explore/simplified-exploration/profiles/). Profile types, labels, and flame graphs — know expected behavior before changing anything.

### Determine expected behavior first


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/profiles-drilldown](https://github.com/grafana/profiles-drilldown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
