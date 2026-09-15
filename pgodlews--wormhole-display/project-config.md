---
trigger: always_on
description: handles HEVC VPS/SPS/PPS and random access, and `VideoRenderer` selects that
---

# AGENTS.md — guide for AI agents working on Wormhole Display

Read this before touching anything. It encodes hard-won facts about the AirPlay
protocol, this Android port, and the target devices. Skipping it will cost you
hours.

## What this is

An AirPlay-compatible **mirroring and extended display receiver** for Android,
developed and tested on Meta Portal+ (Gen 2), Portal Go and Portal TV (Android
9–10 / API 28–29, arm64, Wi-Fi only). A Mac, iPhone or iPad sees the device in
its native Screen Mirroring picker and streams video and audio to it. The heavy
lifting (RTSP, FairPlay, decryption, mDNS) is a vendored copy of UxPlay's C core
at `app/src/main/cpp/uxplay/`; the app layer is Kotlin + MediaCodec/AudioTrack.

## Repo layout

```
app/src/main/cpp/uxplay/lib/              vendored UxPlay core (DO NOT casually edit — see below)
app/src/main/cpp/uxplay/lib/llhttp/       bundled HTTP parser (MIT)
app/src/main/cpp/uxplay/lib/playfair/     FairPlay handshake crypto
app/src/main/cpp/uxplay/lib/mdnsd/        embedded mDNS responder (the Bonjour advertiser)
app/src/main/cpp/wormhole_jni.c           JNI shim: server lifecycle + all raop callbacks
app/src/main/cpp/deps/                    committed prebuilt static libs (openssl, plist)
app/src/main/java/io/github/pgodlews/wormhole/
    MainActivity.kt                       Compose dashboard + video surface
    WormholeService.kt                    foreground service: keeps the receiver discoverable, boot autostart
    WormholeServer.kt                     process-wide owner of the native server, renderers and UI state
    NativeBridge.kt                       JNI declarations
    VideoFrameQueue.kt, VideoRenderer.kt  compressed-frame queue → MediaCodec → Surface (H.264/HEVC)
    AudioRenderer.kt                      AAC-ELD decode → AudioTrack
    WormholeIdentity.kt                   persisted deviceid, service name, Wi-Fi multicast lock
scripts/build.sh                          build APK → dist/wormhole-display.apk
scripts/build-deps.sh                     rebuild the native deps (rarely needed)
scripts/test-native.sh                    host regression tests for the vendored mirror loop
docs/architecture.md                      design overview
docs/uxplay-patches.md                    inventory of vendored-code changes
```

Vendored UxPlay changes are listed in `docs/uxplay-patches.md` and marked
"Android port" in the source (the `raop_get_pk_str` getter is marked "Android
JNI addition"). Keep that inventory current and avoid unrelated vendor changes
so future upgrades remain diffable.

## Environment

- JDK 17 and the Android SDK. `scripts/build.sh` sets `JAVA_HOME` and
  `ANDROID_HOME` defaults, so plain `java` does not need to be on PATH.
- Native deps are **committed prebuilt** (`app/src/main/cpp/deps/`). You only
  need the NDK if you edit C code: AGP will use `.tools/ndk-r27d` if present
  (see `app/build.gradle.kts`) or any SDK-installed NDK. Pinned version:
  27.3.13750724. If you change the NDK version, update `ndkVersion` there.

## Build, deploy, run

```bash
./scripts/build.sh
adb install -r dist/wormhole-display.apk
adb shell am start -n io.github.pgodlews.wormhole/.MainActivity
```

Portal devices are reachable over ADB via USB-C. Device and sender must share a
Wi-Fi LAN (mDNS does not cross the USB cable; Portal has no ethernet).

## How a connection works (protocol flow)

1. Native `mdnsd` advertises `_airplay._tcp` and `_raop._tcp` (both → port 7000)
   with TXT records including `pk=<hex ed25519 pubkey>`, `model=AppleTV3,2`,
   `srcvers=220.68`, `features=0x5A7FFEE6,0x0`. This is what populates the
   sender's picker.
2. Client connects to TCP 7000, does `GET /info` (RTSP-style, **with CSeq**),
   then ed25519 `pair-verify` (transient — no PIN with our feature bits).
3. `POST /fp-setup` × 2 — FairPlay SAP handshake (in `playfair/`).
4. `SETUP` negotiates streams: video over TCP 7100 ("mirror data"), timing/control
   UDP 7011/7001/7101. Then `RECORD` starts the stream.
5. Video arrives AES-128-CTR encrypted; the core decrypts and calls
   `video_process` with **Annex-B H.264/HEVC** (parameter sets prepended after each
   parameter packet). The JNI shim copies it into a jbyteArray → `VideoFrameQueue`
   → `VideoRenderer` → MediaCodec.
6. Audio (AAC-ELD, 44.1 kHz stereo) is decrypted by the core and delivered to
   `audio_process`; the shim copies each frame to `AudioRenderer`.

## Verification recipes

Device seen by the sender (run on the Mac):

```bash
dns-sd -B _airplay._tcp local.        # must list the service name, e.g. "Wormhole Plus"
dns-sd -B _raop._tcp local.           # must list "<DEVICEID>@<service name>"
```

Server alive (run on the Mac; substitute the device's LAN IP):

```bash
printf 'GET /info RTSP/1.0\r\nCSeq: 0\r\nUser-Agent: AirPlay/320.20\r\n\r\n' \
  | nc -w 5 <device-ip> 7000 | head -c 200
# expect: "RTSP/1.0 200 OK" + application/x-apple-binary-plist body
```

Decode the plist body with `plistlib` to check advertised `displays[0]` width/
height/maxFPS. A 409 means another client is already connected.

Logs on the device:

```bash
adb logcat | grep -E "uxplay|Wormhole|AndroidRuntime|libc"
```

## Troubleshooting — every one of these was hit for real

| Symptom | Cause | Fix / notes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgodlews/wormhole-display](https://github.com/pgodlews/wormhole-display) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
