---
trigger: always_on
description: This file is loaded into every Claude Code session that opens this repo. Read it first; everything else in the project follows from here.
---

# PaperHub — orientation for Claude Code

This file is loaded into every Claude Code session that opens this repo. Read it first; everything else in the project follows from here.

## What you're working on

PaperHub is a paper-aware chat client with multi-agent tool-routing, an agentic SQLite section-navigation knowledge base (no vector store, v2.27), an in-repo slide pipeline, and a Citation Canvas so every cited chunk traces back to source. It is decomposed from two reference projects (`paper2slides-plus`, `Intro2GenAI-hw1`) — useful utilities are copied + adapted, not run as services.

**Authoritative spec:** [docs/superpowers/specs/2026-05-17-paperhub-srs.md](docs/superpowers/specs/2026-05-17-paperhub-srs.md) (**v2.39.0** latest spec; shipped through **v2.39.0** — **Router cleanup + deterministic multi-intent priority (Plan G2)** — removed the vestigial LLM-emitted `model_tier` (the model is config-driven by `intent`; nothing read it) and added a multi-intent **priority tie-break** (`memory > slides > paper_qa > library_stats > paper_search > paper_suggest > chitchat > clarify`, after the 0-refs / slide-question overrides); **benchmark-gated on the DEPLOY router model** (`gemini-3.1-flash-lite`, NOT a stronger sweep model — the live `:8000` gate caught a lean variant misrouting where a wrong-model sweep had passed), shipping `router/v2` (=`v3c`) at 4/4 multi-intent + zero regression + ~15% leaner than the old prompt (v2.39.0); **Agent Prompt Evaluation (Plan G1)** — a fully-isolated, internal cascade per-stage prompt-eval engine under `benchmark/agent/` (JSONL experiment store + browsable YAML prompt variants + auto-Batch-API/concurrent executor + deterministic/scalar/pairwise judge grading + golden-output propagation + config-driven variants × test-set-bucket sweep), with **zero deploy-code impact** — proven on the router (a concise, un-patched v2 rewrite that fixed a real mis-route at ~25% fewer tokens with zero regression) (v2.38.0); **slide source grounding + manual & structured-citation editing** (per-slide `% cite:` grounding → Sources strip → Citation Canvas; content-only LaTeX frame/whole-deck editors with keep-last-good-on-failure recompile; a deterministic per-slide reference editor; whole-section highlight) + **slide deck-length fixes** (LLM length extraction, configurable default, degenerate-output gate) + **LLM model fallback** (v2.36); **new-session paper attach** (lazy-create the backend session so a fresh chat can take its first paper) + **attach processing feedback** (v2.35); **SQL Agent → intelligent ReAct agent** with curated `library_stats` attachable cards (E1) (v2.34); **Plan F6.1** slide narrative planning + the PaperDigest/targeted-read gather rework + always-on streaming (v2.33); **Plan G** — UI i18n across 8 namespaces × 4 locales + account menu + a DB-backed runtime Settings panel (v2.31)). 
Any architecture / schema / scope question is answered there before code. 
The two-layer schema (`paper_content` for unique papers, `papers` for per-session membership) and the deferred slide-rendering framework choice are the two most load-bearing decisions to keep in mind. The full v2.4-v2.33 feature history and rationale live in the SRS Revision History; read it there for any deeper why-does-X question rather than duplicating it here.

## Implementation plan

The SRS is decomposed into 7 sequential plans, each producing working/testable software:

| Plan | Status | Document |
| --- | --- | --- |
| A — Backend foundation + Router-only chat | **complete** | [2026-05-17-paperhub-A-backend-foundation.md](docs/superpowers/plans/2026-05-17-paperhub-A-backend-foundation.md) |
| B — Frontend foundation | **complete** | [2026-05-18-paperhub-B-frontend-foundation.md](docs/superpowers/plans/2026-05-18-paperhub-B-frontend-foundation.md) |
| C — Paper Pipeline + Research Agent | **complete** | [2026-05-18-paperhub-C-paper-pipeline-research-agent.md](docs/superpowers/plans/2026-05-18-paperhub-C-paper-pipeline-research-agent.md) |
| D — Search results + Reference Sources + Citation Canvas | **complete** | [2026-05-21-paperhub-D-citation-canvas.md](docs/superpowers/plans/2026-05-21-paperhub-D-citation-canvas.md) |
| E — SQL Agent + sqlite MCP + session/global memory governance | **complete** | [2026-05-22-paperhub-E-library-intelligence.md](docs/superpowers/plans/2026-05-22-paperhub-E-library-intelligence.md) |
| F — Slide Pipeline + Report Agent | **complete** | [F1](docs/superpowers/plans/2026-05-23-paperhub-F1-slide-generation-viewing.md) · [F2.1](docs/superpowers/plans/2026-05-24-paperhub-F2.1-async-marker-upgrade.md) · [F4](docs/superpowers/plans/2026-05-25-paperhub-F4-slide-decoupling-editing.md) · [F4.2](docs/superpowers/plans/2026-05-27-paperhub-F4.2-slide-style-customization.md) · [F4.3](docs/superpowers/plans/2026-05-29-paperhub-F4.3-non-arxiv-pdf-ingestion.md) · [F5](docs/superpowers/plans/2026-06-05-paperhub-F5-presentation-voice.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whats2000/PaperHub](https://github.com/whats2000/PaperHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
