---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run all tests (102 tests)
pytest tests/ -v

# Run specific test suites
pytest tests/test_crisis_engine.py tests/test_temporal_graph.py -v
pytest tests/test_persona_agent.py tests/test_audience.py tests/test_memory_store.py -v
pytest tests/test_crisis_engine.py tests/test_experiment.py -v

# Run single test
pytest tests/test_crisis_engine.py -v -k "test_name"

# Run crisis simulation demo (rule mode, no LLM needed)
python -m demos.crisis_simulation

# Web visualization (includes crisis simulation tab)
python run_viz.py
# http://localhost:8765 → 知识图谱 tab + 危机模拟 tab

# Scraper test (mock mode)
python test_scraper.py mock 杨幂

# Scraper test (real mode, needs network)
python test_scraper.py real 杨幂

# Update celebrity data (Baidu Baike + Weibo, needs WEIBO_COOKIE in .env)
python update_weibo_data.py --names 杨幂 赵丽颖
python update_weibo_data.py --all
python update_weibo_data.py --validate  # 仅验证 Cookie
```

## Environment Setup

Copy `.env.example` to `.env` and set `GEMINI_API_KEY`. Other keys (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`) optional. Config via env vars: `LLM_PROVIDER` (default `gemini`), `LLM_MODEL` (default `gemini-3-flash-preview`), `MAX_AGENTS`, `SIMULATION_TICK_INTERVAL`, `DEFAULT_TURNS`.

## Architecture

### Crisis Simulation Flow

```
CrisisSimulation.step() → per day:
  1. Timeline      → CrisisTimeline.advance_day() → determine CrisisPhase
  2. Intervention  → InterventionSystem.check() → apply user's what-if conditions
  3. Decision      → randomized agent order, sequential decisions with peer_actions + audience_reactions
  4. MessageBus    → audience reactions + celebrity actions broadcast to bus
  5. Effects       → CrisisActionSpace or FreeActionSpace (mode-dependent) → approval/heat/brand deltas
  6. Debunk        → STATEMENT/LAWSUIT can debunk rumors (40%/60% chance)
  7. Reactions     → InformationVacuumDetector → silence → event-driven rumor cascade
  8. Update        → trending topics, media headlines, brand statuses
  9. Decay         → heat -10%, approval regresses toward role-specific target
 10. Termination   → heat <15 for 3 days OR regulatory level 5 → early end
```

### Key Module Relationships

```
swarmsim/graph/
├── knowledge_graph.py    # networkx MultiDiGraph engine, loads from JSON/mock
└── temporal.py           # TemporalKnowledgeGraph — date-indexed, person timelines

swarmsim/crisis/          # Crisis simulation engine
├── models.py             # CrisisPhase(6), PRAction(10), CrisisRole(4), InteractionMode, FreeAction(11), CrisisState, etc.
├── timeline.py           # 1 turn = 1 day, 6-phase lifecycle
├── action_space.py       # 10 PR actions × 6 phases effect matrix + Big Five mods
│                         # FreeActionSpace — 11 free actions × 6 phases + personality mods + low-approval penalty
├── persona_agent.py      # CelebrityPersonaAgent (rule/LLM), builds persona from GraphRAG
│                         # crisis_role (PERPETRATOR/VICTIM/ACCOMPLICE/BYSTANDER) injected by engine
│                         # gossip_type (CHEATING/SCANDAL/DIVORCE/DRUGS/TAX_EVASION/OTHER) drives decision
│                         # Role-aware + gossip-type-aware decision: perp banned from comeback, drugs bans counterattack
│                         # Extended peer_influence: strength-weighted (relation strength 0-1 scales boost)
│                         # Memory-driven: consecutive silence → urgency, repeated apology → strategy switch
│                         # Semantic repeat penalty: similar action groups share penalty (SILENCE+HIDE, etc.)
│                         # Audience semantic analysis: keyword extraction → action-specific boosts
│                         # Structured memory via MemoryStore (add/search/get_recent/get_important)
│                         # Personality natural language description for LLM prompts
│                         # LLM prompt: graph context + timeline events + memory summary + personality description
│                         # Free mode: generate_free_response() / _rule_decide_free() / _llm_decide_free()
│                         # Enhanced free LLM prompt: includes state/role/gossip_type/audience
├── vacuum_detector.py    # Silence → rumor cascade, probability escalates with days
│                         # Event-driven rumor generation: extract topic from scenario description
│                         # (e.g. "代言带货/虚假宣传", "出轨/婚变") → combine with rumor angles
│                         # (escalation/coverup/chain/insider/digging) → context-relevant rumors
│                         # Priority: LLM > event-context rumor > title-referenced fallback
│                         # Debunk mechanism: STATEMENT/LAWSUIT can debunk rumors (severity → 20%)
├── intervention.py       # User what-if conditions with trigger types (TIME_ABSOLUTE/TIME_RELATIVE/STATE_THRESHOLD)
│                         # External events (MEDIA_REPORT/VIDEO_LEAK/COMPETITOR_ANNOUNCE/etc.)
├── experiment.py         # ExperimentManager, Experiment, ExperimentGroup, ComparisonResult — A/B testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chenwei-zhang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
