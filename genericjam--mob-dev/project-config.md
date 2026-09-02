---
trigger: always_on
description: generates the Elixir stub via Igniter, and re-runs `mob.regen_driver_tab`
---

# AGENTS.md — mob_dev

You're in **mob_dev**, the build/deploy/devices toolkit. Read
[`~/code/mob/AGENTS.md`](../mob/AGENTS.md) first for the system view, the
three-repo topology, the cross-cutting pre-empt-failure rules, and the
**"Don't write this slop"** list (AI-generated patterns to avoid at write
time, not after credo flags them). The notes below are mob_dev-specific.

## What this repo is

Mix tasks (`mob.deploy`, `mob.connect`, `mob.devices`, `mob.emulators`,
`mob.provision`, `mob.doctor`, `mob.battery_bench_*`) plus their backing
modules (`MobDev.Discovery.{Android,IOS}`, `MobDev.NativeBuild`,
`MobDev.OtpDownloader`, `MobDev.Deployer`, `MobDev.Emulators`).

The **release tooling** lives at `scripts/release/` — shell scripts for
cross-compiling OTP for Android arm64/arm32, iOS sim, and iOS device, then
staging the tarballs and uploading to GitHub Releases. Patches we apply to
OTP source for iOS-device compatibility live at
`scripts/release/patches/` (`forker_start` skip, EPMD `NO_DAEMON` guard).
See `build_release.md` for the full release walkthrough.

## TDD is the practice here

Write tests before or alongside new code. Every new function should have
corresponding tests before the task is considered done. The test suite must
stay green at all times.

```bash
mix test                       # all tests
mix test --exclude integration # skip the device-dependent ones
```

## Things that bite specifically in mob_dev

- **Compile-time regex literals are unsafe** on Elixir 1.19 / OTP 28.0. Use
  `Regex.compile!("...", "flags")` for runtime compilation. Already swept in
  0.3.17 — don't reintroduce.
- **Hex packages omit repository-root dotfiles by default.** Code under `lib/`
  must not compile-time read `.tool-versions` or another root-only file. Keep a
  packaged authority in source, enforce exact lockstep with the root file in a
  source test, and compile the unpacked Hex artifact in the regression suite.
- **`mix mob.deploy --device <id>`** resolves the id via discovery before
  deciding which platform to build. The narrowing logic is in
  `narrow_platforms_for_device/2` and is the single source of truth for both
  build and deploy. Bypass it and you'll get either spurious "No device
  matched" warnings (deploy) or builds for the wrong platform (build).
- **Deployment BEAM discovery follows Mix's active paths.** Use
  `Mix.Project.build_path/0` for dependency output and
  `Mix.Project.compile_path/0` for every application BEAM, including modules
  compiled from `erlc_paths`. Hard-coding `_build/dev` can push a stale,
  incomplete override that shadows the complete application bundle.
- **Physical iOS BEAM overrides must be exact and self-verifying.** The app
  prefers `Documents/otp/<app>` over its complete signed bundle. Replace that
  directory rather than incrementally merging it, require `<app>.beam` before
  transfer, and verify the received bootstrap bytes before restarting.
- **`xcodebuild` errors get rewritten** to actionable hints by
  `diagnose_xcodebuild_failure/1` in `mob.provision`. Apple's verbatim text is
  preserved alongside our hint so the snippet stays google-able. Add new
  pattern matches there when you encounter a new Apple error string.
- **APNs push token never arrives on iOS device** if the binary's codesigning
  entitlements omit `aps-environment`. `NativeBuild.codesign_ios_device_app/3`
  auto-mirrors the value from the embedded provisioning profile into the
  fallback entitlements. If the profile was provisioned without push, no
  mirroring happens — either re-provision with push enabled or create
  `ios/<AppName>.entitlements` with `aps-environment: development`. Test the
  plist text via `NativeBuild.fallback_entitlements_plist/3`.
- **OTP tarball schema changes need bumping `valid_otp_dir?/2`** in
  `otp_downloader.ex` so existing caches auto-redownload. Don't bump the OTP
  hash — the schema check is the right knob.
- **The release scripts assume `~/code/otp` exists** with the right cross-compile
  output. The patches in `scripts/release/patches/` are applied automatically
  by `xcompile_ios_device.sh`, idempotently — re-running is safe.
- **The application-build Zig version is exact.** `MobDev.Toolchain` embeds the
  root `.tool-versions` pin and both native build preflight and `mob.doctor`
  reject any other version. `mob.adopt` deliberately does not rewrite an
  existing Phoenix project's toolchain file: it installs `build.zig`-bearing
  native trees, then the preflight reports a missing or conflicting pin with
  the exact mise command. Changing an adopted app's existing language/runtime
  pins without its owner's consent would be destructive.
- **`mob.add_nif` is the entry point for new NIFs.** Don't add `:static_nifs`
  entries by hand to `mob.exs` — the task already does the AST-aware append,
  generates the Elixir stub via Igniter, and re-runs `mob.regen_driver_tab`
  so `priv/generated/driver_tab_*.zig` stays in sync. `--type` of `c`,
  `zigler`, `rustler` also drops the right native skeleton; `elixir-only`
  (default) leaves the C/Zig/Rust to you. The stubs for zigler/rustler
  carry an explicit static-link warning — those backends produce dlopen'd
  `.so` by default, which is wrong for Mob's iOS App Store /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GenericJam/mob_dev](https://github.com/GenericJam/mob_dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
