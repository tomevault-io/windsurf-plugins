---
trigger: always_on
description: This is a visual-QA service for iOS apps. It uses Codex (subprocess) + mobile-mcp to drive an iOS simulator, screenshot every step of key user flows, then critique the screenshots through a UX rubric.
---

# autobot — project context

This is a visual-QA service for iOS apps. It uses Codex (subprocess) + mobile-mcp to drive an iOS simulator, screenshot every step of key user flows, then critique the screenshots through a UX rubric.

## Architecture

Two-pass loop:

1. **Drive pass** — Codex executes flow goals step-by-step via mobile-mcp. Screenshots every action.
2. **Critique pass** — Each screenshot is re-evaluated with a UX rubric. Issues flagged in a per-run HTML report.

The CLI (`bin/autobot`) is a thin bash wrapper that:
- builds + installs the target app
- boots the simulator
- writes a temporary `.mcp.json` enabling mobile-mcp
- invokes `Codex --print` with a prompt template + tools allowlist
- collects screenshots + writes the HTML report

Codex does the actual thinking. The CLI is plumbing.

## v1 scope (intentional)

- Local Mac only, simulator only
- One app per target repo (no multi-tenant)
- Bash CLI, no compile step
- Discovery is interactive (Codex proposes flows, user confirms in AGENTS.md)
- Reports are static HTML in `.autobot/reports/`

## Audio input (feeding the simulator's mic)

iOS Simulator uses the host Mac's default audio input as the device microphone. To pipe synthesized audio into the simulator, we route playback through a virtual audio device that's also the system input:

1. Install BlackHole: `brew install blackhole-2ch`
2. Audio MIDI Setup app → create a Multi-Output Device that includes BlackHole 2ch + your real output (so you can still hear playback while testing).
3. System Settings → Sound → **Output**: the Multi-Output Device.
4. System Settings → Sound → **Input**: BlackHole 2ch.

Verify with `autobot doctor` (the audio line should report a loopback device).

Then any flow step can use:
- `autobot speak "phrase"` — TTS + play in one shot
- `autobot tts "phrase" out.aiff` — TTS to a file (e.g. archive in the run report)

`autobot speak` works without BlackHole — but the simulator won't hear it; it'll just play through your speakers.

For higher-quality voices: macOS Settings → Accessibility → Spoken Content → System Voice → download a "Premium" / "Enhanced" voice (e.g. "Ava (Premium)"), then `autobot tts "..." with AUTOBOT_TTS_VOICE=Ava`.

## v2 candidates (deferred)

- **Flutter project support**: detect `pubspec.yaml` at the input path → build via `flutter build ios --simulator --debug` → install the `.app` produced under `build/ios/iphonesimulator/Runner.app`. Smoke-test target Constella lives at `/Users/tejas1/Documents/Constella Codebases/mobile` (Flutter, bundle `ai.beemo.constella`, `CFBundleExecutable = Runner`). Flutter apps expose a thin iOS a11y tree — mobile-mcp's Visual Sense will be exercised heavily. Note this for the critique pass: a11y tree calls may return little, fall back to screenshots.
- **GitHub Actions macOS runner**: `macos-14` or `macos-15` images come with Xcode and iOS simulators preinstalled. Sketch:
  - Trigger: `pull_request` on relevant paths
  - Steps: checkout → cache DerivedData → `xcrun simctl boot` → `npm i -g mobile-mcp` → `Codex` login via secret → `autobot run` → upload `report.html` as artifact → post PR comment with summary
  - Gotcha: `Codex` auth in CI needs API key auth (`ANTHROPIC_API_KEY`), not OAuth
  - Gotcha: mobile-mcp requires `appium` for some platforms — pin versions in `package.json` to avoid runner drift
  - Cost: macOS runners are ~10x Linux. Expect 3-5 min/run; gate on `paths:` filter
- Real-device support (`.ipa` install, code-signing)
- Flow drift auto-healing (when a button rename breaks the flow, propose the fix)
- Parallel flow execution (one sim per flow, fan out)
- Comparison mode: diff a PR run vs the main-branch baseline

## Key files

- `bin/autobot` — CLI entry, subcommand dispatcher
- `src/lib/sim.sh` — `simctl` wrappers (boot, install, launch, shutdown)
- `src/lib/build.sh` — `xcodebuild` for `.xcodeproj` / `.xcworkspace`
- `src/lib/Codex.sh` — spawn `Codex --print` with right MCP + prompt
- `src/templates/discover-prompt.md` — system prompt for discovery pass
- `src/templates/run-prompt.md` — system prompt for run+critique pass
- `src/templates/critique-rubric.md` — UX checklist used by critique pass
- `src/templates/app-AGENTS.md` — template written into `<target>/.autobot/AGENTS.md` after discovery

## v2 (`v2-engine/`) — ONE engine tree, explore is the default brain

**`v2-engine/` is the canonical v2 engine** — shared core + platform drivers:

- `v2-engine/core/` — everything platform-independent, written ONCE: the explore
  loop (`runExplore(driver, cfg)`), the exploration doctrine + prompt skeletons
  (`prompts.mjs` — platform text is injected as slots), the schema builders,
  generic state-graph ops, and the critique/annotate passes.
- `v2-engine/mobile/`, `v2-engine/web/` — the "hands": each has `driver.mjs`
  (observe/execute/recover against mobile-mcp or Playwright MCP), `prompts.mjs`
  (slot fills only), `schema.mjs` (action vocabulary + flaw types), platform
  stategraph identity/controls, plus verbatim platform files (mcp.mjs,
  interactions.mjs, report.mjs, rubric.md). Same entrypoint names in both:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tej-Sharma/selfloop](https://github.com/Tej-Sharma/selfloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
