---
trigger: always_on
description: Working guide for Claude Code (and human contributors) in this repo. For the public
---

# CLAUDE.md

Working guide for Claude Code (and human contributors) in this repo. For the public
project overview, supported formats, and algorithm reference, read **README.md** — this
file is the *operating manual* for changing the code, not a re-description of it.

## What this is

FreeNote / NcmDecrypt — an offline Android decoder for DRM-wrapped local music files
(NCM / QMC family / TM / KGM / KWM …). Pure Kotlin, no server. The user supplies files
they already downloaded; the app strips the container encryption and writes plain
FLAC/MP3/OGG/etc.

**Network exception (the only one):** decryption is 100% offline, but the app optionally looks up
*album art* (`CoverFetcher.kt`) **and *lyrics*** (`LyricsFetcher.kt`) online for outputs that ship
without them (QMC/KGM/KWM — and lyrics for all formats, since even NCM headers carry no lyric). It
sends only the song's title/artist text, never audio, and is gated by two independent default-on
toggles: `CoverPrefs` ("联网补全封面") and `LyricsPrefs` ("联网补全歌词"). The `INTERNET` permission and
`res/xml/network_security_config.xml` exist solely for these. If you reintroduce a "no network at
all" claim anywhere, it is now wrong — keep the audio path offline, keep cover + lyric lookup the
single, opt-out-able network use. Lyrics are embedded into the audio tag (`FieldKey.LYRICS`) and
exported as a sidecar `.lrc`; shared HTTP lives in `MusicHttp.kt`.

- `namespace = com.ncmdecrypt`, `applicationId = com.braynlabs.freenote`, app label **FreeNote**.
- The three names (NcmDecrypt / FreeNote / com.braynlabs.freenote) are historical; don't "fix"
  them without a reason — renaming the applicationId breaks update installs.

## Build / install / test

```bash
# Debug build (default debug keystore if no keystore.properties present)
./gradlew assembleDebug          # → app/build/outputs/apk/debug/app-debug.apk

# Release build (signed only if keystore.properties + the .keystore exist; else unsigned)
./gradlew assembleRelease        # → app/build/outputs/apk/release/app-release.apk

# Preferred release signing hygiene: keep signing files outside the repo.
FREENOTE_KEYSTORE_PROPERTIES=/path/to/keystore.properties ./gradlew assembleRelease

# JVM unit tests for codec-adjacent pure logic
./gradlew :app:testDebugUnitTest

# Install to a connected device (-r replace, -t allow test, -g grant runtime perms)
adb install -r -t -g app/build/outputs/apk/debug/app-debug.apk
```

- **Toolchain**: Java 17, Android SDK 34, Gradle 8.5 (wrapper), AGP 8.2.2, Kotlin 1.9.22.
  `versionName` / `versionCode` live in `app/build.gradle.kts` (the *only* source of truth).
- **JVM unit tests now exist** under `app/src/test`. They cover pure logic around filename
  sanitization, EKey/MMKV parsing, QMC footer behavior, malformed header bounds, audio
  format detection, LRC parsing (`LrcParser`), and lyric-provider response parsing (`LyricsFetcher`).
  Full codec vectors should still come only from known-good reference
  implementations (see below). Real-device smoke testing remains manual: Bluetooth/`adb push`
  a real `.ncm` / `.qmc` / `.kgm` / `.kwm` to the phone, open it in the app, play the output.
- Historic test device: Vivo/OPPO `3142621725000KW` (Android 14+). If `adb install` hangs,
  unlock the screen / confirm the on-device install dialog.
- Signing: prefer keeping the real `keystore.properties` and keystore outside the repo, then
  set `FREENOTE_KEYSTORE_PROPERTIES=/path/to/keystore.properties` or pass
  `-PfreenoteKeystoreProperties=/path/to/keystore.properties`. Root-level
  `keystore.properties` still works for local compatibility and remains git-ignored.
- **Release signing material (this machine):** the real signing identity lives OUTSIDE the repo at
  `/Users/xuanfeng/claudecode/freenote-private-artifacts-20260606/signing/` —
  `keystore.properties` + `braynlabs.keystore` (alias `braynlabs`, cert SHA-256
  `8E:E3:94:24:…:DF:32:60:FF`; this is the cert v1.0.0–v1.3.0 shipped with — reusing it is
  required so update installs don't break). Build a signed release with:
  `FREENOTE_KEYSTORE_PROPERTIES=/Users/xuanfeng/claudecode/freenote-private-artifacts-20260606/signing/keystore.properties ./gradlew assembleRelease`.
  Passwords are in that `keystore.properties` only — never copy them here.

## Critical invariants — do not relearn these the hard way

1. **Crypto must match a reference implementation byte-for-byte. Never invent constants,
   tables, or round counts.** A previous version shipped *made-up* KGM/KGG/VPR/KWM algorithms
   (a hardcoded `"3HENGELING"` table, a fixed 256-byte XOR table) that only produced garbage.
   Everything is now ported from **unlock-music** (`algo/qmc/*`, etc.). If you touch a codec,
   re-verify against that source and its official test vectors before claiming it works.

2. **NCM is XOR-over-a-KeyBox, NOT RC4.** Flow: `keyData ^= 0x64` → AES-128-ECB decrypt with
   `SCORE_KEY` → `decryptedKey[17..]` builds a 256-byte KeyBox → XOR the audio with the box
   (not RC4 PRGA). `SCORE_KEY`'s last byte is **`0x57`**, not `0x61` — this exact byte has
   been gotten wrong before.

3. **QMC tail-type detection drives everything** (last 4 bytes of the file):
   `QTag` → ekey in trailer meta; a small LE u32 → `[audio][ekey][keyLen]`; `STag` / `cex\0`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Brayn-Studio/freenote](https://github.com/Brayn-Studio/freenote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
