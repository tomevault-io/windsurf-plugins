---
trigger: always_on
description: Guidance for AI coding agents and LLMs working in the win-glaze-dots repository.
---

# AGENTS.md

Guidance for AI coding agents and LLMs working in the win-glaze-dots repository.

This file describes project-specific architecture, invariants, workflows, and validation expectations. It is intentionally separate from any user's general LLM personality or custom instructions.

## Core rule

Inspect the current repository before acting.

win-glaze-dots changes over time. Current code, tests, CI, Git state, and the exact requested branch/tag/release take precedence over remembered architecture, old conversations, old documentation, or assumptions based on similar projects.

If this file conflicts with the current implementation, verify the implementation and update this file as part of the relevant work when appropriate.

## Project identity

- win-glaze-dots is a Windows 10/11 dotfiles and configuration project maintained by dillacorn.
- `wgdot` is the native Windows maintenance system for installing, reviewing, updating, resetting, and testing managed configuration. Its primary runtime is compiled locally from inspectable repository C# source by `wgdot/bootstrap.cmd`, so normal operation does not depend on `.ps1` execution being allowed.
- The project also documents manual Windows/application setup that is intentionally not fully automated.
- The repository is a local open-source utility. Do not introduce telemetry, hosted-service dependencies, or data collection without an explicit project decision.

## Source-of-truth priority

When sources disagree, use this order unless the task explicitly targets historical behavior:

1. Exact user-requested target and current Git state.
2. Current implementation on that target.
3. Tests and CI that exercise the implementation.
4. Current release/tag metadata when release behavior is involved.
5. Current repository documentation.
6. Recent relevant Git history.
7. This `AGENTS.md` file.
8. Memory, prior conversations, or older architecture knowledge.

Never let memory override inspectable repository evidence.

## System map

```text
Windows 10/11
    |
    +--> wgdot/bootstrap.cmd
    |       |
    |       +--> downloads/uses inspectable wgdot-native.cs
    |       +--> compiles locally with Windows .NET Framework csc.exe
    |       v
    |   %LOCALAPPDATA%\wgdot\bin\wgdot.exe
    |       |
    |       +--> runtime self-refresh from main
    |       +--> explicit feature-branch refresh while maintainer-testing
    |       +--> stable release resolver
    |       +--> managed config planner/executor
    |       +--> required WinGet bootstrap + software reconciliation
    |       +--> reversible application startup / uninstall manager
    |       +--> adjacent backup manager
    |       +--> Git-testing mode
    |
    +--> wgdot/manifest.json
    |       |
    |       +--> managed components
    |       +--> Normal/Work defaults
    |       +--> GlazeWM profile mapping
    |       +--> WinGet package catalog
    |       +--> explicit migrations
    |
    +--> wgdot/wgdot.ps1 + MANUAL_POWERSHELL.md
            |
            +--> compatibility/reference implementation
            +--> paste-only PowerShell fallback for restricted environments

Managed source files
    |
    +--> UserProfile/.glzr/
    +--> UserProfile/.config/
    +--> UserProfile/AppData/
    +--> UserProfile/scripts/

Validation
    |
    +--> tests/test-wgdot.ps1
    +--> .github/workflows/validate-wgdot.yml
```

## Desktop runtime independence

WGDot is primarily a management/configuration tool. Narrow, compiled desktop-session helpers are allowed only where the desired behavior cannot be reproduced cleanly by GlazeWM, YASB, Windows, or the target application.

**Runtime-helper last-resort rule:** do not introduce, extend, or route ordinary desktop behavior through `wgdot.exe` or `wgdotw.exe` merely because WGDot can implement it. First exhaust the target application's own configuration, keybindings, commands, plugins, and APIs; then existing Windows, GlazeWM, YASB, Yazi, terminal, or other already-installed native mechanisms. Reuse an existing approved WGDot runtime helper only when it already owns the required primitive cleanly. Add or extend compiled WGDot runtime behavior only when those native/configuration paths cannot provide the required behavior reliably and the exception is narrow, documented, and covered by tests. If a task can be solved cleanly without WGDot runtime code, solving it through WGDot is an architectural regression.

- Managed GlazeWM and YASB configuration should remain broadly usable when copied manually without WGDot, but approved custom surfaces may degrade when the compiled helper is absent.
- Runtime ownership is hybrid and capability-driven. Ordinary actions already supported cleanly by GlazeWM, YASB, Windows, or the target application must stay native; WGDot runtime helpers are allowed only for custom behavior that genuinely requires code, cross-component state coordination, or low-level Windows APIs.
- GlazeWM owns GlazeWM keybindings and binding modes. YASB owns its native widgets and callbacks where those widgets satisfy the intended behavior. Installed applications should be launched directly when practical.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dillacorn/win-glaze-dots](https://github.com/dillacorn/win-glaze-dots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
