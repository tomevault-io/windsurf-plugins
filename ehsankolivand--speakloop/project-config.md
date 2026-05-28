---
trigger: always_on
description: <!-- SPECKIT START -->
---

<!-- SPECKIT START -->
Active feature: 006-feedback-quality-reliability — make the existing AI-derived
  feedback (grammar suggestions, cross-attempt narrative, single top-priority)
  reliably higher-quality. Same model family (Qwen3-8B); report format & schema_version
  stay 1; fully offline; no new feedback dimension (no ideal-answer judging). Grammar
  call site adopts vendor non-thinking config (temp 0.7) + repetition_penalty/stop +
  json-repair recovery (one new dep); narrative & top-priority STAY deterministic.
  Decisions: (1) single LLM call site, thinking off — no dual-mode; (2) stay 4-bit;
  8-bit OUT OF SCOPE this sprint (no A/B, no threshold; Constitution VI/VII); (3)
  json-repair now, Outlines deferred (its rep-penalty gap is RESOLVED — see research.md). New: eval/grammar/ held-out set +
  offline harness (baseline→post) proves SC-001/SC-002. Iterative: US1 reliability (MVP)
  → US2 grammar accuracy → US3 narrative + top-priority grounding.

Plan: specs/006-feedback-quality-reliability/plan.md
Spec: specs/006-feedback-quality-reliability/spec.md
Research: specs/006-feedback-quality-reliability/research.md (lifts doc/QWEN_IMPROVMENT_RESEARCH.md)
Data model: specs/006-feedback-quality-reliability/data-model.md
Contracts: specs/006-feedback-quality-reliability/contracts/ (grammar-output-schema · eval-set-format · report-invariance)
Code touchpoints: llm/qwen_engine.py (sampler + rep-penalty + stop), feedback/grammar_analyzer.py
  (json-repair + bounded regenerate; keep verbatim/coherence/no-op verification),
  feedback/narrative.py (tighten deterministic grounding); + eval/grammar/; pyproject.toml (+json-repair).

Prior feature: 005-context-engineering-audit — audited & rewrote the CLAUDE.md layer
  (root + 13 module files) as a code-true deliverable; launch footprint ≤ 6000 tokens.
  Plan: specs/005-context-engineering-audit/plan.md · Spec: specs/005-context-engineering-audit/spec.md

Prior feature: 004-public-release-readiness — cloneable & runnable by a stranger.
  Default questions ship in-repo at `content/questions.yaml`; `~/.speakloop/qa.yaml`
  is an opt-in override (precedence: --qa-file → home override → repo default).
  Adds a stdlib+git path-portability audit (pytest, < 2 s). No new dependency;
  report schema_version stays 1; MIT LICENSE present.
  Plan: specs/004-public-release-readiness/plan.md · Spec: specs/004-public-release-readiness/spec.md

Prior feature: 003-asr-l2-accent-accuracy — faithful transcripts on Persian-L1
  accented technical English. Default ASR Whisper-large-v3-turbo (mlx-whisper),
  Parakeet-TDT via `--asr-engine parakeet` + automatic fallback; per-session domain
  biasing + Silero-VAD; additive `asr:` frontmatter key (schema_version stays 1).
  Plan: specs/003-asr-l2-accent-accuracy/plan.md · Spec: specs/003-asr-l2-accent-accuracy/spec.md

Prior feature: 002-post-session-debrief — educational LLM grammar feedback
  (Persian-L1 catalog) + in-terminal interactive debrief.
  Plan: specs/002-post-session-debrief/plan.md · Spec: specs/002-post-session-debrief/spec.md
  New module: src/speakloop/debrief/ (render + audio + menu).

Base feature: speakloop v1 — local English interview-practice CLI.
  Plan: specs/001-v1-product-spec/plan.md · Spec: specs/001-v1-product-spec/spec.md

Engine selections cite the in-repo research documents:
  doc/research_tts.md (Kokoro-82M),
  doc/research_asr.md (Parakeet-TDT-0.6b-v3),
  doc/research_llm.md (Qwen3-8B 4-bit — deviates from the initial Qwen3.5-9B
    research choice because that HF repo turned out to be a VLM incompatible
    with mlx_lm.load(); see installer/manifest.py rationale comment).

Constitution: .specify/memory/constitution.md (v1.0.0).
Shipping order is three phases (A: listen-only, B: attempts + metrics, C: LLM feedback + trends);
each phase is a complete working system per Principle XII.
<!-- SPECKIT END -->

<!-- Human-authored map below. Anatomy (FR-010): overview · tech-stack · layout ·
commands · conventions · maintenance · traps · never-do · pointers. Code is the
source of truth; the constitution wins on any documentation conflict. -->

# speakloop — top-level map

## Overview

speakloop is a fully local, offline English speaking-practice CLI for non-native
software engineers preparing for technical interviews. It runs three local AI models
on Apple Silicon — TTS (Kokoro-82M), ASR (Whisper-large-v3-turbo), LLM (Qwen3-8B
4-bit) — to drive a listen → attempt (4/3/2) → feedback loop, written as Obsidian-
compatible Markdown reports. After the initial model download it makes zero network
calls (Constitution Principles II, III).

## Tech stack

Derived from `pyproject.toml`, confirmed against actual imports.

- **Python 3.12** — pinned `requires-python = ">=3.12,<3.13"` (`pyproject.toml:7`).
- **`uv`** — the only package manager (no `pip` workflows). Run everything via `uv run`.
- **CLI / UI**: `typer` (≥0.12), `rich` (≥13.7) — CLI only, no GUI (constitution constraint).
- **Config / data**: `pyyaml` (≥6.0, YAML user config), `python-frontmatter` (report YAML), `numpy` (≥1.26).
- **Audio**: `sounddevice` (≥0.4), `soundfile` (≥0.12).
- **Models / download**: `huggingface_hub` (≥0.24, resumable).
- **Engine packages** (each imported function-local in exactly ONE wrapper — Principle V):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ehsankolivand/speakloop](https://github.com/ehsankolivand/speakloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
