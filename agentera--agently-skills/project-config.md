---
trigger: always_on
description: Central catalog and documentation for Agently Skills V2. Use when working with Agently skill installation, routing, and installed-skill usage guidance.
---


# Agently Skills Catalog

This package publishes the current Agently Skills catalog generation `v2` under
`skills/`. Historical catalogs are preserved on frozen archive branches instead
of the default branch so coding-agent retrieval only sees the current catalog.

Use this file as installation-time guidance after the skills are added into another project or agent environment.

## Usage Priorities

- Route unresolved product, assistant, and workflow requests through `agently` first.
- Prefer Agently-native capabilities before custom output parsers, retry loops, or orchestration layers.
- Apply Occam's razor to APIs, architecture, and examples: do not add a new entity,
  method, facade, or compatibility patch when an existing Agently surface already
  carries the concept clearly. Prefer a narrow alias or docs clarification for unclear names.
- Default to async-first guidance for services, streaming, TriggerFlow, and concurrent execution. Treat sync APIs as wrappers for scripts, REPL use, or compatibility bridges unless there is a clear reason not to.
- Treat `agently-devtools` as an optional companion package installed from PyPI, not as a required source-repo dependency.
- Keep public skill boundaries capability-first and mutually exclusive.
- Treat multi-agent, judge, and review flows as scenario recipes unless they need a dedicated framework surface.
- For Agently framework internals, follow the core module style: class-owned
  runtime state, typed data contracts under `agently/types/data`, protocol or
  handler seams under `agently/types/plugins`, and retained implementation
  owners for Action, ExecutionEnvironment, TriggerFlow, and DynamicTask. A
  high-level capability can live outside `agently/core` when it composes
  several core systems; split large implementations by registry, planner,
  executor, adapter, facade, and contract boundaries rather than by arbitrary
  line count.
- Do not recommend archived catalog branches for new projects; they exist only
  for explicit rollback or historical inspection.

## Coordinated Release Rules

- For an Agently framework release, validate the main repository and companion repositories before merging or publishing.
- Feature acceptance is not complete until each relevant spec records the final
  implemented design, any fully landed planned spec has moved to `spec/implemented/`,
  and `spec/README.md` points to the completed location.
- When reporting public API, recommended usage, examples, or compatibility-line
  changes, include concise sample code that shows the updated usage shape.
  Prefer current usage snippets or before/after snippets over abstract prose
  when that will make the change easier to inspect.
- Version numbers are part of the release-prep change and must be updated before final validation, merging, or publishing; do not rely on post-publish metadata-only edits to trigger a release workflow.
- Development-line planning must not change package release numbers. In the
  main repository, do not update `pyproject.toml`, `agently/compatibility.py`,
  `compatibility/index.json` `latest_release`, or create
  `compatibility/releases/<future-version>.json` only to mark the next planned
  version. Use `compatibility/in-development.json` for the next target until an
  actual release-prep change begins.
- If new version-development work starts after the previous public version has
  already been released, treat `compatibility/in-development.json` as the
  current work version while no release-prep or release-promotion action has
  started for a newer batch. Do not ask the maintainer to restate that version
  on every task. If the in-development manifest is missing, stale, or being
  replaced, ask for the current work version. If the intended task branch is
  not specified, ask for the branch before editing implementation, specs, docs,
  examples, compatibility metadata, or companion guidance.
- The main repository release commit must update `pyproject.toml`, `agently/compatibility.py`, `compatibility/index.json`, and the matching `compatibility/releases/<version>.json`; keep `compatibility/in-development.json` aligned until the release line moves on.
- If the release recommends a new `agently-devtools` build, update the DevTools package version in `../Agently-Devtools/packages/python/pyproject.toml` during the same release-prep pass; changing only docs, tests, or compatibility text does not trigger the DevTools publish workflow.
- Keep the Agently DevTools `recommended_version_specifier` in the current release manifest aligned with the version that will be published to PyPI.
- Before creating or updating the main repository PR, run `pyright` and `python -m pytest` in the main repository, using the same Python environment that will validate the release.
- Before considering this Skills repository aligned, run `python validate/validate_compatibility.py`, `python validate/validate_catalog.py`, `python validate/validate_bundle_manifest.py`, `python validate/validate_trigger_paths.py`, and `python validate/validate_native_usage.py`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentEra/Agently-Skills](https://github.com/AgentEra/Agently-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
