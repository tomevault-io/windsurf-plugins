---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🚨 HARD RULES — read before touching anything 🚨

These are non-negotiable. Violating them has burned the user before.

### 1. NEVER downgrade external models, APIs, libraries, or capabilities
Your training cutoff is older than the user's reality. The user is a
working developer with access to the latest releases — Gemini 3.5,
Claude 4.x, GPT-5 variants, whatever's actually current. If a model
name, API endpoint, library version, or product feature looks unfamiliar
or "doesn't exist," **assume YOUR knowledge is stale, not theirs**.

Concrete rules:
- **Do NOT remove model entries** from catalogs (e.g. `GEMINI_MODELS`,
  Claude/OpenAI/Grok caps maps in `backend/assistant_routes.py`)
  because you don't recognize them.
- **Do NOT pin libraries down** to versions you "know" exist when a
  newer one is in the lockfile.
- **Do NOT replace a "preview" / "experimental" / "-latest" model id**
  with a stable one you remember from training.
- **If you genuinely need to update a model list**, fetch the source
  of truth FIRST (WebFetch on `https://ai.google.dev/gemini-api/docs/models`,
  `https://docs.anthropic.com/en/docs/about-claude/models`,
  `https://platform.openai.com/docs/models`, etc.) — never write from
  memory. Then if you're proposing a downgrade, ASK the user first and
  let them confirm.

If you accidentally do downgrade, immediately fetch the docs and
restore the full catalog.

### 2. NEVER allow ruff version drift
Exactly ONE ruff version exists in this repo's tooling chain at all
times. It's pinned in `pyproject.toml` (`dependency-groups.dev`) AND
`.github/workflows/lint.yml` (the `RUFF_VERSION` env var) AND used via
`uv run ruff …` so the project venv's ruff is what runs. Symptoms of a
violation: `ruff format --check` complains about reformatting files
that were clean last commit, with no semantic edits in between.

Concrete rules:
- **Never `pip install ruff` or `pipx install ruff`** globally without
  matching the pinned version exactly.
- **Never edit only one of the two pin sites** — always update both in
  the same commit.
- **Before committing**, run `uv run ruff check .` AND
  `uv run ruff format --check .` from the repo root. Both must pass.
- **If `ruff format` drifts** after a session where nothing semantic
  changed, the FIRST suspect is a version mismatch — investigate
  before you "fix" the drift.

### 3. Form controls MUST have real labels and valid ARIA
Every form/control change must include an accessibility check before it is
considered done.

Concrete rules:
- Native fields (`input`, `select`, `textarea`) must have stable `id` and
  `name` values, plus either `<label htmlFor="that-id">` or a valid wrapping
  `<label>`.
- Custom controls (`div role="slider"`, button-based selects/dropdowns,
  canvas/WebGL pickers, etc.) must **not** be wrapped in `<label>` because
  browsers do not associate that label with non-native controls.
- Custom controls must use `aria-label` or `aria-labelledby`; button dropdowns
  must also expose state/relationship where applicable (`aria-expanded`,
  `aria-haspopup`, `aria-controls`, `role="listbox"` / `role="option"`).
- When touching form UI, sweep nearby code for `<label` and custom controls so
  existing invalid patterns are fixed, not copied.
- Never silence, suppress, or ignore accessibility warnings; fix the DOM
  relationship that caused the warning.

See the `## Ruff Configuration` section below for more detail.

## Project Overview

Stable Audio 3 is a text-conditioned audio generation system. It generates audio from text prompts using a two-stage architecture: a DiT (diffusion transformer) generates latents, then the SAME autoencoder decodes them to 44.1kHz stereo audio.

## Commands

```bash
# Install dependencies
uv sync --group dev

# Launch the app (Windows: bootstraps deps on first run, then runs backend + frontend in ONE console)
.\theDAW.bat

# Or launch the two dev servers manually (any OS)
uv run uvicorn backend.server:app --host 0.0.0.0 --port 8600 --reload   # backend  -> :8600
cd frontend && npm run dev                                             # frontend -> :5173

# Run tests (requires model weights downloaded)
uv run pytest

# Run single test file
uv run pytest tests/test_inference.py

# Run tests and save generated audio for inspection
uv run pytest --save-audio

# Lint (runs on CI for PRs)
uv run ruff check
uv run ruff format --check
```

## Architecture

### Two-Stage Pipeline

1. **SAME Autoencoder** (`models/autoencoders.py`) — Compresses 44.1kHz stereo audio to 256-dim continuous latents at 4096x downsampling. Two variants: SAME-S (266M, CPU-capable, chunked attention) and SAME-L (1.7B, GPU-required, sliding window attention).

2. **DiT** (`models/dit.py` → `models/transformer.py`) — Conditional diffusion transformer that generates SAME latents. Uses T5Gemma text conditioning, duration embeddings, and optional inpainting inputs. Three sizes: Small (433M), Medium (1.4B), Large (2.7B, API-only).

### Key Files

- `pipeline.py` — Public API. `StableAudioPipeline` and `AutoencoderPipeline` classes. All inference flows go through `generate()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gantasmo/theDAW](https://github.com/gantasmo/theDAW) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
