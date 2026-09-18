---
trigger: always_on
description: This repository contains the source code for Drive Assist: the primary driver application (`drivemem/`), the privileged system helper (`modehelper/`), and the standalone setup installer (`installer/`).
---

# Geely / Drive Assist — Developer & Architecture Guide

This repository contains the source code for Drive Assist: the primary driver application (`drivemem/`), the privileged system helper (`modehelper/`), and the standalone setup installer (`installer/`).

The build toolchain (`sdk/`, `car-stubs/`) and keystore (`debug.ks`) can be resolved via `GEELY_TOOLS` or `build-env.sh`.

---

## Access & Security: ADB vs. In-App Privileges

### ADB runs as root (uid 0)
`adb shell` on this head unit connects directly as **uid 0 (root)**. The system runs an Android `user` build signed with **test-keys**, `ro.secure=0` and `ro.debuggable=1`. Commands executed over ADB do not need `su` escalation.

```bash
$ adb shell id
uid=0(root) gid=0(root) groups=0(root),1004(input),1007(log),1011(adb),... context=u:r:su:s0
```

> [!NOTE]
> `/system/xbin/su` is the AOSP form (`su [UID] [CMD]`), not `su -c CMD`. Because the ADB shell is already root, invoke commands directly.

### SELinux is Enforcing
Root over ADB does not grant Drive Assist in-app root. Under Enforcing SELinux, third-party application domains cannot transition into the `su` domain:
* **Workstation ADB commands** (inspecting `/proc`, `/sys`, modifying system settings) run as root without limitation.
* **Autonomous vehicle operations** inside the car must route through **`modehelper`** (platform-signed, `android.uid.system`, holding `INSTALL_PACKAGES`).

---

## Multi-Worktree Publishing Guard

`build.sh` protects against accidental OTA overwrites when multiple working trees or branches exist:

```bash
cd drivemem && ./build.sh     # compiles, verifies guard, then publishes
```

`build.sh` blocks publishing (not compilation) unless:
1. The integration branch is contained in `HEAD`.
2. The commit currently published on the server is contained in `HEAD`.

If the guard blocks a publish, it prints the missing commits and the exact `git merge <sha>` required.

* `NO_GUARD=1` overrides this protection when an overwrite is explicitly intended.
* `NO_DEPLOY=1` compiles without publishing (safe for rapid local iteration).

---

## Branching & Release Workflow (Public `next` -> Curated `master`)

* **`feat/*`, `fix/*`, `docs/*` (Public Feature Work)**: Branch from `next`,
  keep scope narrow, run `./tools/check-pii.sh --staged` before every push,
  and open PRs into `next`. A one-time `./tools/install-git-hooks.sh` setup
  adds the same guard to every push in this checkout.
* **`next` (Public Integration)**: Receives reviewed public feature/fix PRs and
  produces downloadable nightly candidate artifacts. Never deploys an OTA.
* **`release/vX.Y` (Public Stabilization)**: Cut from `next` when a version is
  feature-complete; only release-blocking fixes and validation work belong here.
* **`master` (Public Release)**: Tracks `origin/master`. Only clean, curated
  release commits and immutable version tags are pushed here.
* **To publish a new public release**:
  ```bash
  ./tools/validate-release.sh release/vX.Y
  ./tools/push-release.sh vX.Y.Z "summary of changes" release/vX.Y
  ```
  This creates a curated commit on `master` matching the validated release
  branch tree, adds an immutable tag, and pushes both to `origin`.

---

## Build System: Gradle & `build.sh`

`drivemem/` is a standard Gradle/AGP Android module (`src/main/java/com/geely/drivemem/`, `src/main/res/`, `src/main/AndroidManifest.xml`).

```bash
./gradlew :drivemem:assembleRelease   # compile + sign only (verify, iterate, test)
./gradlew test                        # unit tests (OdoStats, ChargeSession, CarDataHub, etc.)
```

### Full Release Pipeline (`build.sh`)
For release builds and deployment:
```bash
source ./build-env.sh                 # sets up JDK, Android SDK, and paths
cd drivemem && ./build.sh             # compile, verify guard, publish, and install
```

`build.sh` handles:
* Building both `drive_assist.apk` and the standalone `drive_assist_installer.apk`.
* Copying release artifacts and `changelog.txt` to the Home Assistant update directory (`/config/www/`).
* Retaining OTA announcement URLs on MQTT (`drivemem/<vin>/update/set`).
* Running `adb install -r` when a vehicle connection is active.

---

## Key Technical & Vehicle Safety Rules

### 1. Vehicle Safety & Park (`P`) Enforcement
* **Never show update modals while driving**: Update prompts and changelog modals must only appear when the vehicle is in Park (`CarState.isParked() == true`).
* If the vehicle shifts out of Park while a dialog is displayed, the dialog is immediately dismissed.
* OTA installations are blocked while the vehicle is in motion.

### 2. Display Sizing (1920 x 1080 @ 1.0 Density)
* **1 dp = 1 px = 1 sp**: The dashboard display is **1920 x 1080 dp** (density 1.0).
* A phone is ~390 dp wide; this panel is nearly five phones wide and viewed from ~75 cm away.
* Type and interactive touch targets must be sized for vehicle viewing distances (e.g., 13sp is only ~1.8 mm tall on this panel and is illegible from the driver's seat).

### 3. Application Lifecycle & Suspend
* **The head unit suspends rather than fully shutting down**: `BOOT_COMPLETED` does not fire on brief wakeups.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SuspiciousRepair/drive_assist](https://github.com/SuspiciousRepair/drive_assist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
