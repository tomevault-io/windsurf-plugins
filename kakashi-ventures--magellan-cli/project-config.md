---
trigger: always_on
description: Autonomous AI experiment in cross-disciplinary scientific discovery. Tests
---

# MAGELLAN — Multi-Agent Generative Exploration of Latent Links Across kNowledge

Autonomous AI experiment in cross-disciplinary scientific discovery. Tests
whether a multi-agent system with frontier models (March 2026) can find real
connections between existing bodies of knowledge that humans haven't
linked yet — with zero human input on what to explore.

## Platform: Claude Code

MAGELLAN is a Claude Code application. The entire pipeline — agents, commands,
skills, hooks, MCP servers — runs within Claude Code's infrastructure.

- **Dispatch model**: `/discover` loads `.claude/agents/discovery-orchestrator.md`
  into the top-level session and the top-level Claude acts as the orchestrator,
  dispatching the 14 pipeline sub-agents via `Agent`. Sub-agents cannot spawn
  further sub-agents (Claude Code runtime constraint), so orchestration must
  run top-level. Sub-agents communicate exclusively via files in
  `results/{session-id}/`.
- **Quality enforcement**: SubagentStop hooks (`scripts/*-stop-gate.py`) validate
  each sub-agent's output. A Stop hook (`scripts/orchestrator-stop-gate.py`)
  BLOCKS session termination when required sub-agent dispatches are missing
  from `state/dispatch-log.json` (critical set: `generator`, `critic`,
  `quality-gate`). Hook schema: `exit 2` = block, `exit 0` = approve.
- **Optional env**: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is set in
  `.claude/settings.json`. Not required for `/discover` (MAGELLAN uses classic
  sub-agent dispatch, not agent teams). Retained for unrelated workflows in
  this repository.
- **Config locations**: agents in `.claude/agents/`, commands in `.claude/commands/`,
  skills in `.claude/skills/`, hooks + permissions in `.claude/settings.json`,
  MCP servers in `.mcp.json`.

## How to Run

### Primary mode (fully autonomous — this is the point)
```
/discover
```
The Scout autonomously decides WHERE to look. The full pipeline runs.
You come back to find hypothesis cards in `results/{session-id}/`.
**IMPORTANT**: Do NOT run `/discover` in plan mode. The pipeline auto-exits plan mode if active.

### Alternative modes (for testing/debugging)
```
/discover circadian biology × tumor immune evasion
/discover solve: antibiotic resistance
/discover quantum coherence in biology
```

### Launch mode
```
claude --enable-auto-mode
```

### Publishing results
Results are automatically uploaded to https://magellan-discover.ai via API
at the end of each session (requires `/connect <key>` — see README).
The upload script (`scripts/upload-session.mjs`) is run by the orchestrator
and reads `ingest.json` from the results directory.

## Architecture

Fifteen agents. Orchestrator dispatches to all — never executes phases inline.

| Agent | Model | Effort | Role |
|---|---|---|---|
| **Scout** | Opus | max | Finds WHERE to look: 10 strategies (incl. structural isomorphism + serendipity), bridge concepts, strategy diversification, exploration slot, rotating creativity constraint, TARGET QUALITY CHECK reflection |
| **Target Evaluator** | Opus | max | Adversarial challenge of Scout targets on 4 axes (popularity bias, vagueness, structural impossibility, local-optima) |
| **Literature Scout** | Sonnet | high | MCP-first retrieval (Semantic Scholar, PubMed), WebSearch fallback, full-text papers, disjointness verification |
| **Computational Validator** | Sonnet | high | Programmatic bridge verification: KEGG, STRING, PubMed co-occurrence, back-of-envelope physics |
| **Generator** | Opus | max | Hypotheses from parametric knowledge + literature + computational validation. Bisociation + multi-level abstraction. SELF-CRITIQUE with claim-level verification |
| **Critic** | Opus | max | 9 adversarial attack vectors including claim-level fact verification. META-CRITIQUE reflection. Writes critic_questions |
| **Ranker** | Sonnet | high | 6-dimension weighted scoring, per-hypothesis table, diversity check, Elo tournament sanity check, cross-domain creativity bonus (+0.5 for 2+ discipline boundaries) |
| **Evolver** | Sonnet | high | Genetic operations with diversity constraint. EVOLUTION QUALITY CHECK reflection. Conditionally skippable |
| **Quality Gate** | Opus | max | 10-point rubric + web novelty + per-claim grounding verification. META-VALIDATION reflection |
| **Session Analyst** | Sonnet | high | Post-pipeline meta-learning: strategy performance, kill patterns, bridge type analysis, creativity metrics (disciplinary distance, abstraction level, novelty type) → knowledge/meta-insights.md |
| **Cross-Model Validator** | Sonnet | high | Calls GPT-5.5 Pro (background submit + poll, reasoning xhigh, web search + code interpreter + shell) + Gemini Deep Research Max (Interactions API agent: google_search + url_context + code_execution, autonomous ~80-160 searches) for independent hypothesis validation. Generates consensus report |
| **Convergence Scanner** | Sonnet | high | Post-QG: searches ClinicalTrials.gov, NIH Reporter, patents for independent convergence signals. Finds partial mechanism confirmations from sources not consulted by pipeline |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kakashi-ventures/magellan-cli](https://github.com/kakashi-ventures/magellan-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
