---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

This is a single-machine interactive PowerShell deployment tool used by IT technicians to install applications and configure new Windows devices.

* Plain `.ps1` files only.
* No build system, package manager, CI pipeline, or compiled output.
* `Start.ps1` is the entry point.
* Modules are dot-sourced explicitly from `Start.ps1`.
* Application definitions live in `Config\Applications.json`.
* Offline installer binaries live under `Installers\` and are gitignored.

## Non-negotiables

1. Never log, print, commit, or persist secrets, credentials, CrowdStrike CID/token values, account passwords, or Office product keys.
2. Validation, availability, readiness, and detection functions must remain read-only.
3. `Config\Applications.json` is the source of truth for application configuration. Do not duplicate catalog data in PowerShell modules.
4. New installation types must be added to both dispatch switches in `InstallationRouter.ps1`.
5. New module files must be explicitly added to `$ModulePaths` in `Start.ps1` in dependency order.
6. Do not add dependencies, frameworks, abstractions, or configuration layers unless the requested change requires them.
7. Do not refactor unrelated code while implementing a focused change.
8. Do not leave TODOs, stubs, fake success results, debug code, or placeholder implementations and present the work as complete.
9. Do not bump `$AppVersion` unless explicitly instructed.
10. Preserve existing behavior and configuration compatibility unless the requested change explicitly requires a breaking change.

## Commands

Run normally:

```powershell
.\Start.ps1
```

Read-only startup validation:

```powershell
.\Start.ps1 -ValidateOnly
```

`-ValidateOnly`:

* does not request elevation;
* does not open the main menu;
* loads all configured modules;
* verifies required functions;
* verifies `Config\Applications.json`;
* exits with code `0` or `1`.

Run it after changes involving:

* module loading;
* module paths;
* public function names;
* public function signatures;
* startup dependencies.

There is no automated test framework or linter configuration.

`TESTING.md` is the manual acceptance checklist. Update the relevant section when behavior changes. Hardware-dependent tests may intentionally remain unchecked until tested on an authorized Windows device.

## Working rules

Before modifying code:

1. Read the relevant module.
2. Read neighboring modules that perform similar work.
3. Search for existing helpers before creating a new one.
4. Follow the repository's existing naming, logging, error handling, parameter, and return-value conventions.

Prefer the smallest change that correctly implements the request.

Do not create a new abstraction for logic used in only one place unless doing so materially improves correctness or maintainability.

Comments should explain non-obvious reasons, Windows behavior, compatibility requirements, or workarounds. Do not add comments that simply narrate the code.

Use only PowerShell commands, .NET APIs, functions, registry paths, exit codes, and application behavior that are known from the repository or can be verified. Do not invent commands, parameters, identifiers, or package behavior.

## Architecture

### Module loading

`Start.ps1` loads modules through the explicit `$ModulePaths` array.

Loading order represents dependencies.

Examples:

* `Core\Logging.ps1` must load before modules that log.
* `Installation\InstallationResult.ps1` must load before installer modules using result normalization.
* Individual installer modules must load before `InstallationRouter.ps1`.

There is no dynamic module discovery.

A `.ps1` file placed under `Modules\` is not available unless it is listed in `$ModulePaths`.

When adding a module:

1. Put it under the appropriate `Modules\<Category>\` directory.
2. Add it to `$ModulePaths` at the correct dependency position.
3. Add important new public entry points to `$RequiredFunctions` when startup validation should require them.

### Installation routing

Application installation is dispatched by:

`Modules\Installation\InstallationRouter.ps1`

Applications define an `InstallType`, such as:

* `Winget`
* `Exe`
* `MSI`
* `ZIP`
* `CrowdStrike`
* `OfficeISO`
* `Office2021IMG`
* `Teams`

The router contains two parallel dispatch paths:

`Test-ApplicationInstallerAvailable`

* read-only;
* verifies that the required installer/package is available and usable.

`Install-ApplicationByType`

* performs the actual installation;
* delegates to the appropriate installer module.

When adding an installation type:

1. Create its installer module under `Modules\Installation\`.
2. Implement an availability check.
3. Implement the installation function.
4. Add the type to `Test-ApplicationInstallerAvailable`.
5. Add the type to `Install-ApplicationByType`.
6. Add the module to `$ModulePaths`.
7. Run `.\Start.ps1 -ValidateOnly`.

Never make an availability check install, download, modify, or configure anything.

### Installation results

Result normalization is handled by:

`Modules\Installation\InstallationResult.ps1`

Primary functions:

* `New-ApplicationInstallationResult`
* `ConvertTo-ApplicationInstallationResult`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [I-Descry/it-deployment-tool](https://github.com/I-Descry/it-deployment-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
