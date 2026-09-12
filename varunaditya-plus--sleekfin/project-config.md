---
trigger: always_on
description: These instructions apply to the entire repository. Treat `references/` as read-only research material unless the user explicitly names a reference repository as the target of a change.
---

# SleekFin Repository Instructions

These instructions apply to the entire repository. Treat `references/` as read-only research material unless the user explicitly names a reference repository as the target of a change.

## Priorities

1. Implement exactly what the user requested. Do not add adjacent features, broad cleanup, dependency upgrades, redesigns, or formatting sweeps.
2. Keep SleekFin specific to Jellyfin 12.0, .NET 10, and the Jellyfin Web `v12.0` contract unless the user explicitly requests a compatibility change.
3. Keep production code concise. Do not add abstractions, helpers, state, compatibility branches, dependencies, support files, or tests unless they are necessary for the requested behavior.
4. Inspect the relevant source, history, and surrounding conventions before editing. Do not infer behavior from filenames or issue labels alone.
5. Preserve unrelated user changes. Never overwrite, discard, stage, or commit work that is outside the current task.
6. Distinguish implemented, build-verified, manually tested, and live Jellyfin-verified outcomes. Never present one as another.

## Reference baseline

- Use SeerrFin as the structural reference: one solution, one Jellyfin plugin project, embedded vanilla JavaScript and CSS, a small plugin entry point, role-based folders, and version-triggered release automation.
- Reuse architectural patterns, not SeerrFin product behavior, names, selectors, API integrations, or legacy compatibility code.
- Check `references/jellyfin` and the exact `v12.0` state of `references/jellyfin-web` before relying on a Jellyfin API, DOM structure, route, layout behavior, or CSS class.
- Use the other reference plugins to compare integration patterns, not as authority over Jellyfin 12 source.
- Do not build, format, rename, or edit files under `references/` during ordinary SleekFin work.

## Required change workflow

For every feature, fix, refactor, style change, configuration change, documentation change, build change, or CI change:

1. Inspect the current worktree and the complete relevant code path.
2. Define the narrow change boundary. A working vertical slice may cross configuration, controller, service, model, JavaScript, and CSS files when all of them are required for one behavior.
3. Implement only that coherent change.
4. Run verification proportionate to the affected behavior.
5. Review the complete diff for correctness, accidental scope expansion, secrets, stale names, generated files, and unnecessary comments.
6. Report the changed files, verification results, and any unavailable live checks.
7. Propose an exact commit subject and ask the user for explicit permission to commit.
8. Commit only after that approval. If approval is declined or unavailable, leave the verified work uncommitted.

The original request to make an edit is not commit approval. Approval for an earlier commit is not approval for a later one.

## Repository organization

Keep production code under `src/Jellyfin.Plugin.SleekFin/` and preserve these boundaries:

| Location | Responsibility |
| --- | --- |
| `SleekFinPlugin.cs` | Plugin identity, Jellyfin lifecycle, configuration ownership, and dashboard page registration only. |
| `Configuration/` | Persisted settings, safe defaults, normalization, migrations, and dashboard configuration UI. |
| `Controllers/` | HTTP routes, authorization, input validation, request mapping, response mapping, and embedded asset delivery. |
| `Services/` | Application behavior, external integrations, stateful work, and background operations. |
| `Helpers/` | Focused stateless normalization, transformation, and reusable operations. |
| `Model/` | Request, response, transport, and internal data shapes. |
| `Inject/` | Browser-only JavaScript and CSS, split by cohesive feature responsibility. |
| `Properties/AssemblyInfo.cs` | Assembly identity and version. |
| `meta.json` | Installed plugin metadata. |
| `manifest.json` | Plugin repository/catalog metadata. |
| `.github/workflows/release.yml` | Version-triggered packaging and release automation. |

- Prefer one primary type per C# file. Closely coupled DTO item types may share a file when that improves discoverability.
- Add a subfolder only when a cohesive feature or domain has outgrown its current category.
- Keep controllers thin. Split a controller or service when it starts handling separate resources, integrations, or workflows.
- When dependency injection becomes necessary, use `PluginServiceRegistrator` as the single registration point. Do not create or retain an empty registrator.
- A new injected asset is incomplete until it is embedded in the project file, served by a controller, injected in the correct order, and verified in the built assembly.
- Update the README only when user-visible behavior, prerequisites, installation, configuration, compatibility, or troubleshooting changes.

## C# conventions

- Keep `<TargetFramework>net10.0</TargetFramework>` and Jellyfin package version `12.0.0` synchronized with the workflow and metadata.
- Keep nullable reference types and implicit usings enabled.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [varunaditya-plus/SleekFin](https://github.com/varunaditya-plus/SleekFin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
