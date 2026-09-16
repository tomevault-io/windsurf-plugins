---
trigger: always_on
description: This is a centralized dotfiles repository that automates all system configurations across multiple machines and operating systems. It uses a PowerShell module architecture with a single `Configuration.psd1` as the central hub for Windows. Linux integrations is yet to be made.
---

# Project Guidelines

## Repository Overview

This is a centralized dotfiles repository that automates all system configurations across multiple machines and operating systems. It uses a PowerShell module architecture with a single `Configuration.psd1` as the central hub for Windows. Linux integrations is yet to be made.

**Before working on any task**, read the relevant context document:

- General repository work → `AI/Context/REPOSITORY_CONTEXT.md`
- Windows/PowerShell work → `AI/Context/WINDOWS_CONTEXT.md`

## Code Style

- **PowerShell conventions**: Read `AI/Instructions/PowerShellConventions.md` before writing or modifying PowerShell code
- **Output formatting**: Read `AI/Instructions/OutputFormatting.md` for function signatures, config entries, and documentation format
- **Configuration changes**: Read `AI/Instructions/ConfigurationPatterns.md` before modifying `Configuration.psd1`
- **Documentation updates**: Read `AI/Instructions/DocumentationStyle.md` before updating user-facing docs
- **Never use em-dashes**: always use a plain hyphen `-` instead of the em-dash character (U+2014), in all prose, comments, documentation, log messages, string literals, and commit messages. The em-dash must not appear in any managed file. See `AI/Instructions/OutputFormatting.md`. (The sole exception is the browser-title regex classes in `Configuration.psd1` `BrowserGroups` and `Window/Layouts/*`.)

## Documentation Updates (Required)

**Docs are always kept up to date.** ANY change - not only to functions - that adds, removes, or alters observable behavior, structure, configuration, data, workflow, or usage MUST update the corresponding documentation in the SAME change. Docs are never a follow-up task: a change that touches a documented surface without updating its page is incomplete. When in doubt about whether a change is user-facing, assume it is and update the docs. The exported-function case below is the most common instance of this rule, not the whole of it - see the trigger lists for the non-function surfaces (configuration, data files, bootstrap flow, AI system, pages) that are equally covered.

Since the 2026-06 documentation refactor, the docsify docs under `docs/` are the **SINGLE SOURCE OF TRUTH**. `README.md` at the repo root is now a **minimal pointer** (logo + intro + demo-video placeholder + link to the docs) and carries NO function reference. NEVER add `#### [Name]` function entries, a function Table of Contents, or per-function content to README, and do NOT update README when a function changes.

Every change that adds, renames, removes, or modifies the **behavior, parameters, or signature** of an exported PowerShell function MUST be reflected in the docs in the same change:

- `docs/modules/<Module>.md` - the function reference. ONE man-style entry per function = a `## [FunctionName](<github-source-url>)` heading followed IMMEDIATELY by a contiguous `- **Key:** value` bullet block (Description first, then Parameters / Usage / Alias as applicable; omit a bullet when not applicable). Optional human-only prose, parameter tables, examples, and a `**See also:**` line may follow after a blank line. Entries are alphabetical within the page.
- The module `.psd1` `FunctionsToExport` - keep it in sync when adding/renaming/removing a function.
- `docs/configuration/guides/<module>/<Function-Name>.md` - the function's configuration guide. EVERY exported function has exactly one, named after it. Use the FULL template when the function reads configuration and the STUB template when it does not (see `AI/Instructions/DocumentationStyle.md` for both templates and the mandatory sentinel sentence). Add the function to the module's `docs/configuration/guides/<module>/README.md` index in the same change.
- `docs/docs_overview.md` - internal maintenance reference (process + per-module index); update it when adding/renaming/removing a function.
- `docs/_sidebar.md` - only when adding/removing a page (not for per-function changes). Per-function guides are NOT sidebar entries - the 11 module index pages are.

`List-Functions` (Helper module) PARSES the module pages. After adding/renaming/removing a function, update its entry alphabetically in `modules/<Module>.md`, update the module `.psd1` `FunctionsToExport`, and run `List-Functions -ListDiscrepancies` (must report none).

**Fork-only functions (the Custom area):** functions not (yet) shipped by WinuX live under `Windows/PowerShell/Modules/Custom/<Module>/Functions/` and are documented with the SAME man-style entry format in `docs/custom/<module>.md` (the heading link points at the fork's source URL). Their name goes in `Custom.psd1`'s `FunctionsToExport` (the fork-owned manifest, not an engine module's) - which is what makes them autoload - and `List-Functions -ListDiscrepancies` checks them against their `docs/custom/` pages. Graduation into WinuX follows `Windows/PowerShell/Modules/Custom/README.md`.

Use **genericized example values** in docs (placeholders like MyProject, MyRepo, GroupName, Work-PC) - no real personal/work identifiers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IvanPavlak/WinuX](https://github.com/IvanPavlak/WinuX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
