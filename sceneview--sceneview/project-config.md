---
trigger: always_on
description: SceneView is an **AI-first SDK**: its primary goal is to enable Claude (and other AI
---

# SceneView — Claude Code guide

## Project purpose

SceneView is an **AI-first SDK**: its primary goal is to enable Claude (and other AI
assistants) to help developers build 3D and AR apps in Jetpack Compose. Every design
decision — API surface, documentation, samples, `llms.txt` — should be optimized so
that when a developer asks an AI "build me an AR app", the AI can produce correct,
complete, working code on the first try.

**Implication for contributors:** when adding or changing APIs, always ask "can an AI
read the docs and generate correct code for this?" If not, simplify the API or improve
the documentation until it can.

> **Start here.** Read [`.claude/STATE.md`](.claude/STATE.md) for *where we are* and
> [`.claude/workflows/README.md`](.claude/workflows/README.md) for *how we work* (the v2
> working methodology). This file holds stable project facts only — never session state.

## QUALITY RULES (MANDATORY — every session, every commit)

**ZERO TOLERANCE for bugs reaching the user.** Every change must be verified before push.

### Before EVERY push to main:
1. **Compile check**: `./gradlew :sceneview:compileReleaseKotlin :arsceneview:compileReleaseKotlin`
2. **Unit tests**: `./gradlew :sceneview:testDebugUnitTest :arsceneview:testDebugUnitTest`
3. **Bundle build** (if store-affecting): `./gradlew :samples:android-demo:bundleRelease`
4. **Website JS** (if website changed): `node -c website-static/js/sceneview.js`
5. **Full gate**: `bash .claude/scripts/pre-push-check.sh`

### Rules:
- NEVER push code that doesn't compile
- NEVER push without running tests
- NEVER modify website JS without validating syntax
- NEVER deploy to stores without verifying the bundle builds locally first
- When an agent modifies code, ALWAYS verify compilation before committing
- If a review finds blockers, fix them ALL before pushing — no exceptions
- If you bump `gradle/libs.versions.toml` → `filament = "X.Y.Z"`, you MUST recompile every `.filamat` blob in the same PR with the matching `matc` toolchain — see [CONTRIBUTING.md "Filament runtime ↔ .filamat ABI invariant"](CONTRIBUTING.md#filament-runtime---filamat-abi-invariant). v4.1.0 shipped split halves and crashed 10 demos at runtime.

### Quality plan: `.claude/plans/v4.0-quality-plan.md`

## Device QA

The **autonomous device-QA harness** (umbrella [#1560](https://github.com/sceneview/sceneview/issues/1560))
drives the demo apps **like a real user** — taps, swipes, camera-orbit drags,
navigation — and asserts no crash across every platform, unattended (no
screenshot-by-screenshot loop). CI-green plus self-review is not enough: a demo
can compile, pass unit tests, and still crash the moment it renders on a device.

### Run it

```bash
# Full cross-platform pass — every platform feasible on this host.
bash .claude/scripts/device-qa.sh --platform=all

# One platform, or a fast per-category subset.
bash .claude/scripts/device-qa.sh --platform=android
bash .claude/scripts/device-qa.sh --platform=web --fast
```

`device-qa.sh` is the single orchestrator entrypoint. It boots the
emulator/simulator each leg needs, builds + installs the demo app, delegates to
the per-platform harness, and aggregates every verdict into one
`device-qa-report.json` at the repo root (override the directory with `--out`).
Exit status is non-zero if any selected platform failed. Flags: `--platform=android|ios|web|ar|all`,
`--fast` (per-category subset, not the full catalog), `--ci` (treat a skipped
platform as a failure), `--out <dir>`.

### What each leg covers

| Leg | Harness | Drives | Report |
|---|---|---|---|
| `android` | Maestro flows `.maestro/android/` via `qa-android-demos.sh` | All 53 demos on an emulator | `device-qa-report.json` |
| `ios` | Maestro flows `.maestro/ios/` via `ios-device-qa.sh` | 63 deep-linkable demos on a simulator (AR = launch-only smoke) | `device-qa-report.json` |
| `web` | Playwright suite `samples/web-demo/tests/` | Browser 3D viewer + every catalog tab | `web-qa-summary.json` |
| `ar` | `ar-replay-qa.sh` + `ARReplayHarnessTest` | Every Android AR demo replayed against recorded ARCore sessions — no physical device | `ar-qa-summary.json` |

See [`.maestro/README.md`](.maestro/README.md) for the Maestro flow layout and
known limitations (no pinch gesture → 3D zoom is driven via deep-link param).

**iOS leg status — CI-wired since #2833 (2026-07-20), advisory.** `device-qa.yml`
now defines an `ios` job (Maestro / Simulator; routed to the self-hosted
`sceneview-mac` runner when its heartbeat is fresh, `macos-15` otherwise), the
nightly runs it via `device-qa.sh --platform=ios --fast --ci`, and
`render-tests.yml`'s iOS job drives the `SceneViewDemoUITests` UI-testing
target with real `XCTAttachment` screenshots. The leg is in the default
ADVISORY set — a red ios leg is a `WARN`, not a release block. Caveat: on the
self-hosted Mac the leg is disk-gated (< 10 GB free → honest advisory skip),
so keep the host's disk above the gate for real coverage.

### Release-checkpoint mandate

**A full device-QA pass runs at every release checkpoint, before tagging.**
No release ships with a red *blocking* leg in `device-qa-report.json`. The
gate is enforced in two places — keep both honest:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SceneView/sceneview](https://github.com/SceneView/sceneview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
