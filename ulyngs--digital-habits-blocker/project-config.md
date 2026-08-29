---
trigger: always_on
description: Guidance for coding agents (Claude Code, etc.) working in this repository.
---

# AGENTS.md

Guidance for coding agents (Claude Code, etc.) working in this repository.

This file is loaded on every task, so it holds only what you would get *wrong*
by exploring the repo: invariants, cross-file contracts, and the reasoning
behind decisions that look like bugs from inside a single file. If
`package.json`, a workflow file, or the code already tells you, it is not
repeated here — keep it that way when you edit this file.

## What this is

Digital Habits: Blocker is a cross-platform website/app blocker built as a
**single Tauri v2 app** (Rust backend + HTML/JS/CSS frontend) targeting macOS
11+, Windows 10+, iOS 16+, and Android 8+ (API 26+). One frontend codebase
(`src/`) drives all four platforms; enforcement differs completely per platform.
There is **no** privileged helper daemon and **no** hosts-file writing — the app
itself is the enforcement engine (v3 architecture).

**Blast radius.** Because one `src/` ships to four platforms with four unrelated
enforcement backends, the usual way a change goes wrong here is not a broken
build — it is working correctly on the platform you ran and silently changing
behavior on the other three. Before you call a change to `src/` done, say what
it does on the platforms you did not run, and say plainly which ones you could
not verify. "Tier 1 is green" is not the same claim as "Android still blocks."

## The product principle

The user of this app is adversarial to their own future self. They asked to be
blocked, and in a weak moment they will look for a way out — that is the entire
problem the product exists to solve. Friction is the feature, not a rough edge
to smooth.

Several parts of the codebase look like bugs until you apply this:

- The tray icon has no menu and there is no quit gesture anywhere. Both
  `RunEvent::ExitRequested` and, on macOS, an `applicationShouldTerminate:`
  hook unconditionally turn every exit route into a hide-window. A blocker the
  user can quit is a blocker the user can bypass. Do not add an escape hatch to
  either guard without a deliberate decision — `commands/uninstall.rs` calls
  `std::process::exit(0)` specifically to bypass both, and in-app uninstall
  (macOS) or the OS uninstaller (Windows) is the only intended way out.
- Override challenges (`src/override-challenge.js`) make unblocking
  deliberately effortful. Making them cheaper or skippable is a product
  regression, not an ergonomics win.
- The compliance enforcer force-quits non-compliant browsers, and is
  nonetheless **opt-in and default off** (`settings.enforcementEnabled`).
  Escalation is a choice the user makes, not one we make for them.

**The adversary is a distracted user, not a technical one.** Our users are
non-technical: the bypass we design against is the one reachable in a weak
moment through the app's own UI, not one that requires a devtools console, a
debugger, or a rebuild. That is why `window.__REDDBLOCK_INTERNALS__`
(`src/dev-internals.js`) ships unguarded in production even though it exposes
`saveData`, `acceptEula` and friends — a deliberate accepted cost, not an
oversight. Keep the bar there: adding to that object is fine, but anything that
lowers friction in a path a user can reach *without* opening a console is a
product regression. The `e2e-webdriver` / `system-test` hard stop in `lib.rs` is
the separate, stricter case — a network-reachable automation endpoint is a
bypass anyone could be walked through remotely, so it must never compile into a
release.

**Decide which way failures fall.** When enforcement cannot determine state — a
URL will not parse, a browser will not answer, a query fails — the code must
pick between blocking something it should not and allowing something it should
have blocked. Neither is automatically right, but the choice must be explicit,
and it must be written down where the next reader will see it. This codebase's
worst bugs are the ones that quietly chose "allow" (see the Samsung Internet
case below).

## Where the real documentation is

| Read this | Before you |
| --- | --- |
| [architecture.md](architecture.md) | Touch any enforcement code. **Part I = v3 (current, start here).** Parts II/III are historical v2/v1 kept for migration context — do not reason about current behavior from them, they describe a daemon and a hosts file that no longer exist. |
| [testing.md](testing.md) | Add or debug a test. Has the full "What runs in CI" table with triggers and path filters. |
| [docs/android-build.md](docs/android-build.md) | Build, install, or profile the Android app. Includes the toolchain trap that costs an hour if you hit it blind (`cargo` resolving to Homebrew's rust, failing with `can't find crate for std`) and the adb recipes for granting accessibility and measuring startup. |
| [docs/android-generated-project-manual-edits.md](docs/android-generated-project-manual-edits.md) | Run `tauri android init` or edit `src-tauri/gen/android/` — it is committed, and re-initializing drops hand-applied patches. |

## Commands

`package.json` is the reference for what exists; only the traps are listed here.

- **Version bumps span several files** — always use
  `./scripts/bump-version.sh <version>`. It updates `package.json`,
  `src-tauri/tauri.conf.json`, `src-tauri/Cargo.toml`, and — via

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ulyngs/digital-habits-blocker](https://github.com/ulyngs/digital-habits-blocker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
