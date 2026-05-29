---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dash Chat is an end-to-end encrypted messenger built with Svelte 5 (frontend) and Rust/Tauri (backend), using p2panda for peer-to-peer communication. The application works both with and without internet connectivity.

**Current Status**: Pre-alpha, being rebuilt on top of p2panda.

## Signal UX Reference

Dash Chat aims to match Signal's UX as closely as possible. A private repository of Signal screenshots (Android + iOS) is available at `dash-chat/signal-screenshots`.

**Setup (run once per session if needed):**
```bash
# Clone if not already present (gitignored)
[ -d signal-reference ] || gh repo clone dash-chat/signal-screenshots signal-reference
```

**When building or modifying UI, you MUST:**
1. Read `signal-reference/manifest.json` to find the relevant Signal screenshots for the Dash Chat route you're working on.
2. Read the corresponding screenshots (both `android/` and `ios/` when available) to understand Signal's layout, spacing, typography, colors, and interaction patterns.
3. Model your implementation after Signal's UX. Match the overall feel, not pixel-perfect details — adapt for Konsta UI components and our existing patterns.
4. When verifying your UI changes, compare your screenshots against the Signal reference.

**Directory structure:**
```
signal-reference/
├── manifest.json          # Maps Signal sections → Dash Chat routes
├── android/               # Android (Material) screenshots
│   ├── home/              # Chat list, search, overflow menu
│   ├── create-account/    # Onboarding flow
│   ├── direct-chat/       # 1:1 chat view + chat-settings/
│   ├── group-chat/        # Group chat view
│   ├── message-types/     # Image/voice/reactions/context menu
│   ├── new-message/       # Contact picker + new-group/
│   └── settings/          # All settings sub-pages
└── ios/                   # iOS screenshots (same structure)
└── desktop/                   # Desktop screenshots (same structure)
```

Screenshots are named descriptively with sequence prefixes (e.g., `01-chat-list-empty.png`, `02-overflow-menu-open.png`). Browse the directory listing to find what you need.

## General Coding Style

Please read this coding style carefully and take it into account when planning or coding:

- Try to remain as simple as possible with your implementations.
- Try to reuse types and functions across the project rather than reimplement them.
- Don't use `any` or `unknown` typescript types. Instead, try to understand the actual typescript types and use them to infer the appropriate data structures and algorithms to use.
- Prefer Tailwind CSS utility classes over custom CSS styles whenever possible. Use inline `class` attributes with Tailwind classes instead of adding styles to `<style>` blocks.
- **Write very few comments.** Default to none. The only two acceptable reasons to add a comment are:
  1. Documenting what a function does (a doc-comment on the function signature). Skip these for self-explanatory helpers whose name and signature already say everything.
  2. Explaining *why* a non-obvious piece of code is there — a hidden constraint, a workaround for a specific bug, a subtle invariant that would surprise a reader. Keep these to one or two lines, no more.
  Anti-patterns — do NOT write these:
  - **What-comments**: restating what the code does when well-named identifiers already say it.
  - **Derivation comments**: walking the reader through the reasoning that produced the formula or condition right below ("In LTR X is on the right; in RTL it's on the left; therefore we flip Y..."). The formula is the artifact. The derivation belongs in the commit message or your head, not in the source. A reader having to think for a few seconds to recompute the reasoning is the normal cost of reading code, not a signal to comment.
  - **Narrative-of-change comments**: notes about the edit you just made ("added X to support Y", "renamed for clarity", "moved from foo.ts"). That's PR-description territory.
  - **Stale TODOs**: things that belong in the issue tracker.
  Self-test before keeping a comment: "If I delete this, would a reader of the surrounding code be genuinely confused about *why* something is the way it is, or just have to read the code?" If only the latter, delete it. When in doubt, delete it.
- **Avoid deep nesting.** If a function is going to exceed two levels of nesting (e.g. an `if` inside a `for` inside another block, or three nested callbacks), extract the inner work into clearly named utility functions that each do one specific task and call them from the higher-level function. Flat code with named helpers is easier to read, test, and re-arrange than a single deeply nested function.
- **STRICT: Use logical `start`/`end` CSS properties, not directional `left`/`right`.** The app is fully RTL-aware (Farsi is a supported locale) and directional left/right properties break in RTL.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dash-chat/dash-chat](https://github.com/dash-chat/dash-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
