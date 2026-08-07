---
trigger: always_on
description: Operating instructions for any AI coding agent (Claude Code, Codex, Cursor, etc.) working in this repository. Read this before touching any code. Product context, scope, and rationale live in `PRD.md` — read that first if you haven't.
---

# AGENTS.md

Operating instructions for any AI coding agent (Claude Code, Codex, Cursor, etc.) working in this repository. Read this before touching any code. Product context, scope, and rationale live in `PRD.md` — read that first if you haven't.

**Assumption baked into this file:** built as a **Pock plugin (PockKit)**, not a standalone DFRFoundation app (PRD §7.2, Option B). If this project pivots to Option A (standalone app), most sections below still apply — only §3 (Touch Bar presentation) and the dependency setup in §4 change. Flag it explicitly if you're working under Option A instead.

---

## 1. Project Summary

macOS Touch Bar widget showing real-time synced lyrics for whatever's playing in Spotify or Apple Music. Built as a Pock plugin. No backend, no user accounts, fully client-side. See `PRD.md` for full scope, non-goals, and success criteria.

## 2. Stack

- **Language:** Swift (match the Swift version PockKit's host app targets — check Pock's own `Package.swift`/Xcode project before assuming latest).
- **UI layer:** PockKit widget (`PKWidget` conformance), not raw `NSTouchBar`.
- **Now-playing detection:** `MediaRemote.framework` (private, no public header) via `dlopen`/`dlsym`.
- **Lyrics:** LRCLIB REST API (primary, synced LRC), Genius API (fallback, static text only).
- **Persistence:** local cache only — plist or a single SQLite file. No cloud sync, no backend.
- **No third-party dependency manager beyond Swift Package Manager.** Don't introduce CocoaPods/Carthage.

## 3. Touch Bar Presentation Rules

- All Touch Bar rendering goes through PockKit's widget lifecycle (`viewAppeared`, `viewDisappeared`, etc.) — don't bypass it with direct `DFRFoundation` calls. That's the whole point of building on Pock instead of from scratch.
- Widget must degrade gracefully if Pock itself isn't running / isn't installed correctly — never crash the host.
- Keep the widget's rendered width flexible; don't hardcode pixel widths assuming a specific Touch Bar model.

## 4. Setup & Build

- Requires Xcode (version TBD once PockKit's minimum is confirmed — check before scaffolding).
- Requires Pock installed locally for manual testing (widget won't render in isolation — it's hosted inside Pock's process).
- **Cannot be tested in the iOS/macOS Simulator** — Touch Bar hardware or Pock's dev harness only, if one exists. If Pock ships no simulator/preview mode, say so explicitly rather than assuming one.
- Build command / run command: fill in once the Xcode project is scaffolded — don't guess a `swift build` invocation that hasn't been verified against PockKit's actual project structure.

## 5. Code Style & Conventions

- Follow Apple's Swift API Design Guidelines (clear names over abbreviations, no Hungarian notation).
- **No force unwraps (`!`)** except with an inline comment explaining why it's provably safe at that point. Prefer `guard let` / `if let`.
- Use `async`/`await` for all network calls (LRCLIB, Genius) and file I/O — never block the main thread, especially inside the Touch Bar render path.
- Organize files by responsibility, not by type — one file per concern (see §8), not `Models.swift` / `Views.swift` dumping grounds.
- No placeholder/generic naming (`Manager`, `Helper`, `Utils` as a catch-all). Name things for what they specifically do: `NowPlayingWatcher`, `LRCSyncEngine`, `LyricsCache` — not `MusicManager`, `SyncHelper`.
- Comments explain *why*, not *what* — don't narrate obvious code.

## 6. Commit & PR Rules

- **Conventional commits**, small and atomic — one logical change per commit (matches existing project conventions).
  - Examples: `feat(now-playing): add MediaRemote bridging via dlsym`, `fix(sync): correct drift on seek events`, `chore(cache): switch plist to sqlite`.
- Don't bundle unrelated changes (e.g., a lyrics-matching fix + a UI tweak) into one commit.
- If a change deviates from what `PRD.md` specifies, note which PRD section it deviates from in the commit body or PR description — don't silently diverge from the spec.

## 7. Architecture Rules — Do / Don't

**Do:**
- Isolate all `MediaRemote` private-API access behind a single `NowPlayingWatcher` type. Nothing else in the codebase should `dlsym` directly.
- Cache every LRCLIB lookup to disk *before* attempting a second network call for the same track ID. Never refetch on every play of a previously-seen track.
- Treat `MediaRemote` elapsed-time updates as the single source of truth for sync position — don't run a separate local timer that can drift from actual playback.
- Handle "no lyrics found" and "lyrics found but not synced" as distinct, explicit states in the UI layer — not silently falling back to a blank widget.

**Don't:**
- Don't touch `DFRFoundation` or any raw private Touch Bar drawing — that's what Pock/PockKit exists to abstract away here.
- Don't store the Genius API key (or any credential) in source. Use a gitignored local config file or environment variable, and document the expected key name in `README.md` when that's created.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RidhaAF/lirik](https://github.com/RidhaAF/lirik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
