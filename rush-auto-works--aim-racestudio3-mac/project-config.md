---
trigger: always_on
description: Free one-click macOS (Apple Silicon) installer for AiM RaceStudio 3 (Wine-backed, notarized DMG).
---

# aim-racestudio3-mac — project notes for agents

Free one-click macOS (Apple Silicon) installer for AiM RaceStudio 3 (Wine-backed, notarized DMG).
**Read `codemaps/*.md` first** — architecture, the bash engine, and the build/release pipeline live there.
This file is constraints, conventions, and hard-won gotchas only.

## Conventions

- **Releases**: tag `v<RS3 version>-<pkg rev>` (e.g. `v3.83.20-2`) on `main` — Debian/RPM-style
  `upstream-revision`. The tag triggers `release-dmg.yml` → build + notarize + publish. The upstream
  version is `RS3_PINNED_VER`; the packaging revision is `RS3_PKG_REV` (both in `pins.env`). Asset is
  `RaceStudio3-<version>-<rev>.dmg`; `CFBundleShortVersionString` = upstream version (clean, what users
  see), `CFBundleVersion` = `<version>.<rev>`. Bump `RS3_PKG_REV` by hand when re-releasing the same
  upstream version; it resets to `1` on an upstream bump (`check-rs3-update.sh --apply` does this and
  tags `v<newver>-1`). Keep `CHANGELOG.md` in sync (one `## [<ver>-<rev>]` entry per release).
- **Updating RS3**: don't hand-edit version pins. `weekly-rs3-update.yml` (Mon 12:00 UTC) detects a
  newer AiM release and auto-bumps `pins.env` + tags + releases. To do it manually, run
  `installer/build/check-rs3-update.sh --apply` then tag.
- **The DMG build is macOS-only** (osacompile/codesign/hdiutil/notarytool). It does NOT run on
  Ubicloud/Linux. `release-dmg.yml` uses `macos-14`; the cheap weekly *check* uses `ubicloud-standard-2`.
- **A local `build-apps.sh` run is signed but NOT notarized** (no notary creds locally). Notarized
  artifacts only come from CI (ASC key in secrets) or by setting `NOTARY_PROFILE`/`NOTARY_KEY…`.
- **CodeRabbit findings**: use the `cr-check` skill (or `~/.claude/scripts/cr-check.sh`), never raw
  `gh api` comment parsing. Follow the bot-review loop: green CI **and** `reviewDecision == APPROVED` before merge.
- **End-of-session learnings** for this Rush repo: `session-learnings-rush` skill (PR to the shared KB).

## Hard rules (don't break)

- **Data safety** (`lib/data.sh::data_relocate_safe`): copy-if-absent + atomic symlink swap, fully
  re-entrant. Never make it overwrite an existing user file or `rm -rf` the data dir. The user's data wins.
- **App-menu name** comes from `CFBundleName` patched into each Wine unix-loader's embedded
  `__info_plist` at build time (`patch-wine-appname.py`), NOT argv[0]. Keep step 1c before signing.
- **macOS gotchas**: BSD `stat -f` (not `-c`); Unix socket paths ≤104 chars; writing `/Applications`
  needs admin (engine falls back to `~/Applications/AiM`).

## Don't retry (ruled out, with reasons)

- **NSStatusItem menu-bar helper** (removed): invisible under Bartender / Tahoe. Abandoned.
- **Custom items in Wine's macOS app menu** — **SUPERSEDED (achieved, PR #14).** `winemac.drv`
  builds the menu in compiled Cocoa with no config hook, so it *does* require rebuilding from source —
  but that's just ONE module (`winemac.so`), not all of Wine. We patch `dlls/winemac.drv/cocoa_app.m`
  (`installer/wine-patch/winemac-native-menu.patch`) to add Import / Uninstall / Show Logs items + fold
  in ⌘Q, build the single x86_64 module (`build-winemac-so.sh`, under Rosetta to match the osx64
  bundle), and swap it in (`build-apps.sh` step 1d). The old "not worth it" framing assumed a full
  Wine rebuild. Build-verified; on-device acceptance is the final gate.
- **Cmd-Q to quit RS3**: the native app-menu Quit (`terminate:`) **does reliably quit RS3** — verified
  on device 2026-06-07 (⌘⌥Q quit a running RS3). The old "RS3 ignores `WM_QUERYENDSESSION`" worry
  applies to *forwarding a keystroke into the app*, NOT the AppKit menu item, which terminates the Wine
  process directly. We bind it to the Mac-standard ⌘Q in the `winemac.so` source patch (a
  `setKeyEquivalentModifierMask:` change folded into `winemac-native-menu.patch`; was the
  `patch-wine-cmdq.py` post-build binary edit before PR #14, now retired) — no Accessibility grant
  needed (the menu owns ⌘Q; no global keystroke intercept). `wineserver -k` is still the hard kill
  used by the Uninstall app.
- **Trusting `lsappinfo`/`localizedName`** for the menu name (filename-derived, not the menu title).
- **Patching `CFBundleExecutable` to fix the Dock name** ("wine"): doesn't work — macOS derives the
  Dock/process name from the real on-disk loader filename, not the plist. Worse, a `CFBundleExecutable`
  that doesn't match the actual binary breaks LaunchServices icon resolution → blank Dock icon.
  `patch-wine-appname.py` patches `CFBundleName` only. Verified 2026-06-02. Dock "wine" is accepted (like Cmd-Q).
- **`DYLD_INSERT_LIBRARIES` to interpose Wine's sockets** (for the WiFi loopback redirect): doesn't
  fire — DYLD insert is **not honored for Rosetta-translated x86_64 processes** (macOS 26.4.1), and the
  Wine unix-loader is x86_64-under-Rosetta. Proven 2026-06-07 via `installer/bridge/test/interpose_rewrite.c`
  (loads into native arm64, never into `arch -x86_64`; independent of signing/hardened-runtime). The
  WiFi redirect (Phase 2) uses a Wine **source patch** instead. See `docs/plans/2026-06-05-wifi-loopback-bridge.md`.
- **win32 `ws2_32` proxy/hook DLL** (to redirect RS3's sockets without rebuilding Wine): no clean

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rush-Auto-Works/aim-racestudio3-mac](https://github.com/Rush-Auto-Works/aim-racestudio3-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
