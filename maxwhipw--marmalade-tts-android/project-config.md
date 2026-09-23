---
trigger: always_on
description: `REPO-MAP.md` at the project root is the orientation pass for this
---

# marmalade-tts-android — project notes for Claude

## Orientation: read REPO-MAP.md first

`REPO-MAP.md` at the project root is the orientation pass for this
codebase. It covers module structure, key files by concern, data
flow, conventions, and known quirks (TTS engine registration
requirements, nested-Scaffold inset handling, Hilt + ComponentActivity
constraint, the runBlocking hot-path cache pattern, etc.). Read it
before doing exploratory Grep/Glob work.

When spawning a subagent for investigation or implementation in this
repo, include **"Read REPO-MAP.md first"** in the briefing. The
subagent inherits this CLAUDE.md but won't read the map unless told.

Keep `REPO-MAP.md` current — when you discover a new gotcha or
architectural choice a future agent should know, update the map in
the same commit as the change.

## Remotes

**github is authoritative — and PUBLIC.** The repo is live at
github.com/maxwhipw/marmalade-tts-android with users' eyes on it
(issues enabled; Settings → Report a bug links straight to it, and the
privacy policy is served from it).

```
github   https://github.com/maxwhipw/marmalade-tts-android.git   (public, authoritative)
origin   <local-forgejo>/marmalade-tts-android.git   (Forgejo, local)
```

Push posture (Max, 2026-07-27): **Forgejo (`origin`) may be pushed
loosely** — it's local infrastructure, low consequence. **github is
heavily vetted**: every push there needs Max's explicit all-clear,
with the usual pre-push review (secrets, personal/infra details,
half-finished work) — an all-clear for one push does not carry to the
next.

## Versioning

Bump `versionCode` + `versionName` in `app/build.gradle.kts` per
release. v0.1.x is debug-signed only — `applicationIdSuffix = ".debug"`,
so the installed package on devices is `app.marmalade.tts.debug`.
Commits of the form `vX.Y.Z: ...` mark a version bump.

When working on a batch of changes that would warrant separate
logical commits, split them — even if the work was done in one
session (recent v0.1.15/16/17 splits used `git stash` to peel apart
mixed working trees cleanly).

## Working patterns that have produced good results

These are conventions Max + Claude have converged on; following them
reproduces the rhythm that landed v0.3.0-alpha.7's perf work.

### Letter-named feature atoms

When working through a multi-step optimization or refactor, name each
discrete change by a single letter (A, B, C, …) and reuse those names
through the conversation, commits, and task descriptions. Lets both of
us track parallel threads at a glance — "did we land C yet?" beats
"the per-device thread autodetect change with the setting."

When you discover a follow-up to an already-named change after the
fact, suffix the digit: `A2` for "extension of A". Don't reflow letters.
Always keep the letter assignments in your task tracker.

### One change → compile → install → test → iterate

Land each lettered atom *individually* on the device before moving to
the next. Each step is small enough that:
- Compile-check via `./gradlew :app:compileDebugKotlin` (10s) catches
  trivially-bad refactors before the longer `assembleDebug` (1-2 min).
- Per-change logcat traces let you attribute deltas correctly. Bundling
  A+B+C into one APK and seeing a 40% speedup tells you nothing about
  *which* change earned it.

### Adaptive auto-detection + manual override

For per-device tunables (thread count is the canonical example), pair
a runtime autodetect (`CpuClusterDetector`) with a Settings-screen
manual override. Auto handles the 95% case; the override exists for
the long tail (exotic SoCs, user benchmarks). Same pattern fits
intra-op spinning, XNNPACK toggles, EP selection, etc.

### Make on-device behavior identifiable from logs

When the build state has multiple dimensions a future you might want
to attribute behavior to (precision variant, EP choice, thread count,
quantization strategy), log the active selection at engine load. The
goal isn't a particular log format — it's that when comparing a synth
that sounds good against one that doesn't, you should be able to tell
from logcat alone which build state produced each.

### Trust on-device evidence over speculation

The session that landed the perf work moved fast because each
hypothesis was verified on the device before committing to the next
step — even when the model said "this should help." XNNPACK looked
like it was regressing per-frame time until logcat surfaced the
spinning-contention warning; F (Euler tensor reuse) turned out to be
below measurement noise, which we'd never have known without
shipping it alone. Default to measuring rather than assuming.

### Compare at the model boundary when integrations diverge

When two integrations of the same model disagree on audio quality
(e.g. sherpa-Kitten vs KittenDirect, both running the identical
KittenML ONNX), the difference is in the **inputs**, not the model.
Inspect, in order: ONNX `metadata_props` for baked-in priors (this is
how we found sherpa's hidden `speaker_speed_priors: 0.8,...,0.9`), the
exact token sequence reaching the model, the voice/style indexing
logic, and the scalar reaching the `speed` input. Days of code
investigation won't find what one `onnx.load(...).metadata_props` will.

## Distribution flavors — `play` vs `fdroid`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxwhipw/marmalade-tts-android](https://github.com/maxwhipw/marmalade-tts-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
