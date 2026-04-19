---
trigger: always_on
description: - Always read this file before every code change
---


# CLAUDE.md — Fireplace

**Rules:**
- Always read this file before every code change
- Update this file after every code change
- **Before any review or code change:** read ALL files the task touches before writing anything; trace every code path; verify method/field/event names in actual source — never guess
- Single source of truth for agents — if CLAUDE.md says X, X is correct
- All code in English (vars, functions, comments, commits). Polish OK in .md files only

---

## 0. Quick Start

```bash
# Terminal 1: Backend + DB (auto hot-reload)
docker-compose up

# Terminal 2: Flutter web (press 'r' for hot-reload)
cd frontend && flutter run -d chrome
```

**Before start:** Kill stale node processes: `taskkill //F //IM node.exe`
**Low C: disk workaround:** `cd frontend && .\run_android_on_x.ps1` (sets `GRADLE_USER_HOME` + `TEMP`/`TMP` on `X:`, junction `frontend/build` → `X:\fireplace-build\frontend-build`; keep default Pub cache on `C:`). **Device:** `.\run_android_on_x.ps1 -DeviceId emulator-5554` (do not use `-d` here — PowerShell/flutter shim can misparse it as “Target file … not found”). Plain `flutter run` for Android still uses `%USERPROFILE%\.gradle` on `C:`; `flutter clean` does **not** clear it — corrupt `metadata.bin` there needs cache delete or using this script / `GRADLE_USER_HOME=X:\gradle-home`.

**Ports:** Backend :3000 | Frontend :random (check terminal) | DB :5433 (host) -> :5432 (container)

**Stack:** NestJS 11 + Flutter 3.x + PostgreSQL 16 + Socket.IO 4 + JWT + self-hosted media (`MEDIA_BASE_URL` / disk volume; Nginx `X-Accel-Redirect` in prod)

**Phone (same WiFi):** `cd frontend && .\run_web_for_phone.ps1` or `flutter run -d web-server --web-hostname 0.0.0.0 --web-port 8080 --dart-define=BASE_URL=http://YOUR_PC_IP:3000`

**Tests:** `cd backend && npm test` (233 unit tests, 30 suites, no DB required); `cd frontend && flutter test` (88 tests; includes `test/widgets/message/bubble_redesign_test.dart`). Media crypto round-trip tests skip when `webcrypto:setup` was not run (e.g. no cmake on Windows); the 20MB limit test always runs (throws before native crypto).

**Production:** https://fireplace.ignorelist.com — Google Cloud e2-medium VM (Warszawa), Docker + Nginx + Let's Encrypt. Deploy: SSH to server → `~/deploy.sh` (git pull + docker build + flutter web build).

---

## 1. Critical Rules & Gotchas

### TypeORM
- Always `relations: ['sender', 'receiver']` on friendRequestRepository queries — without: empty objects/crash
- Use find-then-remove for friend_requests delete — `.delete()` can't use nested relation conditions
- Always `new Date(val).getTime()` for expiresAt comparisons — TypeORM returns string or Date
- `deliveryStatus` never downgrades — enforced via `DELIVERY_STATUS_ORDER` map
- `synchronize: true` — column additions auto-apply on restart. No migrations

### Frontend
- `file_utils_stub.dart` / `file_utils_io.dart` — conditional import for temp file deletion (web: no-op; native: dart:io)
- Android 16KB page-size compatibility: `zipalign -P 16` can pass while app still shows compatibility warning — verify ELF `LOAD` alignment with `llvm-readelf -l` for `.so` files. In current state `webcrypto` (`libwebcrypto.so`) is built with `Align 0x1000` (arm64 + x86_64), while `libflutter.so` / `libdatastore_shared_counter.so` are 16KB-safe.
- `flutter pub run webcrypto:setup` is for `flutter test` / scripts only (builds `.dart_tool/webcrypto/...`), not for Flutter app plugin binaries packaged into APK/AAB.
- `frontend/patch_webcrypto_16k.ps1` patches cached `webcrypto` Android `CMakeLists.txt` with linker flags `-Wl,-z,max-page-size=16384 -Wl,-z,common-page-size=16384` to produce 16KB-compatible `.so` files; `run_android_on_x.ps1` executes it before `flutter run`.
- `MainActivity` package must match Android namespace/applicationId (`com.fireplace.app`); mismatch (`com.rpgchat.frontend`) causes runtime crash `ClassNotFoundException: com.fireplace.app.MainActivity`.
- Android build outputs should stay at the default `frontend/build` path for Flutter tooling; to use `X:` storage, map `frontend/build` to `X:\fireplace-build\frontend-build` via junction.
- Do not move `PUB_CACHE` to a different drive than the project root on Windows (`X:` vs `C:`) for Android builds; Kotlin incremental caches can fail with `IllegalArgumentException: this and base files have different roots`.
- Android/Kotlin workaround for mixed-drive cache paths: `frontend/android/gradle.properties` sets `kotlin.incremental=false` to avoid daemon cache-close failures on Windows.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lentach) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
