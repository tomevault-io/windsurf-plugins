---
trigger: always_on
description: handles during teardown so pipe readiness cannot create a CPU spin loop.
---

# CodexMeter Development Guide

## Project scope

CodexMeter is a native macOS menu bar app built with SwiftUI. It displays the
remaining Codex account quota without requiring the user to open Codex.

- Support macOS 13 or later.
- Keep the app menu-bar-only; do not add a normal window or Dock icon unless the
  product direction changes.
- Prefer Apple frameworks. Sparkle is the deliberate exception used for signed
  in-app updates without requiring an Apple Developer account.
- Make focused changes and preserve the existing architecture.

## Version baseline

- Version 1.0 (build 1) is the first accepted usable release baseline. The
  current release version is 1.6.2 (build 23).
- Keep source comments in English and reserve them for non-obvious architecture,
  protocol, state, permission, and calculation behavior. Do not narrate obvious
  Swift syntax line by line.
- Keep `README.md` synchronized with public features, build requirements,
  privacy behavior, known limitations, and the experimental App Server caveat.
- The accepted menu bar baseline uses a compact 18-point dual-ring indicator
  with visually prominent strokes: outer quota at 3 points and inner time at
  2 points. Use the brighter blue appearance choice for the default time color
  so the thinner inner ring remains legible.
- The accepted application icon uses a warm ivory background with a burgundy
  abstract code mark and a cream segmented meter with a coral active segment.
  Keep the source icon simple and legible at the 16-point macOS size.

## AGENTS.md maintenance

Treat this file as living project documentation. After completing every task,
review it and update it automatically when the implementation or project state
has changed.

- Mark completed roadmap items as done and add newly agreed follow-up work.
- Keep architecture, product rules, supported systems, privacy boundaries, and
  verification commands synchronized with the codebase.
- Remove or revise instructions that are no longer accurate.
- Record durable project decisions, not temporary debugging details or
  one-off conversational context.
- Keep edits concise. If a task does not change any documented fact or roadmap
  status, do not make a cosmetic `AGENTS.md` edit merely to touch the file.

## Architecture

- `CodexMeterApp.swift` owns the `MenuBarExtra` and shared usage service.
- `ContentView.swift` renders quota details and user actions.
  Its quota and time progress bars use the same configurable status and time
  colors as the menu bar indicator. The popover also presents compact weekly
  quota history and 30-day token activity as separate, divider-separated links
  to the full Usage History window. Render optional popover sections from the
  user's stored order. Bound the popover to the visible height of its current
  display: keep the header, Settings, and Quit controls fixed while only the
  dynamic status and content area scrolls when it exceeds the available space.
  Keep those regions as true vertical siblings and clip the middle scroll view
  so translucent fixed controls never reveal content rendered underneath them.
  On macOS 15+, hide popover scroll indicators on opening until the first user
  scroll, then restore automatic native visibility and fade-out for that opening;
  older systems retain native behavior.
  Reserve a 12-point trailing gutter for the popover scroller by extending only
  the scroll viewport into the outer padding; keep content aligned with the header.
- `MenuBarProgressView.swift` draws the selected ring, bar, percentage, and
  caption style into an original-color `NSImage`. Keep the status-item label
  free of nested dynamic layout containers. Omit time indicators when reset
  timing is missing.
  When fresh quota decreases within the same account and selected window, hold
  only changed percentage digits red for 0.5 seconds, then fade back over 2 seconds.
  Reset the comparison on context,
  style, stale-state, or preview changes; use a cancellable bounded fade and keep
  the percent sign unchanged. With Reduce Motion, restore the color without fading.
- `CodexUsageService.swift` launches the installed `codex app-server` process
  over stdio, communicates with it using newline-delimited JSON-RPC, and owns
  refresh/freshness state. Build a deterministic child-process `PATH` from the
  selected Codex executable directory plus common local package locations so
  npm-installed launchers can find Node without invoking a login shell. Always
  discover Codex installed under NVM version directories as well as the stable
  local, Homebrew, and npm-global locations. Prefer stable direct locations
  before selecting the newest executable NVM installation. Always drain both
  stdout and stderr, retain only a bounded in-memory diagnostic tail,
  detach file-handle callbacks at EOF or process termination, and close retained
  handles during teardown so pipe readiness cannot create a CPU spin loop.
  Bound stdout reads to 64 KiB and each JSON-RPC line to 1 MiB with main-queue
  backpressure; stop the child and mark the retained snapshot stale on overflow.
  Reset framing state at teardown and ignore output from replaced processes.
  Read stderr in at most 8 KiB chunks without an unbounded termination read.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raycalrui/CodexMeter](https://github.com/raycalrui/CodexMeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
