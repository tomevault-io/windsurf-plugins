---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Andrew's Patches** — a **Morphe Patches** bundle (Morphe is a fork of the ReVanced patching ecosystem). It produces an `.mpp` patch bundle that the Morphe CLI / Manager applies to third-party Android APKs to rewrite their bytecode. The current focus is **LINE** (`jp.naver.line.android`). Base package/group is `app.andrewliang` (app-agnostic); per-app patches nest under `app.andrewliang.patches.<app>` (e.g. `app.andrewliang.patches.line`), and target-app compatibility lives in `app.andrewliang.patches.shared.Constants`. The developed-against version is pinned there.

## Commands

```bash
# Build the patch bundle -> patches/build/libs/patches-*.mpp
./gradlew buildAndroid

# Build, then regenerate patches-list.json from the compiled bundle
./gradlew generatePatchesList

# Compile-check without producing a release (what CI runs on non-release commits)
./gradlew :patches:buildAndroid clean --no-daemon
```

There is no test suite. Correctness is validated by applying the built `.mpp` with the Morphe CLI against a real target APK. `generatePatchesList` reflectively loads the built `.mpp` and re-emits `patches-list.json`, so it depends on `build` having run.

`settings.gradle.kts` pulls the `app.morphe.patches` Gradle plugin and patcher libraries from GitHub Packages (`maven.pkg.github.com/MorpheApp/registry`). Building requires `gpr.user`/`gpr.key` Gradle properties **or** `GITHUB_ACTOR`/`GITHUB_TOKEN` env vars with a PAT that can read those packages.

**Local build/verify without a PAT:** if the patcher/plugin artifacts are already in the Gradle cache, build fully offline with *dummy* credential values — `./gradlew :patches:buildAndroid --offline --no-daemon -Pgpr.user=dummy -Pgpr.key=dummy` (the settings plugin only needs the credentials to be non-null when nothing is fetched). Then apply with the bundled Morphe CLI (`java -jar work/morphe-desktop-*.jar patch --exclusive -e "<name>" -o work/out.apk … work/apkm-extract/base.apk`) and inspect the patched dex with **dexlib2** (no `baksmali` CLI ships — STRIP_FAST writes modified classes into a small fresh `classes.dex`). Full recipe + the LINE class/anchor map live in **`docs/line-patch-map.md`**.

## Architecture

Two Gradle modules (`settings.gradle.kts`):

- **`patches/`** — Kotlin. The patches themselves, written against the `app.morphe.patcher` API. This is where nearly all work happens.
- **`extensions/extension/`** — Java, compiled as an Android library to `extensions/extension.mpe`. Holds complex runtime logic that is injected *into* the target app.

### How a patch works

The patching model is: **fingerprint → locate method → inject smali → optionally delegate to extension code**.

1. **Fingerprint** — declaratively describes a method in the *target app* by defining class, name, access flags, return type, parameters, and a list of instruction `filters` (field access, string references, method calls, opcodes, literals). Partial/obfuscation-tolerant matching applies. Prefer anchoring on **string literals** and non-obfuscated class names, since obfuscated names (`Sg1.c`, method `b`, …) change between LINE versions. Declaring fingerprints as named objects/classes means failures name the fingerprint in the stack trace.
2. **Patch** — `bytecodePatch { ... }` with `name`/`description`/`default`. In `execute { }` it resolves the fingerprint's `method` and mutates it via extensions like `addInstructions(index, smali)`. Injected smali calls the extension with `invoke-static {}, Lapp/andrewliang/extension/...;->method()Z`.
3. **`extendWith("extensions/extension.mpe")`** — bundles the compiled extension so injected smali can call it. Simple fixed-value overrides need no extension; use extension Java only for real logic.
4. **`compatibleWith(...)`** / `dependsOn(...)` — declare target-app compatibility (`Constants.COMPATIBILITY_LINE`) and patch dependencies.

**Patch visibility:** a `bytecodePatch` with a `name` is user-facing (shown in Manager/CLI); a nameless one is an internal dependency, hidden from users but pulled in via `dependsOn`.

**Compatibility** (`app/andrewliang/patches/shared/Constants.kt`) — `Compatibility` objects declare target `packageName`, app name, `apkFileType`, icon color, and `AppTarget` version list. `version = null` means "any/latest" (often `isExperimental = true`); always pin at least one confirmed-working version.

### Patcher API notes (hard-won)

- **Finding instruction indices:** there is no `indexOfFirstInstructionOrThrow` (that's ReVanced). Use `fingerprint.instructionMatches[i].index` — one match per instruction `filter`, in program order — or `instructionMatchesOrNull` for best-effort. `.method` and `.instructionMatches` are context-receiver accessors usable inside `execute { }`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrewliang25/morphe-patches](https://github.com/andrewliang25/morphe-patches) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
