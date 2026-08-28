---
trigger: always_on
description: Flux Launcher is a native Windows 11 launcher written in Rust. The GUI must use the vendored `windui` framework exclusively. Do not introduce WebView, Electron, egui, iced, Tauri, or another GUI framework.
---

# Flux Launcher Agent Guidelines

## Project scope

Flux Launcher is a native Windows 11 launcher written in Rust. The GUI must use the vendored `windui` framework exclusively. Do not introduce WebView, Electron, egui, iced, Tauri, or another GUI framework.

The launcher is a tray-resident application. It must use the real Windows DWM Acrylic or Mica system backdrop through the existing Win32 and DirectComposition path. Keep the entire launcher surface transparent so the system material fills the complete window. Do not replace the backdrop with fake gradients, opaque cards, tinting gradients, or WCA AccentPolicy as the primary solution.

## Repository language and ownership

All Flux-owned source comments, documentation, release notes, and user-facing application strings must be written in English. Do not add Chinese symbols or Chinese comments to Flux-owned code. Conversation with the project owner may use Russian.

Use concise English commit messages. Do not commit Manus-internal scratch notes, generated planning files, temporary screenshots, or unrelated artifacts. User-facing documentation may be committed when it is part of the requested product change.

## Required working process

Work in a deliberate, manual-first loop. Before making a change, inspect the relevant source, workflow, release history, and existing tests. For a multi-step change, create a concrete plan with investigation, implementation, validation, and delivery phases. Do not guess when a repository file, CI log, installer log, or Windows smoke result can answer the question.

Make the smallest complete change that solves the observed problem. Preserve existing architecture and avoid broad rewrites. Keep the user informed at meaningful checkpoints, especially when a CI runner is waiting, a workflow fails, or an external review is pending. Never claim a fix is ready until the relevant local and Windows validation has passed.

Every completed product fix must be followed by one manually prepared beta release. Beta releases must use `prerelease: true` and must not include `(beta)` in the release name. The release body must be written or manually corrected before delivery and must state what changed, what was tested, known limitations, download links, and what the project owner should verify. Do not create empty, duplicate, or unexplained releases. Do not create releases automatically on every push, schedule, or internal commit. A release workflow may be started manually to build Windows artifacts, but the release version, channel, and detailed notes must be intentionally selected and checked by the agent.

Stable releases require explicit promotion or user instruction. Beta releases must not be submitted to WinGet. WinGet automation must never create a GitHub release; it may only prepare or submit a stable manifest PR when the stable-only policy is explicitly enabled. Do not request or create `WINGET_GITHUB_TOKEN` or signing secrets for a manual WinGet PR unless the user explicitly asks to enable that automation.

## Windows lifecycle and startup behavior

The default activation hotkey is Alt+Space and must remain configurable. Repeated activation must toggle visibility. Search receives keyboard focus immediately when shown. Clear-query-on-activation is enabled by default. Game Mode and fullscreen hotkey protection are enabled by default. Application results must rank before ordinary files and folders. Keyboard navigation must support Up, Down, Home, End, Enter, Right, Left, and Escape according to the existing Flow-style behavior.

The Windows startup checkbox in the installer is enabled by default but must remain user-selectable. The installer must also show a post-install **Launch Flux Launcher now** option selected by default. These are separate choices: launching immediately after installation must not be confused with enabling startup on future Windows sign-ins.

The startup registry command must use `--startup`. Startup mode must call windui `start_hidden()` so Windows sign-in creates only a running tray process and does not display the search window. The global hotkey or the tray's Show launcher action must be required to show the search window. Installer smoke must verify both default startup and the opt-out path.

The Start Menu shortcut must target the installed executable and explicitly reference the Flux Launcher `.ico` resource. The installer must include the multi-resolution icon resource in the installed directory. Installer smoke must verify the shortcut target and icon metadata, not only that the shortcut file exists.

## Windows Acrylic and lifecycle invariants

The Win32 lifecycle is sensitive to ordering. `ShowWindow` must establish visibility before application show callbacks mutate layout state. After a visible activation, the first transparent D2D frame must be invalidated and presented before relying on user input or a query change.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m1nuzz/flux-launcher](https://github.com/m1nuzz/flux-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
