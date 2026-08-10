---
trigger: always_on
description: > Durable onboarding for Claude Code and other MCP-capable agents. **Code wins over docs.**
---

# CLAUDE.md — Vocello (QwenVoice)

> Durable onboarding for Claude Code and other MCP-capable agents. **Code wins over docs.**
> Scripts and machine-readable contracts are the gates; optional MCP tools and skills never are.
> When scope, platform, or gate expectations are unclear, **ask before editing**.
>
> **Plans and progress:** [`docs/ROADMAP.md`](docs/ROADMAP.md) · **Active narrative:** [`docs/development-progress.md`](docs/development-progress.md) · **Project map:** [`docs/project-map.html`](docs/project-map.html) · **Architecture:** [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) · **Domain rules:** [`.claude/rules/`](.claude/rules/)

## What this is

**Vocello** (`Vocello` repo, formerly `QwenVoice`; the local checkout directory and Xcode project keep the old name): local-first TTS on Apple Silicon — **Qwen3-TTS + MLX**, Swift 6, macOS/iOS 26+. No bundled weights; models download from Hugging Face. Also ships the `vocello` CLI, `scripts/`, benchmarks, and `website/`.

macOS **2.4.0** released (2026-08-01); iOS build 23 (v2.4.0) is the current TestFlight build (submitted to beta review with distribution to both groups) — internal group live, and the
**public beta link is live** (approved 2026-07-28): `https://testflight.apple.com/join/Cvp6yCv7`,
linked from the README, the repo description, and the website.
Releases are cut only on an explicit maintainer call — landed roadmap work never implies one.
Minimum support is Apple Silicon Mac with 8 GB or iPhone 15 Pro or newer. Canonical benchmark
evidence uses Mac mini M2 8 GB and iPhone 17 Pro; support and evidence hardware are not synonyms.

## Source of truth

`Sources/` → `project.yml` → machine-readable `config/` contracts → `scripts/` →
`.github/workflows/` → `CLAUDE.md` → other prose.

Model/speaker schema: [`Sources/Resources/qwenvoice_contract.json`](Sources/Resources/qwenvoice_contract.json).
The cross-platform artifact catalog
[`Sources/Resources/qwenvoice_production_model_catalog.json`](Sources/Resources/qwenvoice_production_model_catalog.json)
is complete for all six Speed/Quality artifacts and is the fail-closed macOS/CLI/iOS download
source. Static completeness does not substitute for explicit post-change live delivery evidence.
**If code or a machine-readable contract invalidates a doc, update the doc in the same change.**

## Before you edit

1. **Resume active work** — run `python3 scripts/roadmap.py status` for plan and item state, then read
   [`docs/development-progress.md`](docs/development-progress.md) for the narrative and confirm its
   checkpoint against the current checkout.
2. **Read the domain rule** — before working in a domain, read the matching file under [`.claude/rules/`](.claude/rules/) (see the routing table below).
3. **Inspect capabilities** — MCP servers and skills are user-scoped; verify a tool is currently callable before relying on it, and read every selected skill before use. Optional assists never substitute for the deterministic script gates.
4. **Minimal diff** — no drive-by refactors; preserve module boundaries and stable `accessibilityIdentifier` values.
5. **Close the loop in the same change** — a substantive arc lands together with its evidence and
   its doc updates, including narrative docs (`docs/development-progress.md`, the matching ADR or
   status report). The derived-catalogs hard rule enforces this for generated docs; this norm
   extends it to prose.
6. **Currency pass after dense workstreams** — close a multi-commit workstream with a
   `docs: currency pass` commit that re-syncs narrative prose with the tree before moving on.
7. **Ask** when the target platform or test scope is ambiguous. Commit/push policy is not
   ambiguous: deterministic verification is sufficient to preserve and share development work.

## Hard rules

| Rule | Detail / verify |
| --- | --- |
| **iOS runtime/UI = physical device + XCUITest** | Never use Simulator. XCUITest drives the paired physical iPhone; scripts provide deterministic device/telemetry proof. The generic physical-device SDK compile builds the app and standalone iOS policy-test bundle without a phone, but the selected Xcode must expose matching iOS Platform Support/runtime availability for `generic/platform=iOS`. `scripts/lib/ios_platform_preflight.py check` validates that host component before build setup and never downloads, boots, or executes a Simulator. Xcode 26 cannot execute the app-host-free tool-hosted bundle on a physical-device destination, so it remains compile-only; `gate` remains a physical-device runtime diagnostic, not a UI-result gate. |
| **`project.yml`, not pbxproj** | After edit: `./scripts/regenerate_project.sh` + `./scripts/check_project_inputs.sh`. iOS resources: `sources:` + `buildPhase: resources` (not `resources:`). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PowerBeef/Vocello](https://github.com/PowerBeef/Vocello) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
