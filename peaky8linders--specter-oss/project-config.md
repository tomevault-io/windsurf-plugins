---
trigger: always_on
description: > Project-local guidance for Claude Code. Cross-project rules live in `~/.claude/CLAUDE.md`.
---

# CLAUDE.md — specter-oss

> Project-local guidance for Claude Code. Cross-project rules live in `~/.claude/CLAUDE.md`.

## What this project is

**Specter** is an EU AI Act compliance toolkit (`pip install specter`) that ships
two surfaces:

1. **Python library** — `specter` package: data-pure article/role/taxonomy
   catalogs, LLM-as-Judge reward-hack detector, three-agent adversarial verifier,
   FastAPI Q&A router with closed-world refusal + reference validation.
2. **Claude Code plugin** — local stdio MCP server + 6 slash commands wrapping
   the same library (`claude-plugin/`).

Frozen knowledge base: **113 articles + 13 annexes** of Regulation (EU) 2024/1689,
plus the 8-control LatticeFlow ATLAS catalog for Article 15.

## Layout

```
specter/
├── data/              article catalog (113 articles + 13 annexes), taxonomy,
│                      9-role obligation registry, Article 15 controls (8 LatticeFlow
│                      ATLAS controls), ontology mapping, rationalizations
├── judge/             LLM-as-Judge: ComplianceRewardHackDetector + three-agent verifier
├── llm/               provider abstraction — Anthropic Claude / OpenAI.
│                      Singleton per provider; lazy SDK init; soft-fail on auth.
├── qa/                grounded Q&A models, auth, retrievers per provider
│                      (claude_retriever / openai_retriever)
├── api/               FastAPI routers — POST /v1/eu-ai-act/ask + POST /v1/case
│                      + GET /v1/case/personas + dev_app.py mounting /webapp/
├── agents/            Suits-themed five-voice overlay
│                      (Harvey / Mike / Rachel / Louis / Jessica)
├── ontology/          RDF/Turtle OWL ontology (AIRO + DPV)
└── mcp_server.py      stdio MCP server for Claude Code

claude-plugin/         Claude Code plugin manifest, .mcp.json, commands/*.md
webapp/                Casebook SPA — vanilla ES2022, two-pane ChatGPT-style
                       layout, persistent case history (localStorage),
                       BYOK settings drawer (provider + key, never sent to a
                       backend other than the user's chosen LLM)
tests/                 pytest smoke tests pinning the public surface
```

## Hard rules (do not break)

- **Data-pure layers stay pure.** `specter/data/*` and `specter/qa/models.py`
  must remain deterministic for the same inputs — no I/O, no global mutable
  state. New side-effecting code goes in `specter/api/*`, `specter/agents/*`,
  or behind a small adapter.
- **Hallucinated articles never reach the wire.** Every external-facing
  reference passes through `reference_from_article_ref` which validates against
  `ARTICLE_EXISTENCE` and returns `None` on hallucination. Don't bypass it.
- **Closed-world refusal lives in the route layer.** The Q&A router replaces
  empty/low-confidence retriever output with a deterministic refusal string.
  Don't paper over it with cosmetic prose.
- **Public surface is locked at v0.1.x.** Additions OK; breaking changes need
  a version bump + CHANGELOG entry.

## Conventions

- Python ≥ 3.11. Type hints everywhere. Pydantic v2. `from __future__ import annotations`.
- Ruff line length 100, ruleset `E F I B UP` (see `pyproject.toml`).
- Tests: `pytest -v` (smoke tests in `tests/test_smoke.py`). New behaviour
  needs a smoke test pinning its contract.
- Comments explain the *why*, not the *what*. The existing codebase has long,
  carefully-worded section headers — match that voice when extending.

## Suits-themed agent layer (`specter/agents/`)

A small overlay that reframes a compliance question as a "case" worked by five
characters loosely inspired by the TV series *Suits* and the local-first OSS
fork of Will Chen's `mike` legal AI platform
([mikeOnBreeze/mike-oss](https://github.com/mikeOnBreeze/mike-oss)).

| Agent | Role | Backed by |
|---|---|---|
| **Harvey Specter** | Senior partner — project mascot. Brand face on the cover panel; not in dialogue turns by default. | n/a — `Voice.HARVEY` exists for completeness; `WORKING_VOICES` excludes him |
| **Mike Ross** | Photographic-memory associate. Recalls articles + prior cases. | `ARTICLE_EXISTENCE`, `articles_for_role`, local JSON memory |
| **Rachel Zane** | Paralegal who structures the case + drives Mike. Frames the question. | `taxonomy`, `articles_requirements` |
| **Louis Litt** | The anti-Specter. Adversarial scrutiny — finds reward-hacks, hallucinations, gaps. | `ComplianceRewardHackDetector`, `ThreeAgentVerifier` (adversary lens) |
| **Jessica Pearson** | The boss. Final ruling, weights conflicting voices. | `ThreeAgentVerifier` (referee lens) |

The agents are **deterministic** by default (rule-based personalities with
voice templates) so the test suite can pin behavior. Optional LLM-backed
mode is *per-persona* — see `PersonaCustomisation` below.

**Mike-OSS bridge** is on by default. The orchestrator constructs a
`MikeOSSBridge` pointed at `MIKE_OSS_BASE_URL` (default
`http://127.0.0.1:3000` — the Next.js dev port for Will Chen's `mike`
legal AI fork). The bridge is fail-soft: if nothing is listening, every
probe returns False sub-millisecond and Mike's recall pass uses the
canonical article catalog only. Disable per-deploy with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Peaky8linders/specter-oss](https://github.com/Peaky8linders/specter-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
