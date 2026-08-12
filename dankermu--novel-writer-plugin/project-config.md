---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code plugin for Chinese web novel (网文) multi-agent collaborative writing. 6 specialized agents orchestrated through a state-machine workflow with spec-driven quality gates and anti-AI output strategies.

## Commands



CI runs on push via `.github/workflows/docs-ci.yml` (markdownlint + lychee + manifest validation).

## Python Environment

All `scripts/*.sh` use a project-local venv (`.venv/`) for Python isolation. Setup:



Scripts auto-resolve `${SCRIPT_DIR}/../.venv/bin/python3`, falling back to system `python3` if venv is absent. Currently stdlib-only (no third-party deps).

## Architecture

### 4-Layer Spec Hierarchy (Spec-Driven Writing)

| Layer | File Pattern | Purpose | Mutability |
|-------|-------------|---------|------------|
| L1 World Rules | `world/rules.json` | Physics, magic, society hard constraints | Immutable |
| L2 Character Contracts | `characters/active/*.json` | Ability bounds, behavior patterns | Protocol-gated |
| L3 Chapter Contracts | `volumes/vol-{V:02d}/chapter-contracts/` | Pre/post conditions, acceptance criteria | Negotiable w/ audit |
| LS Storyline Specs | `storylines/storylines.json` | Multi-POV constraints, prevents cross-line leaks | Volume-scoped |

Agents validate outputs against these layers. QualityJudge performs dual-track verification: contract compliance (hard gate) + 9-dimension scoring (soft eval, including tonal_variance for register micro-injection density). ContentCritic evaluates reader engagement (Track 3) + content substance (Track 4: information density, plot progression, dialogue efficiency) + POV knowledge boundary & cross-storyline leak detection (Track 5: detects proper nouns/info leaking into narration that the POV character shouldn't know yet, and cross-storyline entity leaks in non-convergence chapters) + cross-chapter logic review (Track 6: reads recent 3 chapters full text to detect hard contradictions and plot holes; narrative techniques like time skips/POV switches are not flagged).

### State Machine



State persists in `.checkpoint.json` with fields: `orchestrator_state`, `current_volume`, `last_completed_chapter`, `pipeline_stage`, `inflight_chapter`, `eval_backend` ("codex" or "opus", default "codex" — `/novel:start` 初始化时自动写入).

### Single-Chapter Pipeline

`API Writer(draft, fallback CW) → StyleRefiner(de-AI polish) → [QualityJudge + ContentCritic parallel] → Gate Decision (merge) → Summarizer (pass/polish only)`

API Writer (`scripts/api-writer.py`) calls external model API (default: gemini-3.1-pro-preview) with pure creative system prompt (`prompts/api-writer-system.md`), bypassing Claude Code's engineering-focused system prompt. Falls back to ChapterWriter agent on API failure. CW agent remains available for revision/polish passes (targeted edits benefit from Claude Code's tool integration).

Gate thresholds: ≥4.0 pass, 3.5–3.9 polish, 3.0–3.4 revise, 2.0–2.9 review, <2.0 rewrite. ContentCritic Track 4 substance violation (any dimension < 3.0) forces revise. Track 6 logic_review severity=high forces revise. QJ tonal_variance < 3.0 forces revise.

**Revision loop optimization** (M9.2): revise triggers a tiered sub-pipeline based on `revision_scope`:
- `trivial` (≤1 failed_dimension, no failed_tracks, overall ≥ 3.5, excludes structural dims plot_logic/storyline_coherence/tonal_variance and golden-chapter engagement override): `CW(targeted) → SR(lite) → force_passed` (~15-20K tokens, skip QJ/CC recheck)
- `targeted` (no high_violation, no substance_severe, overall ≥ 3.0, not trivial): `CW(targeted) → SR(lite) → [QJ(recheck) ∥ CC(recheck)]` (~35-45K tokens)
- `full` (has high_violation or substance_severe or overall < 3.0): `CW → SR → [QJ+CC]` re-run (~90K tokens)

Targeted mode passes `failed_dimensions` to CW for scoped edits, uses `lite_mode`/`recheck_mode` flags for downstream agents. QJ/CC have no cross-dependency in recheck mode, enabling parallel dispatch after SR. Summarizer runs only after gate pass, not during revision loops. Trivial: 1 round, force_passed (no recheck needed). Targeted: 1 round, then direct-fix Task agent (SR only, no re-eval) + force_passed. Full: max 2 rounds, then force_passed or pause_for_user.

**Eval backend** (M10, v3.0.0): Summarizer/QJ/CC/sliding-window can run via Codex or Opus agents. New projects default to `eval_backend: "codex"` in checkpoint. Config is global per project, no runtime fallback between backends.

Codex path: `codex-eval.py --agent` (assemble task content from manifest) → `codeagent-wrapper --backend codex` (Codex execution) → `codex-eval.py --validate` (staging output validation). Codex prompts live in `prompts/codex-{agent}.md`, adapted from agent specs without YAML frontmatter or Claude Code tool refs. Writing pipeline (API Writer/CW/SR) is unaffected.

**codeagent-wrapper constraints**: Do not kill long-running processes (wastes API cost). Timeouts: Summarizer/QJ/CC 3600s, sliding-window 7200s (via `CODEX_TIMEOUT` env). SESSION_ID from wrapper logged for audit/resume.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DankerMu/novel-writer-plugin](https://github.com/DankerMu/novel-writer-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
