---
trigger: always_on
description: A macOS menu-bar app that files older downloads into folders named for the day
---

# Tideline

A macOS menu-bar app that files older downloads into folders named for the day
they arrived. Native Swift, SwiftPM, no dependencies — the release zip is about
1.7 MB, and 5.8 MB unpacked because the bundle is universal. That is a feature,
not an accident.

A Windows build lives in `windows/`: the same filing rules rewritten in Rust,
behind a Tauri v2 shell. It builds and its tests pass. The **filing engine is
complete** — grace window, daily and monthly folders, type folders, the skip
list, settling and collision rules. Clearing, duplicates, the big-file review,
catch-up, folder watching and the updater are **not ported yet**;
`windows/README.md` ranks what is left.

The two apps share no code, only a contract. Nothing in `windows/` should ever
be a reason to change `app/`, or the reverse — see `docs/behaviour.md`.

## Layout

| Path | |
| --- | --- |
| `app/Sources/Tideline/` | The app. `Organizer` files, `Cleaner` clears, `Deduper` collapses copies, `Weigher` finds big files, `Regrouper` catches up, `Controller` schedules |
| `app/Sources/Tideline/Views/` | SwiftUI window, one file per tab or sheet |
| `app/Package.swift` | One executable target, macOS 14+, zero dependencies |
| `app/build.sh` | Build, bundle, icon, sign, notarize, staple, zip |
| `app/Resources/Info.plist` | Bundle template; `__VERSION__` / `__BUILD__` substituted at build time |
| `windows/src-tauri/crates/tideline-core/` | The Rust filing engine. `organizer` holds the rules and is pure; `sweep` does the I/O |
| `windows/src-tauri/src/` | The Tauri shell — tray, schedule, commands, settings on disk |
| `windows/src/` | The window: plain HTML, CSS and JS, no framework and no build step |
| `docs/behaviour.md` | The filing contract both platforms implement |
| `fixtures/sweep-cases.json` | The contract as runnable cases. Add a rule here first |
| `homebrew/` | The Homebrew cask and the script that stamps it into `estruyf/homebrew-tap` |
| `docs/building.md`, `docs/signing.md`, `docs/releasing.md`, `docs/homebrew.md` | Build, distribution, release and cask notes |
| `windows/README.md` | Building on Windows, what is ported, what is not |
| `app/.build/`, `app/dist/`, `windows/**/target/`, `old-scripts/` | Ignored by git; never edit or commit |

## Commands

```bash
npm run build            # universal binary into app/dist/
npm run build:install    # build, copy to /Applications, launch it
npm run logs             # tail ~/Library/Logs/Tideline.log
npm run quit             # quit the running app
npm run clean            # throw away build artefacts
npm version patch        # bump, rebuild, commit and tag

npm run win:test         # the Rust workspace: 47 tests
npm run win:check        # type-check the engine against the Windows target
npm run win:dev          # run the Tauri app (works on a Mac too, see below)
npm run win:build        # NSIS installer — Windows only
```

`swift run` is not a useful way to test a change: it starts the executable
without a bundle, so there is no bundle identity and no scoped Downloads
permission. Build the app and run that instead.

## Verifying a change

There is no Swift test target. Confirm behaviour by running the real bundle:

1. `npm run build:install`
2. Switch on **Preview mode** under **Filing** — sweeps then report what they
   would do without touching a file.
3. `npm run logs` and watch a sweep decide.

The Rust side does have tests — `npm run win:test` runs all 45. The rules are
pure functions taking a list of entries and a moment, so a rule change is
provable without a filesystem; only `sweep.rs` needs a real folder.

The first-run flow — the permission prompt and the question that gates
filing — only happens once per machine. `docs/building.md` has the commands
that put the app back to knowing nothing about you.

`npm run win:dev` works on a Mac: it builds the same Rust and the same front end
into a macOS window, which is enough to iterate on the layout. The `cfg(windows)`
paths — the Recycle Bin, `FILE_ATTRIBUTE_HIDDEN` — are inert there and still
need real hardware. `npm run win:check` compiles them without running them.

Cross-*building* the installer from macOS does not work; CI builds it on
`windows-latest`.

## Rules that matter

- **`docs/behaviour.md` is the contract, and `fixtures/sweep-cases.json` is the
  executable half of it.** Any change to what moves, when, or where it lands is
  a change to the document and a case in the fixtures first, then to both
  implementations. They exist so the Swift and Rust engines cannot quietly
  drift. Only the Rust side runs the fixtures today; the Swift planner reads the
  filesystem directly and needs the same pure/IO split before it can.
- **Nothing is ever deleted.** Removals go to the Trash on macOS
  (`FileManager.trashItem`) and the Recycle Bin on Windows (`sweep::recycle`),
  never `unlink`. A setting someone regrets should be a drag back out, not a
  restore from backup.
- **Nothing is ever overwritten.** A name collision counts up: `report.pdf`,
  `report-1.pdf`, `report-2.pdf`.
- **Preview mode (`Settings.dryRun`) binds every destructive feature.** Filing,
  clearing, duplicate collapsing, large-file review and catch-up all have to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [estruyf/tideline](https://github.com/estruyf/tideline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
