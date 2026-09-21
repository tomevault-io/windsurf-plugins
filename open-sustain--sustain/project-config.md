---
trigger: always_on
description: `Sustain` is a Linux-only, Debian-first music library/player for a single
---

# Sustain Project Basis

`Sustain` is a Linux-only, Debian-first music library/player for a single
primary user. The product target is an iTunes-like desktop music manager,
roughly aligned with the dense, predictable library workflow of iTunes 11,
circa 2012. Sustain is its own product — not a clone of any prior Linux
player, not a continuation of any other project's UX.

NEVER USE CLAUDE MEMORY FEATURE. Your memory are existing projects .md files, github issues and comments. And for critical, always in the back of your head: AGENTS.md/CLAUDE.md (symlinks of each other).

Project and application naming:

- Product/application name: `Sustain`
- Rust binary name: `sustain`
- Rust crate/package prefix: `sustain-*` / `sustain_*`
- Linux application id: `io.github.open_sustain.sustain`

## Approved Stack

- Language: Rust
- UI toolkit: GTK4
- Playback backend: GStreamer
- Database: SQLite
- Metadata reading/writing: start with `lofty`; use TagLib bindings only if
  needed for real compatibility gaps
- Desktop integration: D-Bus/MPRIS via `zbus`
- Target platform: Linux on Debian, Wayland-first
- Packaging: Debian package as the primary distribution format
- License: GPL-3.0-or-later (declared in `[workspace.package]`); do not relicense or add dependencies with incompatible licenses
- Every new `.rs` file starts with `// SPDX-License-Identifier: GPL-3.0-or-later` then `// Copyright (C) 2026 AnnoyingTechnology`

## Product Direction

The application should own its library model, playlists, ratings, play counts,
search behavior, and playback state. The codebase should be structured so these
core concepts are not coupled tightly to GTK widgets.

The core user experience is the main table/list view. Advanced views are not
part of the initial product shape. An album-oriented view is a later nice-to-have,
not a core requirement.

Primary UI modes:

- Songs: default full-library mode, full-width table, no sidebar
- Albums: full-width album-cover grid, no sidebar
- Playlists: playlist sidebar left of the lower content area

Prioritize:

- clean code architecture with precise naming
- focused tests for domain rules, persistence, import behavior, search, and playback state
- dense, keyboard-friendly desktop UI
- compact window chrome; avoid an empty forced titlebar that wastes vertical space
- integrated top bar is intentionally taller than default GTK chrome, with controls scaled up
- playlist sidebar stays below the media top bar, left of the main content
- mode switcher belongs to the main content column, not to the full window root
- predictable iTunes-like library and playlist behavior
- first-class native GTK light and dark appearance; do not add an Sustain theme picker
- every CSS color decision must work in both light and dark themes and respect the system accent color (prefer `alpha(@theme_fg_color, X)` and `@theme_selected_bg_color` over hard-coded colors)
- fast search/filtering over a large local music library
- settings/preferences
- durable SQLite schema with explicit migrations
- clean media-key and MPRIS integration
- boring, maintainable Linux-native dependencies

Core feature set:

- main music library interface
- playlists
- metadata display and editing
- ratings
- listening statistics, such as play count and last played
- search and filtering
- settings/preferences
- playback controls and state

Persistence and tag mirroring:

- SQLite is the source of truth for every value that exists in the
  library: ratings, play count, skip count, last-played, last-skipped,
  and every editable metadata field. Once a track has been imported,
  file tags are NOT consulted to override SQLite values for that
  track, even on rescan. The library wins.
- File tags are read only as INITIAL VALUES when a track is first
  added to the library (e.g. its first scan, before any SQLite row
  exists). After that point, only the SQLite value is authoritative.
- For metadata that the user edits in Sustain (rating, title, artist,
  genre, etc.), the new value IS mirrored back to the file's tags as
  a courtesy to other applications. This applies to MP3/ID3, Ogg,
  MP4/M4A, and FLAC where a standard tag exists for the field. Do
  NOT invent custom tags to bridge format gaps.
- Listening statistics — play count, skip count, last-played,
  last-skipped — are NEVER written to file tags. iTunes never did
  either; they live exclusively in the library database. This also
  avoids touching audio files on every play, which would needlessly
  rewrite tags during playback.
- Sustain writes that touch shared tag frames must not clobber data
  belonging to other tools. For example, writing a rating into POPM
  must preserve any existing `play_counter` in the same frame, even
  though Sustain itself does not consume that counter.
- Artwork is cached separately and is not subject to this policy.

## Performance

Performance is a first-class feature, not a polish step. Target pristine
responsiveness and fluidity on a 10,000-track library: instant search,
smooth scrolling, snappy view switches, fast cold start. Code that ships
visibly sluggish behavior at that scale is incomplete, regardless of
correctness.

The maintainer develops on a Ryzen AI Max+ 395 (laptop) and a Ryzen

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-sustain/sustain](https://github.com/open-sustain/sustain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
