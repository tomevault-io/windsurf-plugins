---
trigger: always_on
description: macOS menu bar voice input tool with dual-engine local ASR, multi-provider cloud ASR, and LLM post-processing.
---

# Type4Me — Development Guide

## Overview

macOS menu bar voice input tool with dual-engine local ASR, multi-provider cloud ASR, and LLM post-processing.
Local ASR: SenseVoice via native sherpa-onnx (streaming) + Qwen3-ASR (final calibration, Python WebSocket service managed by `SenseVoiceServerManager`).
Cloud ASR: 15 providers implemented (Volcano, StepFun streaming, StepFun batch, MiMo batch, OpenAI, Deepgram, Cartesia, AssemblyAI, ElevenLabs, Gemini, Grok, Soniox, Bailian, Baidu, Meta Muse), plus Apple Speech.
Swift Package Manager project, no Xcode project file. Optional `sherpa-onnx.xcframework` enables local SenseVoice, Silero VAD, and punctuation restoration.

## Branch Naming and Lifecycle

Use one short-lived branch for one independently reviewable change. Branches must
use lowercase ASCII kebab-case and follow this form:

```
<type>/<issue-number-optional>-<concise-description>
```

Examples:

```
feat/241-cartesia-streaming-asr
fix/247-agent-context-cleanup
docs/intelli-sense-pangu-terminology
perf/reduce-startup-latency
refactor/session-output-pipeline
test/recognition-session-recovery
chore/update-swift-format
release/2.1.1
hotfix/251-crash-on-launch
```

### Allowed types

| Type | Use for |
| --- | --- |
| `feat` | A user-visible capability or substantial new behavior. |
| `fix` | A defect fix. Prefix the issue number when one exists. |
| `docs` | Documentation-only changes. |
| `perf` | A measurable performance improvement without a primary behavior change. |
| `refactor` | Internal restructuring with no intended behavior change. |
| `test` | Test-only additions or repairs. |
| `chore` | Tooling, repository maintenance, or non-product housekeeping. |
| `release` | Release preparation and versioning. |
| `hotfix` | An urgent production fix. |

### Rules

- Do not use agent, tool, model, or personal prefixes such as `agent/`,
  `codex/`, `builder/`, or a username. These are execution details, not work
  categories.
- Do not create a branch named only after a PR number, and do not combine
  unrelated changes on one branch.
- Keep descriptions specific but compact: prefer `fix/247-agent-context-cleanup`
  over `fix/247-fix-the-problem-reported-in-the-agent-context-issue`.
- Use the same branch name locally and on its published review branch.
  `origin/main` is updated only with explicit user authorization.
- Before merging or opening a PR, rebase or otherwise bring the branch up to
  date with the current `origin/main` as appropriate for the requested
  workflow. Use `--force-with-lease`, never a blind force push, after a
  published branch is rebased.
- After the change is confirmed in `main`, delete its local and published
  remote branches unless there is an explicitly stated reason to retain them.
  Never delete an integration branch without explicit authorization.
- `main` is the integration branch. Work directly on it, or push directly to
  `origin/main`, only when the user explicitly asks for a direct change.

Existing long-lived feature branches are exceptions until they are merged or
explicitly retired; do not rename them merely to satisfy this convention.

## Build & Run

```bash
# Qwen3-ASR server setup (optional, Apple Silicon only)
cd qwen3-asr-server && python3.12 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt && cd ..

# Optional: local SenseVoice, Silero VAD, and punctuation module (~5 min, requires cmake)
bash scripts/build-sherpa.sh

swift build -c release
```

The built binary is at `.build/release/Type4Me`.

- “本地部署”“装一下”“给我体验”等请求默认授权按正式应用流程打包，备份后替换 `/Applications/Type4Me.app`。保留 `com.type4me.app`、登录钥匙串中的 Developer ID Application 签名（Team `T98LK79X2K`）及 designated requirement，完成构建、签名与安装完整性检查后交给用户体验。仅在用户明确要求 Dev 版时使用独立 Dev app。
- 本地 UI 迭代的测试和实际体验默认交给用户。除非用户明确要求，不自行运行测试套件、操作界面测试、切换设置或启动录音；仅完成必要的编译、签名、备份和安装完整性检查。用户提供的截图和体验反馈是下一轮调整依据。
- Use `$type4me-deploy` and the current production packaging flow; default to `pure` + universal. Explicit Dev-only requests use `scripts/dev-run.sh`.
- Local deployment does not authorize GitHub publication or the archived `official` subscription variant.

## Build Variants

Two supported product variants are built from the same codebase via conditional compilation flags. The compact CppJieba experiment is an independent optional capability:

| Variant | `HAS_SHERPA_ONNX` | `HAS_CLOUD_SUBSCRIPTION` | Arch | Description |
|---------|---|---|---|---|
| **pure** | no | no | universal | Open-source cloud edition (BYOK API keys) |
| **official** | no | yes | universal | Archived subscription edition; the build script intentionally rejects it. |
| **local** | yes | no | arm64 | Open-source local edition (bundled SenseVoice + Qwen3-ASR) |

`ENABLE_CPPJIEBA=1` keeps `CppJiebaBridge/marker` visible while running `build-dmg.sh`, which adds the C++ bridge and compact dictionary resources. It defaults to `0`, so normal release packages do not pay the binary/resource cost until the experiment is explicitly enabled. A compiled build also has the local `tf_cppJiebaExperimentEnabled` runtime switch; disabling it falls back to `NLTokenizer` without rebuilding.

### Subscription paused (2026-04)

We are not developing the subscription feature for the foreseeable future. The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joewongjc/type4me](https://github.com/joewongjc/type4me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
