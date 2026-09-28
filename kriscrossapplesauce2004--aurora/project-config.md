---
trigger: always_on
description: Android convergence layer for the reference OnePlus 7 (`guacamoleb`, SM8150 /
---

# Aurora - project context

Android convergence layer for the reference OnePlus 7 (`guacamoleb`, SM8150 /
Adreno 640). Android stays PID1; a Debian LXC guest on the same downstream
kernel takes the display via libhybris→hwcomposer. Ships as custom boot.img +
Magisk module + Zygisk - never a ROM, never touches /system.

**Continuing an existing session?** Read `docs/CONTINUE.md` first. It is the
session protocol: orientation, house rules, tracking, device access, and
private-branch handling. Then run `tools/agent-status.sh`.

**Read first:** `docs/design-spec.md` (authoritative design), `docs/recon-findings.md`
(device ground truth), `README.md` (repo map + milestones).

## Device (recon-verified, don't re-derive)

- crDroid 12.11 / Android 16 (SDK 36), slot `_a`. Magisk 30.7 + ReZygisk.
- Kernel `4.14.357-openela` (crDroid sm8150 fork, branch `16.0`).
- Composer: HIDL `graphics.composer@2.1–2.4` (no AIDL). Gralloc: QTI mapper@4.0.
- binderfs in kernel; guest gets private binderfs in its IPC ns.
- DP-alt over USB-C works; Android native desktop mode runs on it.

## Talking to the phone

- USB cable connected (fastboot recovery possible). Wireless adb as fallback
  (official platform-tools only, port rotates per reboot).
- **Never `adb root`**. Root = `adb shell "su -c '<cmds>'"`. If su returns
  permission denied, Shell toggle in Magisk Superuser tab is off.
- Flash path: `usb-install/host-flash.sh check|flash|restore|verify` or Magisk
  action zips. Dry-run: `touch /sdcard/Download/aurora-dryrun`.

## Build system

- `kernel/build.sh`: merges `aurora.config` onto running kernel config,
  verifies all options, compiles. Toolchain in `toolchain/` (gitignored).
- `boot/repack.sh`: swaps kernel into boot.img via magiskboot (from `toolchain/usr/bin`).
- Zygisk: `cd zygisk && ndk-build NDK_PROJECT_PATH=. APP_BUILD_SCRIPT=jni/Android.mk NDK_APPLICATION_MK=jni/Application.mk`
- Companion: `~/android-sdk/gradle-8.7/bin/gradle --no-daemon assembleDebug` in `companion/`.
- NDK at `~/android-sdk/ndk/27.2.12479018`. Platform-tools at `~/platform-tools`.

## Conventions

- The compositor is named **Hyprland**, with runtime prefix `/opt/hyprland`.
- Current bring-up: `docs/arch-opal-bringup.md`. User requires no mode switches;
  use host builds and display-safe probes until explicitly authorized otherwise.
- Next work handoff: `docs/handoffs/alarm-apps-opal-osk.md`. Fresh ALARM needs
  installed applications; ship fixes through the installer. OSK source work is
  in `~/.local/state/opal/staging/`, not the installed host shell.
- Probe/script outputs → `artifacts/`. Structured recon → `recon/report-*/`.
- Commit as work lands; use the contributor's repository-configured identity.
- `~/op7-port/` + pmOS = mainline kernel track. Don't mix with Aurora.

### Comment discipline

- Comment why only when the code cannot make it obvious. Do not narrate the code.
- Keep routine comments to one line. Put incident history, dates, evidence, and
  extended rationale in docs or commit messages.
- Longer comments are reserved for dangerous invariants, hardware quirks, and
  constraints whose removal could cause data loss, boot failure, or a device wedge.
- During reviews, delete stale or redundant comments instead of preserving them
  as archaeology.

## Graphics invariant — explicit user requirement

Hyprland and Sxmo must use vendor EGL/GLES through **libhybris**,
Android gralloc allocations with complete native handles and sync fences, and
libhybris/hwcomposer for internal presentation. Do not substitute Mesa/Zink,
Turnip, raw KMS, or a nested KWin session to claim delivery. Native graphics
results below are historical experiments, not the product implementation path.
Modern Hyprland uses Aquamarine, not the Droidian wlroots ABI; implement and
validate the needed backend/renderer integration rather than enabling an
incompatible session manifest. `docs/graphics-architecture.md` governs this.

## Key technical facts

**Guest compositor stack:** phoc 0.47 (droidian `group/102/keypad-slide-lights`)
on droidian wlroots fork (`feature/next/backport-0.18`), built in-guest against
upstream libhybris. Build script: `guest/build-wlroots-phoc.sh`.

**libhybris:** built from upstream master (has PR #609, A15/16 support).
`guest/build-libhybris.sh`. Includes: GSK struct-varying rewrite hook (Adreno
flat-struct bug), eglSwapBuffersWithDamageKHR override, epoxy EGL_EXT_device_query
filter, HWCNativeWindowSetBufferCount, setDisplayBrightness(1.0) after power-on.

**hwc2-compat:** standalone NDK cross-build, `hwc2-compat/build.sh`. Installs to
guest `/usr/lib/android/`.

**GPU app buffers:** hybris wayland EGL platform (zero-copy). Clients need
`EGL_PLATFORM=wayland HYBRIS_EGLPLATFORM=wayland`. `GSK_RENDERER=ngl`.
`/etc/profile.d/hybris.sh` sets defaults. Gate: `guest/gpu-smoke.sh`.

**Input:** wlroots EVIOCGRAB handoff, libinput udev properties
(`aurora-input-udevdb`), quirks for touchpanel, seatd needs /dev/tty0-2.

**Non-root session (2026-07-12):** the guest desktop runs as unprivileged
`aurora` (uid 1000), NOT root. `desktop-on` does root-only prep (udev DB, seatd,
create `/run/user/1000`, `chmod a+r /etc/phoc.ini`) then `runuser -u aurora`
launches phoc + phosh; runtime dir is `/run/user/1000`. Device access works

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kriscrossapplesauce2004/aurora](https://github.com/kriscrossapplesauce2004/aurora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
