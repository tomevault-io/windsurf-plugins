---
trigger: always_on
description: Workspace-level rules (search safety, patch performance, build and dependency integrity) also apply when they are present. Project skills live in `.agents/skills/`.
---

# Repository Agent Rules

Workspace-level rules (search safety, patch performance, build and dependency integrity) also apply when they are present. Project skills live in `.agents/skills/`.

## Device safety

- Installing a build on the user's device is allowed when needed for the requested validation.
- Never launch, interact with, navigate, or otherwise control the user's device through adb or any other device-control mechanism without explicit permission. This includes reading logs, clearing logcat, and pulling files.
- For runtime reproduction, ask the user to use the app normally and report the failure or send a screenshot/logs.

## Build and patch testing

- Build the patch bundle from the current checkout before patch testing: `./gradlew :patches:build --no-daemon`.
- Run patch tests through `./patch-ig-cli.sh`, not a direct Morphe CLI invocation. It needs the local patcher runner (`(cd ../morphe-patcher && ./gradlew :cli:installDist)`). Pass the exact APK explicitly, for example `./patch-ig-cli.sh apks/449.0.0.52.84.apk` (the script's default is an older target); the output goes to `~/Downloads/piko-ig-lite-patched-cli.apk` unless `OUTPUT_APK` is set.
- Do not patch with a stale MPP or mix artifacts from different source revisions. Confirm the script's output path and `Patched APK:` line before testing.
- Do not pass `--install` or otherwise control a device unless the user explicitly requests it; ask the user to launch and exercise the patched app.
- Before handing off a resolver change run `./gradlew :patches:lintResolvers :patches:checkExtensionDescriptors --no-daemon`.

## Supported targets

`COMPATIBILITY_INSTAGRAM` in `patches/src/main/kotlin/app/crimera/patches/instagram/utils/Constants.kt` is the only list of supported versions. Do not duplicate it or branch on a version number.

- Every resolver change must be patched against the newest declared target and at least one older declared target, and the resolved names compared before and after.
- Older APKs that are no longer declared (for example 439) are resilience cross-checks only. Do not add version-specific paths for them; report anchors that only hold on one release.
- A successful patch is not proof the feature works. The patch can apply with the wrong member resolved; check what was resolved, not only that `Applied` was printed.

## Patch philosophy and methodology

Instagram is an obfuscated app under active refactoring. A patch must survive ordinary R8 churn where possible, but must never guess when the app's behavior or contract has changed. The architecture is documented in `docs/ig-patch-architecture.md`; the skills `morphe-resolver-anchors`, `morphe-patch-development-workflow` and `morphe-runtime-boundaries` are the detailed rules.

### Philosophy

- Treat the exact target APK's smali as bytecode truth. Use readable builds to understand intent, not to choose production instructions.
- Match semantic invariants, not obfuscated identity. Short class names, method names, field names, and numeric instruction positions are reconnaissance evidence only.
- Resolve capabilities from the APK, not versions. If genuinely different contracts exist (for example a value read before or after its key), select an explicitly validated shape at patch time and share the common mutation afterward.
- Prefer one deep semantic resolver over a growing list of release-specific fingerprints.
- Fail closed. Zero matches, ambiguous matches, an unexpected signature, or an unproven control-flow path must raise a useful `PatchException`; never silently skip required behavior.
- Validate what a resolver returns, not only that it returned something. A resolved field or method must have its type and signature asserted (for example "a `List` field"), because "the first `iget-object` after the key" is not proof.
- An optional resolver (`matchOrNull`) must not leave a live extension placeholder behind. A placeholder that survives patching turns into a runtime failure on a path the patcher reported as `Applied`.
- Patch the smallest value at the latest safe point that controls the behavior. Avoid modifying shared helpers when the same helper serves unrelated features.
- Keep compile-time model contracts separate from release bytecode. Stable models may be `compileOnly`; unstable owners, methods, and descriptors belong in patch-time resolution and direct smali injection. Do not add new runtime reflection over release classes.
- Preserve old targets. A resilience improvement must keep the old path's behavior unchanged.

### Typed bytecode emission

Hooks are emitted through the [`crimera:morphe-bytecode`](https://github.com/crimera/morphe-bytecode) library, never by assembling smali strings:

- `method.insertHook(index, relocateBranchTargets = …) { … }` with `Block` emitters and `Target.Local` / `Target.Original` as branch targets.
- State the branch policy whenever the insertion point carries labels. Scratch registers are bounded by `RegisterLimit` and fail closed.
- A layer fix belongs in the library repository, followed by a version bump in `patches/build.gradle.kts`. Do not add new smali string templates.

### Runtime API compatibility


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crimera/piko-ig-lite](https://github.com/crimera/piko-ig-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
