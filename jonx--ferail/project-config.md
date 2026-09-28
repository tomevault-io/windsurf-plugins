---
trigger: always_on
description: This is the operating manual for AI or human edits in this repo.
---

# Claude Notes for Ferail

This is the operating manual for AI or human edits in this repo.

Read first:

1. [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
2. [TODO.md](TODO.md)
3. The crate-level docs and nearby code for the area you are changing.

## Active Target

`crates/ferail-gpui` is the active app. Run it with:

```sh
cargo run --bin ferail-gpui
```

Ferail has a Windows predecessor, `Ferail-Win32`: a separate, older codebase
in its own GitHub repo, `jonx/Ferail-win32` (local checkout: `../Ferail-win32`,
branch `master`). If you have it, inspect it before redesigning a feature the
user says worked better in the Windows version. Copy intent and lessons, not
Win32-specific shape.

**Name history:** this app was called *Feraille* until 2026-07-30, when it took
over the predecessor's name and the predecessor became *Ferail-Win32*. Anything
older than that commit, git history, branch names, external links, says
Feraille and means this app.

The GitHub repos were renamed the same day: this app is now `jonx/Ferail` (was
`jonx/Feraille`, which still redirects), and the predecessor is
`jonx/Ferail-win32` (was `jonx/Ferail`). So a pre-rename link to
`github.com/jonx/Ferail` meant the *Windows* app and now means this one. Never
create a new repo named `Feraille`: that silently kills the redirect. The
local directory moved `~/Source/Feraille` → `~/Source/Ferail`, and old Claude
Code transcripts were rewritten to match, so sessions predating the rename
still show the new path.

## Prime Directive

The UI must never stop. **This is non-negotiable**: it outranks feature
completeness, code brevity, and every convenience. Full doctrine, the
compliant pattern, and the enforcement machinery:
[Architecture § Prime Directive](docs/ARCHITECTURE.md#prime-directive).

Paint, render, hover, hit-test, scroll, resize, keyboard input, text input,
selection, and modal drawing are read-only and nonblocking. They must not:

- Read files or directories.
- Query Finder/AppKit/NSWorkspace for data.
- Query SQLite or other persistent stores.
- Generate previews or thumbnails.
- Sniff magic bytes.
- Resolve symlinks, aliases, cloud placeholders, or network locations.
- Build context menus by touching filesystem or shell state.
- Allocate heavily in row-by-row hot render paths.

The same applies to **action/click handlers and subscriptions**: they run
on the UI thread too. Any call that can touch a disk or the shell blocks
for *seconds* on a spun-down external drive or network mount, even ones
that look free on a local SSD: `Path::exists`, `metadata`, `canonicalize`,
`read_dir`, `notify`'s `Watcher::watch()` (canonicalizes internally),
NSWorkspace/LaunchServices lookups, xattr reads.

Expensive or possibly-blocking work is scheduled from semantic events, runs
on `cx.background_executor()` (or a worker thread), and reports back through
GPUI entity/update boundaries, guarded by a generation counter and a cancel
flag. If a result arrives after the user moved on, drop it.
`Shell::load_path_for_tab` is the canonical example: copy its shape.

Debug builds enforce this at runtime (`ferail_core::path_guard`): path
resolution during render panics, and known-blocking `ferail-fs-native`
entry points panic when called on the UI thread. **Never fix a guard panic
by removing the guard**: move the work off-thread. When you add a new
blocking entry point, add `assert_off_ui_thread` to it.

## Architecture Invariants

- `ferail-core` owns platform-neutral domain types and command identity.
- `ferail-fs-native` owns native filesystem work.
- `ferail-shell-mac` owns AppKit/Cocoa integrations and does not paint UI.
- `ferail-disk-usage` is pure model/layout logic.
- `ferail-gpui` owns GPUI views, actions, task scheduling, and shell state.
- UI rendering reads cached state; it does not resolve paths or touch I/O.

## Porting Rule From Ferail-Win32

Translate by intent:

- Win32 shell context menus become macOS menus/Finder actions/services.
- Win32 COM drag/drop becomes NSPasteboard and AppKit dragging.
- WSL features are not macOS v1 features unless they map to network or remote
  volumes.
- Direct2D/GDI details are renderer lessons only.
- SQLite metadata, Ant Trail, disk usage, magic, previews, and duplicate
  finding remain valuable but must use Mac-safe workers and identity.

## Icons

Every icon the app draws is cataloged in
[docs/features/ICONS.md](docs/features/ICONS.md): its source (macOS
NSWorkspace / local Lucide-derived bundle / upstream `gpui-component-assets`),
attribution, and the exact command/surface that uses it.

**When you add, move, or repurpose any icon, update [docs/features/ICONS.md](docs/features/ICONS.md) in
the same change.** Specifically:

- Do not reuse an existing command's glyph for a different command: the
  command→icon mapping is meant to stay ~1:1 so weak/overloaded icons are easy
  to spot. Draw a distinct glyph instead.
- Stay **platform-neutral** when possible: one icon set serves macOS, Windows,
  and Linux, so avoid OS-specific metaphors (⌘/`command`, the Windows logo,
  Finder chrome) for generic commands: use a universal glyph (`keyboard`, not
  ⌘, for shortcuts). Platform-flavored glyphs are only OK on `#[cfg]`-gated
  controls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonx/Ferail](https://github.com/jonx/Ferail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
