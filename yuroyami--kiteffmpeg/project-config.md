---
trigger: always_on
description: KiteFFmpeg is a Kotlin Multiplatform binding to FFmpeg's libav* libraries, built for thirteen
---

# KiteFFmpeg, for whoever works in this tree

KiteFFmpeg is a Kotlin Multiplatform binding to FFmpeg's libav* libraries, built for thirteen
targets. The player that consumes it lives in the sibling checkout, `../KitePlayer`. Open work
for both is in GitHub Issues, one tracker per repository.

`CONTRIBUTING.md` has the build prerequisites, the ground rules and the gate. This file has
only what reading the code or running the gate would not teach you.

## How work happens here

- Work on `main`. Never create a branch without asking. Commit locally, never push. The owner
  pushes, publishes and cuts every release.
- Commit subject is one imperative sentence about the outcome. Short prose body. No trailers.
- When the tree contradicts an issue, stop and report it. Do not improvise the issue back into
  truth. Prose drifting from the tree is this project's measured failure mode.
- A design act is its own commit. Deciding a public API shape and executing it never happen in
  the same breath.
- Size estimates rot the same way claims do. An estimate made behind a blocker is a guess about
  what the blocker hides; re-size when the blocker falls.

## Gotchas

Each line is something that bit someone. Delete a line when it stops being true.

### Build and toolchain

- `apiCheck` and `apiDump` run with `-Pkiteffmpeg.requireAllTargets=true` and all eleven native
  trees present, because the dump covers thirteen targets. Dumping with `hostTargetsOnly` writes a
  three-target dump and CI fails on the target lines alone, which looks exactly like a real break.
  Cinterop on a machine with one FFmpeg tree still wants `-Pkiteffmpeg.hostTargetsOnly=true`.
- Publishing for the sibling player needs all three flags together,
  `-Pkiteffmpeg.phoneTargetsOnly=true -Pkiteffmpeg.withDesktopTargets=true -Pkiteffmpeg.jni.linux=true`,
  because a publish regenerates the root module metadata and a host-only publish deletes the
  ios, linux and mingw variants from it.
- `-Pkiteffmpeg.jni.linux=true` and `-Pkiteffmpeg.jni.windows=true` cross-link the desktop JNI
  libraries with konan and write the small platform JNI header themselves, so Docker is not
  needed; a Maven Central publish refuses to run without both switches, and the jar then carries
  Linux x64, Linux arm64 and Windows x64 libraries beside the macOS one.
- FFmpeg's configure cannot handle a `#` anywhere in its path, so the build tasks build in the
  system temp directory; Gradle's own `temporaryDir` is inside the project and fails.
- `--disable-postproc` does not exist in the vendored FFmpeg line and makes configure fail
  outright; postproc is already off by default.
- `--disable-asm` also disables SIMD, because SIMD is gated as an architecture extension, so a
  build labelled "simd" that carries the flag is a plain build and its measurement is a lie.
- The Android NDK is probed from `ANDROID_NDK_HOME`, `ANDROID_NDK_ROOT`,
  `ANDROID_NDK_LATEST_HOME` and two default directories only, never from `sdk.dir` in
  `local.properties`, so an SDK outside the standard paths fails every Android target with
  "Android NDK not found" until the variable is exported.
- A rebaked FFmpeg tree does invalidate its consumers, and an earlier note claimed the
  opposite: the wiring is deliberate, documented beside the cinterop block, and reading a
  second invocation's UP-TO-DATE as evidence of a break is how the wrong claim was made.
- `symbol-audit.sh` prefers the Gradle-built archive and only falls back to the host one when
  Gradle produced nothing, so running `build-host.sh` and then the tool reports on whatever
  Gradle last built, which can be days old.
- `run-c-tests.sh` never builds anything, so on its own it proves nothing about a source
  change; run `build-host.sh <variant>` first, every time.
- Moving or renaming the checkout breaks the prebuilt C test binaries: they carry an absolute
  rpath to their interpose library from link time, so every suite aborts naming the old path,
  which reads like a broken test and is a stale binary.
- A Gradle compile task with no sources prints `NO-SOURCE` and exits zero, so "the target
  compiles now" can mean "there was never anything there to compile".
- Adding a dependency can poison Kotlin's incremental-compilation cache, and the failure names
  a stdlib function and reads like a compiler bug in your own code; delete the module's
  `build/kotlin` and build again.
- `./gradlew ... | tail` reports the exit code of `tail`, so a failed build can look like a
  successful one; pipe to a file and check the log for BUILD FAILED.
- Scraping every Gradle configuration gives a load-dependent answer, because which
  configurations are realised depends on the rest of the task graph; only `api`,
  `implementation`, `compileOnly` and `runtimeOnly` can reach a POM.
- The atomicfu Gradle plugin is banned in every module: its bytecode transform registers a task
  depending on `androidMainClasses`, which the Android plugin's multiplatform library variant
  does not create. The library dependency itself is fine.
- The two macOS CI jobs deliberately both build FFmpeg on a cold cache, one running the
  ratchets and one building the real reduced permissive profile users get; merging them was

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuroyami/KiteFFmpeg](https://github.com/yuroyami/KiteFFmpeg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
