---
trigger: always_on
description: **Any time the user asks to build or rebuild an AAB**, always bump the version first:
---

# GitHub Copilot Instructions for ObsidianBackup

## AAB Build Rule (MANDATORY)
**Any time the user asks to build or rebuild an AAB**, always bump the version first:
1. Increment `versionCode` by 1 in `app/build.gradle.kts`
2. Increment the patch in `versionName` (e.g. `1.0.3` → `1.0.4`)
3. Then build with `./gradlew bundleFreeRelease` (or requested flavor)

Never build an AAB without bumping both values first.

---

## Session Startup: ADB Logcat (MANDATORY)

**BEFORE issuing any `adb` command in this session**, always launch logcat first as a background async shell to capture all device output:

```bash
adb -s BAUOUKZ9ZTNVZPXK logcat -v time > /mnt/workspace/ObsidianBackup/logcat_session.log 2>&1 &
```

This must be the **first action** taken at session start in `/mnt/workspace/ObsidianBackup`, even before the user explicitly asks for ADB operations. Use `mode="async"` with the bash tool so logcat runs in the background throughout the session. Confirm it started by checking the PID, then proceed with whatever the user asked.

---

## Quick Reference: Build Commands

```bash
# Assemble specific variant (preferred for development)
./gradlew assembleFreeDebug
./gradlew assemblePremiumRelease

# Run a single unit test
./gradlew test --tests "com.obsidianbackup.SpecificTest"

# Run unit tests for a module/variant
./gradlew :app:testFreeDebugUnitTest

# Instrumentation tests (requires device/emulator)
./gradlew connectedAndroidTest
./gradlew connectedAndroidTest -Pandroid.testInstrumentationRunnerArguments.class=com.obsidianbackup.SpecificTest

# Static analysis
./gradlew detekt
./gradlew lintRelease

# Coverage
./gradlew jacocoTestReport

# Full test pipeline (script)
./scripts/run_tests.sh

# Build optimizations
./gradlew buildInfo                    # Show build configuration
./gradlew cleanAll                     # Clean all build outputs including cache
```

## Quick Reference: Physical Device Testing

**Primary test device:** Ulefone Armor X13, Android 15, Magisk-rooted
- Serial: `BAUOUKZ9ZTNVZPXK`
- su path: `/system_ext/bin/su`
- Package: `com.obsidianbackup.free.debug`
- Navigation drawer Y coords (720×1600): Dashboard≈420, Apps≈533, Backups≈620, Automation≈827, Gaming≈940, Health≈1053, Plugins≈1166, Logs≈1346, Settings≈1453

```bash
# Install free debug (PRO_GATING_ENABLED=false — no paywalls)
./gradlew assembleFreeDebug
adb -s BAUOUKZ9ZTNVZPXK install -r app/build/outputs/apk/free/debug/app-free-debug.apk

# Launch
adb -s BAUOUKZ9ZTNVZPXK shell am start -n com.obsidianbackup.free.debug/com.obsidianbackup.MainActivity

# Logcat (app only)
adb -s BAUOUKZ9ZTNVZPXK logcat -s "ObsidianBackup" -v time

# Screenshot
adb -s BAUOUKZ9ZTNVZPXK shell screencap /sdcard/screen.png && \
  adb -s BAUOUKZ9ZTNVZPXK pull /sdcard/screen.png /mnt/workspace/ObsidianBackup/screen.png

# Root verification
adb -s BAUOUKZ9ZTNVZPXK shell /system_ext/bin/su -c "id"
# Expected: uid=0(root) gid=0(root)
```

## Quick Reference: Architecture

### Modules
- `:app` — Main Android app
- `:root-core` — Root detection library (Magisk/KernelSU/APatch), ShellExecutor, SELinux, BusyBox
- `:tv` — Android TV variant
- `:wear` — Wear OS variant
- `:enterprise` — **NEW** Enterprise backend (Spring Boot 3.2 + Kotlin, PostgreSQL, Redis)

### Layers (Clean Architecture)
```
Presentation (Compose UI + ViewModels)
        ↓
Domain (UseCases + BackupOrchestrator)
        ↓
Data (CatalogRepository + CloudProviders)
        ↓
Storage (Room DB + BackupCatalog + File I/O)
```

### Backup Pipeline
```
AppsViewModel.backupApps()
  → BackupAppsUseCase
    → BackupOrchestrator.executeBackup()  # retry logic, incremental strategy
      → ObsidianBoxEngine.backupApps()    # root shell: cp APK, tar data
      → verifyAndCatalog()                # Merkle verify + Room DB + JSON save
```

---

# COMPREHENSIVE IMPLEMENTATION GUIDE

## CONTEXT
You are working on **ObsidianBackup**, a production-grade Android backup application with root capabilities, multi-platform support (Wear OS, Android TV, Web), cloud integrations, and enterprise features. This codebase has been audited and is 72% production-ready with specific gaps that need addressing.

**Version:** Kotlin 2.0.21 | compileSdk/targetSdk 35 | minSdk 26 | JDK 17

## YOUR ROLE
Implement fixes and features following the audit findings. Maintain the existing high-quality standards while addressing critical blockers, security vulnerabilities, and incomplete implementations.

---

## CRITICAL PRINCIPLES

### 1. Security First
- **NEVER store sensitive data unencrypted** - Always use Android Keystore or EncryptedFile
- **ALWAYS validate external inputs** - Checksums for binaries, signature verification for plugins
- **NEVER trust user input in shell commands** - Use the existing ShellSanitizer
- **ALWAYS encrypt PHI/PII** - Health Connect data MUST use AES256_GCM encryption
- **Validate JWT secrets** - Never use defaults in production (generate with `openssl rand -base64 32`)
- **Use Row-Level Security (RLS)** - PostgreSQL multi-tenant isolation via `app.current_organization`
- **Hash-chain audit logs** - SOC 2 compliance requires tamper-evident logging

### 2. Billing & Monetization Protection
- **Enforce feature gates at repository/use-case level** - Never just UI-level

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [canuk40/ObsidianBackup](https://github.com/canuk40/ObsidianBackup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
