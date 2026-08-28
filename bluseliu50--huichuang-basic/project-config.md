---
trigger: always_on
description: Guidance for AI coding agents (and humans) working in this repository.
---

# AGENTS.md — huichuang_basic

Guidance for AI coding agents (and humans) working in this repository.

## Project

Unofficial Flutter client for 国家中小学智慧教育平台 (basic.smartedu.cn).
Product name: **惠窗中小学端**. Platforms: macOS / Windows / Linux / Android / iOS.
License: **CC BY-NC-SA 4.0 — non-commercial**. Never add code whose purpose is
commercial distribution.

## Hard rules

1. **NEVER commit secrets.** No tokens, `access_token` / `refresh_token` values,
   passwords, account numbers, API keys, `.env*`, `*.key`, `dev_secrets*.json`,
   `*token*.txt`. These paths are gitignored from the first commit — keep it that
   way. Before every push, audit the full history for leaked credentials —
   substitute the real test-account phone and password from your gitignored
   `dev_secrets.json`, then run:
   `git log -p | grep -iE '<phone>|<password>|access_token"[[:space:]]*[:=]|refresh_token"[[:space:]]*[:=]'`
   and confirm no *values* (test fixtures with dummy data are fine; real
   credentials are not). Test accounts go in a gitignored `dev_secrets.json`
   under the repo root or `/tmp`, never in the repo.
2. **Do not regress the pinned fixes** (see "Load-bearing decisions" below).
3. Platform API facts live in `.agents/skills/smartedu-streaming/SKILL.md`
   (exposed to Claude Code via the committed `.claude/skills/smartedu-streaming`
   relative symlink — do not delete that link). If the platform changes,
   re-verify with curl/browser first, fix code, then update that skill — it is
   the durable knowledge base.
4. Biometric auth gates **only** the credential vault (saved-password unlock),
   never app launch. Tests assert this; keep them green.
5. New dependencies require a strong justification written in the commit body.
   Current dependency set is intentional and locked.

## Architecture

```
lib/main.dart                     — bootstrap: proxy startup, providers, E2E hooks
lib/src/api/                      — SmarteduClient (dual-mirror failover) + models
   └ catalog.dart                 — CatalogService (tag trees, materials, textbooks,
                                     module_version disk cache, local search)
lib/src/stream/proxy.dart         — 127.0.0.1 auth-injecting HLS/PDF proxy (linchpin)
   └ key_vault.dart               — HLS key dance: md5 sign + AES-128-ECB unwrap
lib/src/auth/                     — AuthController (silent refresh), webview login
   └ token_store.dart, biometric.dart
lib/src/store/app_state.dart      — AppController: selection, chapters, history
lib/src/ui/                       — app_shell (adaptive rail/bottom bar), courses,
                                     player (media_kit + custom controls), pdf,
                                     search (local index), settings, home, login
tool/live_check.dart              — real-platform end-to-end proxy verification
test/                             — unit tests with byte-level real fixtures
```

Data flow: UI → AppController → SmarteduClient (public static JSON on
`s-file-{1,2}.ykt.cbern.com.cn`) ; playback → StreamProxy rewrites the playlist
and injects `X-ND-AUTH`, performs the key dance, and fails over
r1→r2→r3 nodes; media_kit (mpv) plays the proxied URL.

## Build & test

The project pins its Flutter SDK with **fvm** (`.fvmrc`, committed; CI reads
the same file). Always run toolchain commands through fvm:

```bash
fvm install                 # once per machine: SDK for the pinned version
fvm flutter pub get
fvm flutter analyze         # must report: No issues found
fvm flutter test            # must be all green
fvm flutter run -d macos    # daily driver platform
HC_TOKEN=<access_token> fvm dart run tool/live_check.dart   # live proxy proof
```

Verification bar for merging "done" work: `flutter analyze` clean + tests green;
any proxy / player / auth change additionally requires a `live_check` PASS or a
macOS E2E log containing `PLAYER_OPEN` and `PLAYER_DURATION <n>s`.

macOS E2E: `HC_E2E_TOKEN='<full token json>' HC_E2E_RESID=<resId> <debug binary>`.

## Continuous integration (desktop)

`.github/workflows/desktop.yml` builds release bundles for macOS (arm64),
Linux (x64) and Windows (x64) and attaches them to the published release.
It runs ONLY on:

1. Manual dispatch: Actions → desktop-build → Run workflow.
2. `release: published`.

Deliberately no push/commit-message triggers — a three-platform build is
expensive, and commit-message CI ("编译测试" magic strings) invites junk
pushes just to kick builds. If a build is wanted, press the button or
publish the release.

- CI holds no certificates, so the macOS job rewrites signing in the
  checked-out `project.pbxproj` via sed (sdk-scoped identity → `-`,
  Automatic → Manual, team → empty) and builds ad-hoc. Ad-hoc artifacts
  run, but Keychain items do not survive a signing-identity change —
  vault-reliable macOS builds must be signed locally with the real team.
- Linux window icon: `assets/icon.png` is installed into `data/` by
  `linux/CMakeLists.txt` and loaded relative to `/proc/self/exe` in
  `linux/runner/my_application.cc`. Keep all three in sync with the icon
  master (`assets/icon.png` is the single source for every platform).
- Release assets are installers, not raw build trees: Windows Inno Setup

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bluseliu50/huichuang_basic](https://github.com/bluseliu50/huichuang_basic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
