---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Pronto is a native macOS menu-bar app (SwiftUI `MenuBarExtra`, no Dock icon) that
turns a La Marzocco espresso machine (Linea Micra / Mini / GS3) on and off via the
La Marzocco **cloud** API. Bundle id `blog.pacificsky.pronto`. Released on GitHub as
an **unofficial** app (keep the trademark disclaimer in README intact).

## Build & run

```sh
./make-app.sh              # builds dist/Pronto.app (release), code-signs it
open dist/Pronto.app
swift build                # plain build; does NOT assemble the .app bundle
swift build -c release     # what make-app.sh wraps
```

`make-app.sh` is the real build entry point — `swift build` alone produces a bare
binary with no `Info.plist` (so no `LSUIElement`, no menu-bar behavior). The only
tests are `Tests/ProntoTests` (the Sentry scrubber's privacy guarantee); no linter
is configured. CI (`.github/workflows/ci.yml`) runs `swift build` + `swift test` on
`macos-15` after selecting the newest installed Xcode (the SDK must match the macOS
14 deployment target — some APIs like `openSettings` are 14.0).

## Code signing (important — affects Keychain prompts)

The signing identity controls whether stored credentials survive rebuilds:

- **Local dev (default):** `SIGN_IDENTITY="Pronto Local Signing"`. `make-app.sh`
  creates/reuses a self-signed identity. A *stable* signature keeps the Keychain
  ACL valid across rebuilds, so the app stops re-prompting for saved credentials.
  Prefer this when iterating. First build prompts twice: a macOS trust-password
  dialog, then a codesign keychain-access prompt — click **"Always Allow"** on the
  latter or it re-prompts every build (it seeds the key's partition list).
- **CI / release:** the release workflow imports an **Apple Developer ID Application**
  cert and builds with `SIGN_IDENTITY="Developer ID Application: …"`, which makes
  `make-app.sh` add `--options runtime --timestamp` (hardened runtime + secure
  timestamp — both required to notarize). The bundle is then notarized (`notarytool
  submit --wait`, App Store Connect API key) and the ticket **stapled** in. Result:
  no Gatekeeper warning and a *stable* identity across releases (no post-update
  Keychain re-prompt). Only a `Developer ID …` identity triggers those extra codesign
  options; ad-hoc (`-`) and the local self-signed cert don't (they stay offline).

`ensure_identity()` in `make-app.sh` imports the key + cert as **separate PEM
items**, NOT a PKCS#12 bundle — macOS's `security import` rejects OpenSSL 3's
default p12 (SHA-256 MAC / AES-256 bags / empty password), which silently drops
the private key and leaves an orphan cert. Don't switch it back to PKCS#12.

## Releases

Push a `vMAJOR.MINOR.PATCH` tag; `.github/workflows/release.yml` builds, signs with
Developer ID, notarizes + staples, zips, and publishes a GitHub Release. The version
comes from the tag via `APP_VERSION="<tag without v>"`. Signing/notarization is gated
on repo secrets (`DEVELOPER_ID_CERT_P12_BASE64` + password, `NOTARY_API_KEY_P8_BASE64`
+ `NOTARY_API_KEY_ID`/`NOTARY_API_ISSUER_ID`); if the cert secret is unset the release
**fails** (a release must be signed). See RELEASE.md. Apple's notary can occasionally
backlog for 40–60+ min (the CI step retries transient errors but caps at 60 min); when
it won't clear in time, notarize by hand with `notarize-local.sh` (same signing path via
`make-app.sh`, uncapped poll). See RELEASE.md.

`make-app.sh` also emits `dist/Pronto.dSYM` (via `dsymutil` — the plain
`swift build -c release` already carries a debug map, no `-g` needed) keyed by the
binary's `LC_UUID`, which code-signing leaves unchanged. The release workflow uploads
it to Sentry with `sentry-cli debug-files upload --include-sources` so crash reports
symbolicate. That step is gated on a `SENTRY_AUTH_TOKEN` secret (write-scoped; the
DSN, by contrast, is public) with `SENTRY_ORG`/`SENTRY_PROJECT` repo *variables* — if
the token is unset the upload is skipped and the release still publishes. The dSYM
lives beside the bundle, not inside it, so it's never in the user zip.

The workflow also creates + finalizes a **Sentry release** named after the version
(the tag) and associates commits (`sentry-cli releases new/set-commits/finalize`),
so events group by version with suspect-commit detection. This only works because the
app tags events with the *same* name — `CrashReporting` sets `options.releaseName` to
`CFBundleShortVersionString` — so keep those two in sync. Commit association uses
`set-commits --auto` (Sentry's GitHub integration must have the repo added; the
workflow's `fetch-depth: 0` checkout supplies the current SHA) and is best-effort: a
failure there warns but doesn't fail the publish.

## Architecture

Single executable target, `Sources/Pronto`, `@main` is `ProntoApp`. UI is driven
by one `@MainActor` view-model.

- **`ProntoApp.swift`** — `MenuBarExtra` + `Settings` scenes; accessory activation.
  The menu-bar icon reflects the selected machine's power via *distinct glyphs*
  (filled cup = on, `powersleep` = standby, outline cup = unknown, outline cup +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pacificsky/pronto](https://github.com/pacificsky/pronto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
