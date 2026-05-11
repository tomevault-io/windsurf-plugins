---
trigger: always_on
description: > **Audience:** All AI coding agents (Claude Code, Cursor, Copilot, etc.)
---

# LifeOS — Agent Reference

> **Audience:** All AI coding agents (Claude Code, Cursor, Copilot, etc.)
> **Status:** Complete
> **Last Updated:** 2026-03-04

LifeOS is a self-hosted AI assistant that indexes personal data (notes, emails, messages, photos, financial data) for semantic search, synthesis, and proactive intelligence. Runs on Linux or macOS. Optionally, a Mac can act as an Apple Data Agent for iMessage, phone calls, and contacts.

---

## Key Concepts

- **Two-tier data model**: SourceEntity (raw observations from each data source) → PersonEntity (canonical, merged records per person). See [ADR-003](docs/adr/003-two-tier-data-model.md).
- **Hybrid search**: Vector similarity (ChromaDB) + keyword matching (BM25/FTS5), fused via Reciprocal Rank Fusion. See [ADR-004](docs/adr/004-hybrid-search.md).
- **Entity resolution**: Links emails, phones, and names across sources to canonical people using fuzzy matching with scoring.
- **Sync phases**: Seven-phase nightly pipeline — Collection → Entity Processing → Relationship Building → Indexing → Content Sync → Entity Cleanup → Consistency Verification.
- **Agentic chat**: Local LLM autonomously calls 15 tools (search, calendar, email, tasks, etc.) across multiple rounds to answer queries.

## Tech Stack

| Component | Technology |
|-----------|------------|
| Backend | FastAPI (port 8000) |
| Vector DB | ChromaDB (port 8001) |
| Keyword Search | SQLite FTS5 (BM25) |
| Query Router | Ollama + Qwen 2.5 (local) |
| LLM (orchestration + synthesis) | Local model via OpenAI-compatible API, or Claude API (`LIFEOS_LLM_BACKEND`) |
| LLM Client | `api/services/llm_client.py` — unified wrapper with Anthropic↔OpenAI tool format translation |
| Embeddings | sentence-transformers (GPU via ROCm/CUDA) |
| Frontend | Vanilla HTML/JS (no build step) |
| Job Queue | SQLite-backed background workers |
| Reminders | SQLite + cron scheduler |
| Service Management | systemd (Linux) / launchd (macOS) |

## Documentation Structure

| Category | Path | Purpose |
|----------|------|---------|
| **WHY** (Decisions) | `docs/adr/` | Immutable architecture decision records |
| **WHAT** (Product) | `docs/specs/product/` | Consumer-facing specifications |
| **HOW** (Design) | `docs/specs/technical/` | Engineering specifications |
| **HOW** (Standards) | `docs/specs/standards/` | Coding and testing conventions |
| **HOW** (Operations) | `docs/guides/` | Setup, config, and operational guides |
| **WHEN** (Plans) | `docs/plans/` | Ephemeral roadmap and backlog |
| **WHY** (Vision) | `docs/vision/` | Project philosophy and guiding principles |
| **History** | `docs/archive/` | Superseded documents |

### Navigation — "What question → which doc"

| Question | Document |
|----------|----------|
| How is data modeled? | [specs/product/data-model.md](docs/specs/product/data-model.md) |
| What API endpoints exist? | [specs/product/api-reference.md](docs/specs/product/api-reference.md) |
| How does the sync pipeline work? | [specs/technical/data-and-sync.md](docs/specs/technical/data-and-sync.md) |
| What does the code structure look like? | [specs/technical/architecture.md](docs/specs/technical/architecture.md) |
| How does hybrid search work internally? | [specs/technical/search-indexing.md](docs/specs/technical/search-indexing.md) |
| How is perf traced and monitored? | [specs/technical/observability.md](docs/specs/technical/observability.md) |
| How do I set up the project? | [guides/installation.md](docs/guides/installation.md) |
| What scripts are available? | [guides/scripts.md](docs/guides/scripts.md) |
| Why does LifeOS exist? What guides decisions? | [vision/philosophy.md](docs/vision/philosophy.md) |
| Why was X chosen over Y? | `docs/adr/` (specific ADR) |
| How do we review PRs? | `/review-pr`, `/pr-check`, `/implement` skills in `.claude/skills/` |

---

## Development Principles

These apply to all agents. Bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Acting

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.
- Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nbramia/LifeOS](https://github.com/nbramia/LifeOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
