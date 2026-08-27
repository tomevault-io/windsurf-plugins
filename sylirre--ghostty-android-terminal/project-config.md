---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Android terminal emulator backed by Ghostty's VT engine (`libghostty-vt`).
Runs a Linux userland under `arm64chroot` — a bundled from-scratch AArch64
Linux user-space emulator (qemu-user-style ISA emulation + proot-style rootfs
containment, optional `--jit`) — so the aarch64 rootfs runs on both arm64-v8a
(JIT arm64→arm64) and x86_64 (JIT arm64→x86_64) hosts. Rootfs tarballs are
optional, gitignored APK assets from `UserlandRootfs/` — one per distro,
named `<id>_<version>_aarch64_rootfs.tar.xz` (built by
`scripts/build-alpine-rootfs.sh` / `scripts/build-debian-rootfs.sh`; aarch64
only, always); a first-run onboarding wizard (`OnboardingActivity`) explains
the app and installs the chosen distro. A third session type runs a whole
**guest machine** under `arm64emu` (full-system emulation: real kernel, real
init, real root) from optional, gitignored `VmImages/` assets — an EDK2
firmware and a bootable aarch64 ISO fetched by `scripts/fetch-vm-images.sh`;
its tabs are guest terminals, not processes. Also `/system/bin/sh` with
`PATH=/system/bin`; session tabs; extra-keys toolbar above the soft keyboard.
minSdk 29, targetSdk 36, ABIs arm64-v8a + x86_64.

Docs: [docs/architecture.md](docs/architecture.md) (design, data flow, key
decisions), [docs/native-build.md](docs/native-build.md) (Ghostty
cross-compile pipeline), [docs/testing.md](docs/testing.md) (test suites and
emulator setup), [README.md](README.md) (build requirements, usage).

## Commands

Gradle must run on JDK 17–21. If the system `java` is newer, prefix every
gradlew call, e.g.:

```sh
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
```

```sh
./gradlew :app:assembleDebug                  # build APK (also compiles JNI via CMake)
./gradlew connectedDebugAndroidTest           # all integration tests (needs device/emulator)

# Single class or test:
./gradlew connectedDebugAndroidTest \
  -Pandroid.testInstrumentationRunnerArguments.class=io.github.sylirre.terminal.EmulatorVtTest#lineWrap

scripts/setup-emulator.sh                     # one-time AVD creation (API 34 x86_64)
scripts/run-emulator.sh                       # boot it headless; needs /dev/kvm
```

There are no JVM unit tests: everything meaningful crosses JNI, so the whole
suite is instrumented (`app/src/androidTest/`). Espresso requires device
animations off (`adb shell settings put global window_animation_scale 0`,
plus `transition_animation_scale` and `animator_duration_scale`).

Regenerating the Ghostty prebuilts is only needed when bumping the pinned
commit (top of `scripts/fetch-ghostty.sh`) and requires Zig **0.15.x
exactly** — Ghostty rejects other majors at build time:

```sh
scripts/fetch-ghostty.sh && ZIG=/path/to/zig-0.15 scripts/build-ghostty-vt.sh
```

## Architecture

Three layers; native code is limited to what Java cannot do.

```
Java  app/src/main/java/io/github/sylirre/terminal/
  term/  TerminalNative (JNI surface + shared constants)
         TerminalEmulator (owns the native handle, all calls synchronized)
         TerminalSession (PTY + shell pid + reader thread)
         SessionCommand (execve command or arm64chroot argv + env + tab label)
         UserlandDistro (bundled rootfs asset discovery for the distro chooser)
         UserlandRootfs (rootfs asset → tar.xz install → arm64chroot command line)
         SessionManager (process singleton: sessions survive Activity recreation)
         VmOptions/VmMachine (arm64emu guest machine: one per process, a
         socketpair per guest terminal, tabs attach with a dup)
         ScreenSnapshot (flat viewport arrays for rendering)
  ui/    TerminalView (Canvas grid renderer + TYPE_NULL InputConnection)
         ExtraKeysView, TabStripView, MainActivity
         OnboardingActivity (first-run intro + distro chooser + install)
         Chrome/ChromePalette/TopBarView/EdgeInsets/Dialogs/KeyCaps (shared
         chrome: drawable factories + design tokens, theme-derived main-screen
         palette, secondary-screen top bar + insets, dialog kit, keycap factory)
JNI   app/src/main/cpp/   → libterm.so (CMake, NDK)
  pty_jni.c       openpt/fork + execve(sh) or arm64chroot_main(), TIOCSWINSZ, waitpid/kill
  terminal_jni.c  libghostty-vt bindings, snapshot flattening, key encoding
C     native/arm64chroot/ → arm64chroot (AArch64 user-space emulator) linked
                            into libterm.so; vendored from a sibling project,
                            `main` renamed under ANDROID_JNI. No loader.
C     native/arm64emu/    → arm64emu (AArch64 *full-system* emulator: a QEMU
                            'virt' machine, real kernel on emulated hardware).
                            Its own libarm64emu.so, -fvisibility=hidden,
                            exporting only `arm64emu_main` — it shares ~60
                            global names with arm64chroot, which in one object
                            collide and across two would cross-bind.
Zig   native/ghostty-vt/  → libghostty-vt.a prebuilt per ABI + vendored headers
```

Data flow: reader thread reads the PTY → `emulator.feed()` → response bytes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sylirre/ghostty-android-terminal](https://github.com/sylirre/ghostty-android-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
