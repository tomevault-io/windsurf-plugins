---
trigger: always_on
description: A Quickshell overlay for Hyprland: one layer-shell surface pinned to the top of
---

# Working on this project

A Quickshell overlay for Hyprland: one layer-shell surface pinned to the top of
the screen. The QML is the product; `backend.sh` is a thin shell layer that
reports state and performs actions. There is no build step — Quickshell watches
the files and reloads on save.

This file is the contract for anyone (human or agent) making changes here.

---

## Before every commit

Three things, in order. None are optional.

### 1. Write the changelog entry

`CHANGELOG.md` (and its Turkish counterpart `CHANGELOG.tr.md`) is the single
source of truth for what shipped. A release is written as Markdown, newest
first:

```md
## [2026.08.17] - 2026-08-17

**A short sentence, not a version bump**

Two or three sentences: what changed and why it was worth doing.

### Added

- One bullet, one sentence: what the change does **for the person using it**,
  not which function was edited. "The chime picker ran past the right edge of
  the window — four of the eleven sounds could not be reached" beats "fixed
  ChimePicker layout".

### Fixed

- ...

### Shots

- ![what the image shows](screenshots/changelog/name.png "wide") — **Caption**
  — Sentence under it. The markdown title `"wide"` makes the card span the
  full grid; a shot without it shares a row with the next one.
```

- **Versions are dates** (`YYYY.MM.DD`), because releases here are "the day the
  work landed", not semver.
- If a release lands on a date that already has an entry, **edit that entry**
  rather than adding a second one for the same day.
- Groups are `### Added` / `Changed` / `Fixed` / `Removed` / `Shots`.
  A change group's `type` is its heading's lowercase name, so anything else is
  rejected by the script with a named error.
- `Shots` bullets reference files in `docs/` (they live in
  `docs/screenshots/changelog/`). The image's alt text is for screen readers
  and describes the image. `caption` is the short bold label. `text` is the
  sentence under it.
- The English and Turkish files must list the same versions; the script
  refuses to run while they diverge.

Then regenerate everything the entry feeds — the READMEs' latest-release
section and `docs/changelog.json` (what both site pages read):

```sh
python3 scripts/changelog.py        # rewrite the generated blocks
python3 scripts/changelog.py --check   # exit 1 if any block is stale
```

`docs/changelog.json` is generated output — never edit it (or release markup
in the HTML pages) by hand; edit `CHANGELOG.md` and let the script apply it.
`--notes 2026.08.17` prints one release as Markdown, for a GitHub release body.

### 2. Take the screenshots

**Always `tools/capture.sh`. Never a raw `grim`, never a hand-written
rectangle, never a full-screen capture as a fallback.**

```sh
tools/capture.sh media                        # the island → docs/screenshots/media.png
tools/capture.sh --target settings themes     # the settings window
tools/capture.sh --dir docs/screenshots/changelog --target settings my-change
```

Why this matters: both surfaces are **full-screen** layer-shell windows with
their content centred inside. `hyprctl layers` reports the screen, not the
pill — so a hand-picked crop is wrong the moment the island resizes, and a
wrong crop on a full-screen surface **captures whatever the user has open
behind it**. The script asks the running shell where its content actually is
(the `islandWidth` / `islandHeight` / `islandTopMargin` IPC properties on
`dynamicIsland`) and crops to exactly that.

Set the widget up over IPC first, then capture:

```sh
qs -p ~/.config/hypr/scripts/quickshell/Shell.qml ipc call dynamicIsland settingsSection appearance
tools/capture.sh --target settings --delay 1.5 themes
```

`make <target>-ss` does the same for the Makefile's widget shortcuts.

The `full` target exists for deliberate whole-desktop shots only. It will
capture every window on screen. Do not reach for it because a crop failed —
fix the crop.

### 3. Re-record the tour if the visuals changed

`docs/demo.mp4` is the video on the front page: every feature in order, on an
empty desktop.

```sh
tools/demo.sh --dry-run   # play the scenes, record nothing — check the script first
tools/demo.sh             # record → docs/demo.mp4
```

Only the newest video is committed; it replaces the previous one. Re-record
when the island's appearance changes in a way the current video no longer
shows. A copy edit or a backend fix does not need a new recording.

The scene list lives in `run_scenes()` in that script and is the video's
script — edit it there when a feature is added or removed.

---

## Verifying a change

Quickshell reloads on save and writes to a log. **A QML error does not surface
in the UI — it silently refuses to load the whole config, and the shell keeps
running the last good version.** If a change appears to do nothing, read the
log before assuming anything about the code:

```sh
tail -20 "$(ls -t /run/user/$UID/quickshell/by-id/*/log.log | head -1)"
```

`Configuration Loaded` means it took. `Failed to load configuration` names the
file and line.

`qmllint <file>.qml` catches syntax errors but **not** type errors like the two
below, so it is a first pass, not a verdict.

### Traps that have actually cost time here


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lunanoir21/quickshell-dynamic-island](https://github.com/lunanoir21/quickshell-dynamic-island) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
