---
trigger: always_on
description: Echoglossian is a real-time FFXIV text translation plugin for Dalamud on .NET 10.
---

# AGENTS.md

## Echoglossian

Echoglossian is a real-time FFXIV text translation plugin for Dalamud on .NET 10.

Default goal: make the smallest correct change that fits the current architecture, preserves existing behavior unless explicitly asked otherwise, avoids regressions, and minimizes latency or UI instability.

## Priorities

- Preserve existing behavior unless the task explicitly requests a behavior change, migration, or refactor.
- Prefer the smallest correct change over broad rewrites.
- Optimize for low latency, stable UI behavior, backward-compatible persistence, and minimal-risk edits.
- Prefer reusable repo-local tooling for recurring work.
- Be concise and avoid restating repository guidance unless directly relevant.

## Architecture

- Prefer current architecture over legacy paths:
  - `NativeUI/AddonHandlers/...`
  - `NativeUI/Handlers/...`
  - `UIOverlays/TranslationOverlay/...`
- Reuse existing shared infrastructure before adding new paths:
  - `Translators/TranslationService`
  - existing async or brokered translation flow
  - shared caches
  - shared overlay renderer, sizing, and wrapping logic
- Do not create parallel translation queues, duplicate caches, or one-off handler infrastructure when shared solutions already exist.
- Always use resx translations for our plugin UI elements and Notifications.

## Translation Rules

Treat capture, translation, overlay rendering, and native mutation as separate stages.

- Overlay-only mode: capture source text, translate it, render in overlay only, and do not mutate native addon nodes, text nodes, or `AtkValue`s.
- Native mode: translated text may be written into the native UI.
- Swap mode: native UI shows translated text and the overlay shows the original text.

Do not restore or touch native state unless that code path actually mutated it.

## Performance

- Avoid using Reflection whenever possible.
- Avoid repeated work per frame.
- Cache, queue, or short-circuit repaint-heavy paths.
- Reuse prior translations when visible text already matches applied output.
- If translation fails or returns empty, do not retry every frame; cache failure or apply a cooldown.
- Keep logs quiet by default.

For dense or frequently repainted windows, prefer shared in-memory state and avoid retranslating already-applied text.

## Tooling

For inspection, validation, repo analysis, log parsing, repo-wide search, auditing, or other repeatable workflows:

- Prefer existing repo scripts over repeated manual steps.
- If no suitable script exists and the workflow is likely to recur, prefer adding a reusable repo-local script.
- Make scripts safe to re-run.
- Keep scripts scoped to real repo workflows.
- Document usage briefly in code comments or nearby docs.
- Do not add heavy tooling for a trivial one-off task.
- For PowerShell prefer a broad reusable command prefix for the session instead of narrow one-off approval prompts. Keep that prefix scoped to safe repo work such as build, test, search, and inspection commands.

When runtime behavior depends on Dalamud services, plugin startup, plugin-window hosting, font/ImGui behavior, or integration that pure unit tests cannot cover:

- Use `Echoglossian.Mock` and/or the DalaMock-backed harness when necessary and feasible.
- Prefer `Echoglossian.Mock.Tests` for hosted startup, shutdown, configuration, database-path, and plugin-window validation before relying on manual in-game checks.
- For behavior that reads real game data, Lumina sheets, FFXIVClientStructs-backed state, addon lifecycle events, `AtkValue`, `AtkUnitBase`, or native UI payload capture/application, validate with `Echoglossian.Mock`/DalaMock whenever feasible before claiming the behavior is covered.
- If the current harness cannot drive the needed game-data or native UI payload, extend `Echoglossian.Mock` or DalaMock first when the extension is practical; otherwise document the gap and keep the required in-game verification explicit.
- Do not claim `.Mock` validates capture/application unless the test actually drives the relevant mocked game-data, addon lifecycle, or native UI payload. Startup-only Mock tests prove wiring/load, not text capture or translation application.
- Use:
  - `dotnet build Echoglossian.Mock.Tests\Echoglossian.Mock.Tests.csproj -c Debug --no-restore`
  - `dotnet test Echoglossian.Mock.Tests\Echoglossian.Mock.Tests.csproj -c Debug --no-build -p:VSTestMaxCpuCount=1`
- If DalaMock cannot model the target game state or addon behavior, say so explicitly and note the remaining in-game verification.

For release submissions that update `DalamudPluginsD17` manifests:

- Never type commit hashes manually.
- Resolve the release commit with `git rev-parse <ref>`.
- Verify the same hash exists on the remote with `git ls-remote <remote> <hash>`.
- Only then write the `commit = "..."` field and open the official PR.

## Issue Workflow

- Treat each GitHub issue as its own branch by default.
- Prefer branch names in the form `issue-<number>-<short-slug>`.
- Do not incubate partial, unstable, or release-blocking issue work directly on `v4-series`.
- When issue work is not ready for release, extract or keep it on its issue branch and keep `v4-series` focused on releasable changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lokinmodar/Echoglossian](https://github.com/lokinmodar/Echoglossian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
