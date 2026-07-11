---
trigger: always_on
description: Kindalive is a robot emotion system that models emotions through simulated neurochemistry. The owner types a paragraph of natural language ("you won the lottery", "Friday, finances up, day off tomorrow"); an LLM interpreter translates that paragraph to chemical impulses, which drive a neurochemical engine. Emotions are computed projections of the chemical state — never stored directly.
---

# CLAUDE.md — Instructions for Working on Kindalive

## What This Project Is

Kindalive is a robot emotion system that models emotions through simulated neurochemistry. The owner types a paragraph of natural language ("you won the lottery", "Friday, finances up, day off tomorrow"); an LLM interpreter translates that paragraph to chemical impulses, which drive a neurochemical engine. Emotions are computed projections of the chemical state — never stored directly.

## Architecture (Read This First)

Before making any changes, read `docs/architecture.md`. It is the **source of truth** for the entire system. All other docs derive from it.

Key architectural rules that must not be violated:
- Emotions are **computed, never stored**. They are pure functions of `ChemicalState`.
- All chemicals are clamped to `[0.0, 1.0]`. All emotions are clamped to `[0.0, 1.0]`.
- The decay formula is `level += (baseline - level) * (1 - 2^(-dt / half_life))` — true half-life. Do NOT use `e^(-dt/half_life)`.
- The engine sub-steps any `dt > 0.5s` into 0.5s increments for numerical stability. Clamping happens after each sub-step.
- The LLM interpreter returns structured JSON: an object `{"reply": "<spoken line>", "impulses": [ ...impulse dicts... ]}` (a bare impulse array is still accepted for back-compat via `_split_reply_and_impulses`). The parser (`_extract_json_payload` + `_sanitize_json_quirks` in `interpreter/llm_interpreter.py`) tolerates markdown code fences, prose preambles, leading `+` on positive numbers, and trailing commas — but must ultimately yield valid JSON.
- `SeedChemistry` configures a robot's baseline. Personality presets are sugar over `SeedChemistry`.

## Documentation

| File | Role |
|------|------|
| `docs/architecture.md` | Source of truth. System design, formulas, data types, project structure. |
| `docs/web-ui.md` | The only UI — NiceGUI dashboard around the LED dot-matrix face. Layout, features, LLM setup, `.env` auto-loading. |
| `docs/testing-strategy.md` | Test layers with executable code examples. Build order. |
| `docs/llm-benchmark.md` | Scenarios for LLM interpretation quality (pass/fail scorecard). |

## Build Order

Follow the build order in `docs/testing-strategy.md` strictly. Each step is designed so it can be fully tested before the next step begins. Do not skip ahead.

Current status — feature-complete through the face pivot and packaged for open-source release:

- **Input**: a freeform text paragraph through `Robot.interpret_text(...)` or the web UI textarea. The old data fetchers, event batching, scenario generator, and TUIs are gone; `UserText`/`RealtimeRouter` are the only input/router names (no legacy aliases).
- **Face**: 12-muscle `FaceState` projection rendered as a retro LED dot-matrix panel on a 2D canvas (`web_assets/face3d.js` + `face_3d.py`). The full chain — text → LLM → impulses → `ChemicalState` → `FaceState` → renderer payload — is locked by `test_full_pipeline.py::test_text_to_face_payload_chain`.
- **Web UI**: mobile-responsive, installable PWA (manifest + icon in `web_assets/`); `Dockerfile` + `KINDALIVE_HOST`/`$PORT` for always-on hosting. Header Reset rebuilds the robot with a slightly-randomized chemical state (±`RESET_JITTER`=0.22 per chemical, also applied at server start) and a fresh conversation.
- **Voice**: the LLM returns `{"reply", "impulses"}`; `reply` surfaces as `Robot.last_reply`, shows as a 🗣 bubble, is spoken via the browser Web Speech API (mood-mapped rate/pitch, 🔊 toggle, iOS gesture-priming), and drives a lip-sync mouth flap (`setSpeaking`/`mouthPulse`). A bare impulse array is still accepted for back-compat.
- **Memory**: `Robot.conversation` (bounded to `MAX_CONVERSATION_MESSAGES=40`) is fed to the LLM as multi-turn chat via `RobotContext.history`; the paragraph cache is bypassed once a thread is underway.
- **Backends**: `AnthropicBackend` (Claude) and `OpenAICompatBackend` (Ollama/LM Studio/vLLM/OpenAI/OpenRouter).
- **Packaging**: the core (`engine/`, `emotions/`, `expression/face.py`, `Robot`) has **zero third-party dependencies**; NiceGUI and the LLM libraries live in extras (`[web]`, `[anthropic]`, `[openai]`, `[all]`, `[dev]`). MIT licensed. CI (`.github/workflows/ci.yml`) runs tests + the >=80% coverage gate on Python 3.10–3.13, a core-only run on 3.9 (NiceGUI needs 3.10+), and `mypy --strict` — all three must stay green.
- **Hardware**: `examples/` maps `FaceState` to a MAX7219 LED matrix and PCA9685 servos, with terminal fallbacks when the driver libraries aren't installed.

~181 tests passing. Primary UI: `python3 -m kindalive.expression.web_ui`. CLI: `python3 -m kindalive.main --text "..."`.

## Code Conventions

- **Python 3.9+**, asyncio for all I/O
- **Type hints everywhere** — the codebase should pass `mypy --strict`
- **Enum values are lowercase** strings (`Chemical.DOPAMINE.value == "dopamine"`), enum names are uppercase (`Chemical.DOPAMINE`)
- `Chemical.from_string()` is case-insensitive

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smithandrewjohn/kindalive](https://github.com/smithandrewjohn/kindalive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
