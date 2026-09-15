---
trigger: always_on
description: Luminous is not trying to be a best-of-all-worlds app. It covers the tagging, library, and
---

# Luminous Music Player

## Product Scope

Luminous is not trying to be a best-of-all-worlds app. It covers the tagging, library, and
playback needs of most users well, but it deliberately does not chase feature parity with
specialist tools. For example: if a user needs heavy-duty batch tagging (AcoustID-driven
bulk retagging, complex fingerprint-based lookups, tag scripting), the answer is to direct
them to a dedicated tool like MusicBrainz Picard rather than building that depth into
Luminous. When scoping a feature request, prefer "good enough, well-integrated" over
matching a specialist tool's full depth — and when a request is clearly outside that bar,
recommend the existing dedicated tool instead of expanding scope.

This is not a blanket "no tagging features" rule — the line is *canonical lookup vs. personal
curation*. Picard's job is resolving a file against MusicBrainz's canonical database: correct
artist/album/track identity, official release metadata, AcoustID fingerprint matching, bulk
retagging a whole library against that source of truth. That's squarely out of scope for
Luminous. But organizing music *the user's own way* — layered on top of whatever canonical tags
already exist — is core to what a personal library manager should do, and belongs in Luminous
natively. Example: MusicBrainz has no opinion on whether a track is Folk Metal, Progressive
Metal, or Symphonic Metal — genre is often too broad and any subgenre taxonomy is inherently
personal/subjective. A user-defined tag system for exactly this (see #224) isn't scope creep
into Picard's territory; it's a different, complementary axis Picard was never meant to cover.

## Branching & PR Rules

- NEVER commit or push directly to `main`. All changes ship via PR, even release version bumps and docs-only edits.
- Merging is allowed once — and only once — every check on the PR has actually finished, not just the required ones GitHub's branch-protection summary cares about. Do not treat the green "Able to merge this pull request" banner or the API's `mergeable: MERGEABLE` field as that signal on their own — both go green as soon as required checks pass while non-required checks (e.g. Backend Tests, CodeQL) can still be `in_progress`. Confirm via `gh pr checks <pr> --watch` (blocks until every check concludes) or `gh pr checks <pr>` showing zero `pending`/`in_progress` rows, then run `gh pr merge <pr>`. Tell the user once it's merged.
- PR base branch depends on the target issue's milestone — see Branching Model below (2.0-milestone work targets `next`; everything else targets `main`).
- Before creating a branch, confirm the base: `git fetch origin && git switch -c <branch> origin/<base>`.

## Destructive Operations

- NEVER use `rm -rf`, `git clean`, or unconditional deletes on directories that may contain untracked files. Run `git status --porcelain --ignored` first and list what would be lost.
- `git mv` only works on tracked files — for untracked assets use plain `mv` and verify afterward.
- Prefer moving to a temp dir over deleting; let the user do the final delete.

## Worktrees

- All feature work happens in a dedicated worktree under the standard worktree root (`.claude/worktrees/` for Claude, `.worktrees/<name>/` for other assistants — see Version Control below). NEVER edit files in the main checkout while a worktree exists for that branch.
- Before any Edit/Write, run `pwd` and confirm the path is the intended worktree.
- Never run `bun install` or `cargo build` in the main checkout during worktree work — it dirties `bun.lock`/lockfiles.

## Tech Stack

- **Frontend**: SvelteKit + Svelte 5 (Runes) + TypeScript + Tailwind CSS v4
- **Backend**: Rust (edition 2021) + Tauri v2
- **Database**: SQLite via rusqlite + r2d2
- **Audio**: Symphonia (decode) + CPAL (output)

## Project Structure

- Rust source lives in `src-tauri/src/`; Svelte source in `src/`
- Shared TypeScript types in `src/lib/types/`; Svelte 5 stores (Runes) in `src/lib/stores/`
- No dedicated IPC wrapper layer — components and stores call Tauri commands directly via `invoke()` from `@tauri-apps/api/core`

**Key frontend stores** (`src/lib/stores/*.svelte.ts`):

- `player.svelte.ts` — playback state, queue, volume, shuffle/repeat
- `collection.svelte.ts` — library metadata, folder list
- `playlists.svelte.ts` — playlist CRUD, undo/redo
- `theme.svelte.ts` — color schemes, artwork extraction

**Core Rust modules** (`src-tauri/src/`):

- `audio.rs` — Symphonia decode + CPAL output pipeline with gapless playback
- `player.rs` — playback state machine (shuffle, repeat, queue control)
- `collection.rs` — library scanner (incremental; respects file mod times) + file watcher
- `db.rs` — SQLite schema, connection pool (r2d2), migrations
- `playlist.rs` — playlist CRUD + undo/redo command stack
- `equalizer.rs` — biquad DSP filters (10-band graphic, 20-band parametric)
- `analyzer.rs` — real-time FFT spectrum processing
- `lyrics.rs` — LRCLIB + Lyrics.ovh clients
- `covermanager.rs` — embedded art extraction + iTunes API fallback
- `tageditor.rs` — lofty tag reader/writer + AcoustID fingerprinting
- `commands/` — all `#[tauri::command]` IPC handlers (registry in `commands/mod.rs`)

## Package Manager


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [esoltys/luminous](https://github.com/esoltys/luminous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
