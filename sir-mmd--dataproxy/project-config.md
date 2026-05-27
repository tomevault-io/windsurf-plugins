---
trigger: always_on
description: A SOCKS5 proxy for Android that pins outbound traffic to the cellular network.
---

# Working in this repo

A SOCKS5 proxy for Android that pins outbound traffic to the cellular network.
Project directory is still `xlink/` (kept for git history); the Gradle project,
Android package, and brand are all `DataProxy` / `com.dataproxy`.

Latest released version is **v1.1.0** (versionCode 6). Fixes the long-running
"rapid stop→start leaves proxy stuck" bug, restores LAN inbound (which
v1.0.0 silently broke via `bindProcessToNetwork`), adds a live cellular
tech + operator header indicator, and overhauls the permissions dialog.

## Build & run

The user has Android Studio installed at `/opt/android-studio` (bundled JBR
Java 21) and the SDK at `/home/mmd/apps/Android/sdk` (only `android-36` /
`build-tools 36.x` are present — **do not** drop `compileSdk` back to 35).
There's a downloaded Gradle at `/home/mmd/apps/gradle-8.10.2/bin/gradle`.

```bash
# Release APK (the one shipped to GitHub Releases)
/home/mmd/apps/gradle-8.10.2/bin/gradle :app:assembleRelease \
  --no-daemon --console=plain 2>&1 | grep -E "^(e:|FAIL|BUILD)" | tail -15

# Install on the connected Samsung device
cp app/build/outputs/apk/release/app-release.apk /tmp/DataProxy-vX.Y.Z.apk
adb -s R5CY50CMNWM install -r /tmp/DataProxy-vX.Y.Z.apk

# Crash check
adb shell am force-stop com.dataproxy && adb logcat -c
adb shell monkey -p com.dataproxy -c android.intent.category.LAUNCHER 1
adb logcat -d AndroidRuntime:E "*:S" | tail -20

# End-to-end smoke test once the proxy is running (use time.ir, NOT 8.8.8.8 /
# example.com — the user's mtnirancell cellular network blocks those):
curl --socks5-hostname 127.0.0.1:1080 -s -o /dev/null \
  -w 'http=%{http_code} time=%{time_total}s\n' --max-time 25 https://time.ir
```

`org.gradle.java.home=/opt/android-studio/jbr` is set in `gradle.properties`
so command-line builds pick up the JBR automatically (system Java is 26,
which Gradle 8.10 doesn't officially support — it works but warns).

If multiple ADB devices show up (e.g. an emulator + the phone), pass
`-s R5CY50CMNWM` — that's the physical Samsung the user actually tests on.

## Release process

1. Bump `versionCode` + `versionName` in `app/build.gradle.kts`.
2. Bump `app_version` in `res/values/strings.xml` and the literal `v0.x.y` in
   `HomeScreen.kt`'s `Footer()`.
3. Build, install, smoke-test on device.
4. Commit, push, then:
   ```bash
   gh release create vX.Y.Z /tmp/DataProxy-vX.Y.Z.apk \
     --repo Sir-MmD/dataproxy \
     --title "DataProxy vX.Y.Z" \
     --notes "..."
   ```

**Never push without explicit instruction.** Default is to build + install
locally, hand the APK to the user, and let them test before any commit.
Commit messages must not include `Co-Authored-By` or any AI attribution.

The signing keystore is `app/keystore/dataproxy-release.jks` (gitignored,
password `dataproxy`). Same key has been used for every release — installs
upgrade in place. If the keystore file is missing, the build still succeeds
with an unsigned APK (signing config is conditional in `build.gradle.kts`).

## Architecture invariants

These are load-bearing — break them and the app stops working.

- **Never call `cm.bindProcessToNetwork(cellular)`.** It tags every socket
  the process creates — including the SOCKS5 listener — with the cellular
  netId. The kernel then routes the listener's SYN-ACK reply via the
  cellular route table, so external clients on Wi-Fi never finish the TCP
  handshake (SYN_RECV → retransmits → timeout). v0.2.1 → v1.0.0 had this
  bug; v1.1.0 removed all `bindProcessToNetwork` calls. The "DNS leak"
  rationale was theoretical only — every hostname resolution in this app
  goes through `Network.getAllByName()` via `cellular.resolveHost()`, and
  there is no `URL` / `OkHttp` / `HttpClient` anywhere on the data path.
- **Domain DNS goes through `Network.getAllByName`.** In `Socks5Connection
  .openRemote`, hostnames are resolved via `cellular.resolveHost(host)` —
  this is now the *only* DNS path, not belt-and-suspenders. If you ever
  add a code path that calls `InetAddress.getByName(hostname)` you've
  introduced a leak.
- **`CellularNetworkProvider` is a lazy singleton on `ProxyService`** (`val
  cellular by lazy { ... }`). It is *not* recreated per start/stop cycle.
  An attempt to make it per-cycle (during the v1.0.0 stale-state-after-
  `adb install -r` investigation) regressed rapid power-toggle: building a
  *new* `NetworkCallback` object every cycle exposes a race in
  `ConnectivityManager.unregisterNetworkCallback` → `requestNetwork` when
  the second call lands a few ms after the first. The same callback object
  re-registered is fine. **Don't reintroduce per-cycle recreation.**
- **`fullCleanup()` is called on BOTH start and stop.** `ProxyService` has
  a single helper that cancels every job (including the `startJob` that
  wraps `awaitAvailable`), stops the server, calls `cellular.stop()`,
  resets the registry/sampler, and releases the wake lock. It runs at the
  top of `startProxy` *before* `cellular.start()` and again inside
  `stopProxy`. Without the start-side call, a leftover `awaitAvailable`
  watcher from a partially-aborted previous cycle could fire `onAvailable`
  after the new launch had already built a `Socks5Server`, racing them on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sir-MmD/dataproxy](https://github.com/Sir-MmD/dataproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
