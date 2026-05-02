---
trigger: always_on
description: 1. Think Before Coding
---

# AGENTS

## You Must Obey

1. Think Before Coding
Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:

State your assumptions explicitly. If uncertain, ask.
If multiple interpretations exist, present them - don't pick silently.
If a simpler approach exists, say so. Push back when warranted.
If something is unclear, stop. Name what's confusing. Ask.
2. Simplicity First
Minimum code that solves the problem. Nothing speculative.

No features beyond what was asked.
No abstractions for single-use code.
No "flexibility" or "configurability" that wasn't requested.
No error handling for impossible scenarios.
If you write 200 lines and it could be 50, rewrite it.
Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

3. Surgical Changes
Touch only what you must. Clean up only your own mess.

When editing existing code:

Don't "improve" adjacent code, comments, or formatting.
Don't refactor things that aren't broken.
Match existing style, even if you'd do it differently.
If you notice unrelated dead code, mention it - don't delete it.
When your changes create orphans:

Remove imports/variables/functions that YOUR changes made unused.
Don't remove pre-existing dead code unless asked.
The test: Every changed line should trace directly to the user's request.

4. Goal-Driven Execution
Define success criteria. Loop until verified.

Transform tasks into verifiable goals:

"Add validation" → "Write tests for invalid inputs, then make them pass"
"Fix the bug" → "Write a test that reproduces it, then make it pass"
"Refactor X" → "Ensure tests pass before and after"
For multi-step tasks, state a brief plan:

1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

These guidelines are working if: fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## What this repo is

Windows OEM vendor-key remapping tool.

It has two runtime processes:
- `Controller`: WinUI 3 desktop app for editing config and controlling the service
- `Worker`: background process that listens for OEM events and executes actions

The UI edits config only.
The Worker performs runtime behavior.

---

## Repo layout

- `src/MeowBox.Controller/`
  WinUI 3 control panel
- `src/MeowBox.Worker/`
  background worker, tray, OSD, IPC host
- `src/MeowBox.Core/`
  shared models, IPC contracts, config, native/system services
- `src/MeowBox.Setup/`
  WiX MSI installer project
- `assets/`
  packaged assets and default config templates
- `build/`
  all intermediate outputs and package staging
- `artifacts/`
  final distributables (`MeowBox/`, optional `.zip`, `.msi`)

Do not introduce new top-level output folders unless truly necessary.

---

## Current product shape

Controller pages:
1. `Mappings`
2. `Touchpad`
3. `Battery`
4. `Settings`

The app should feel like a compact desktop utility, not an admin console.

---

## Runtime model

### Controller

Main coordinator:
- `src/MeowBox.Controller/Services/MeowBoxController.cs`

Worker process startup:
- `src/MeowBox.Controller/Services/WorkerProcessService.cs`

IPC client:
- `src/MeowBox.Controller/Services/WorkerPipeClient.cs`

### Worker

Main host:
- `src/MeowBox.Worker/WorkerHost.cs`

Support services:
- `src/MeowBox.Worker/Services/WorkerPipeServer.cs`
- `src/MeowBox.Worker/Services/TrayIconService.cs`
- `src/MeowBox.Worker/Services/WorkerOsdService.cs`

Worker execution flow:
1. receive `InputEvent`
2. match against configured `Keys`
3. resolve enabled `Mappings`
4. execute the selected `Action`

---

## Config model

Runtime config path:
- `%LocalAppData%\MeowBox\config.json`

Primary model:
- `src/MeowBox.Core/Models/AppConfiguration.cs`

Config service:
- `src/MeowBox.Core/Services/AppConfigService.cs`

Current schema centers on:
- `Theme`
- `Preferences`
- `Touchpad`
- `Keys`
- `Mappings`

Do not reintroduce legacy config compatibility unless explicitly requested.
This project currently prefers a single clean schema.

---

## Localization architecture

The current localization system must stay on one lifecycle.

### Source of truth

- localized XAML UI text uses WinUI resource resolution via `x:Uid`
- localized code-behind / view-model text uses `MeowBox.Core.Services.ResourceStringService`
- localized language selection / resolution uses:
  - `src/MeowBox.Core/Services/AppLanguageService.cs`
  - `Microsoft.Windows.Globalization.ApplicationLanguages.PrimaryLanguageOverride`

### Required lifecycle

Always resolve the stored language preference to one concrete effective language tag first.

Current flow:
1. read stored preference from config
2. resolve it to a concrete tag (`en-US` or `zh-CN`)
3. apply that same tag to `.NET` culture through `AppLanguageService`
4. apply that same tag to WinUI through `Microsoft.Windows.Globalization.ApplicationLanguages.PrimaryLanguageOverride`

Rules:
- do not split localization into separate `.NET` vs WinUI language lifecycles
- do not keep a special empty-string `System` branch for WinUI after `.NET` already resolved a concrete language

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leehyukshuai/Meow-Box](https://github.com/leehyukshuai/Meow-Box) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
