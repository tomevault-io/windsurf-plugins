---
trigger: always_on
description: TrendR is an automated literature review + platform hotspot monitoring system.
---

# TrendR — Codex / Multi-Agent Configuration

TrendR is an automated literature review + platform hotspot monitoring system.
It works natively with OpenClaw, and this file provides compatibility for Codex and other agents.

## Runtime Contract

TrendR uses canonical runtime names:
- `openclaw`
- `codex`
- `claude-code`
- `cli`

Alias normalization:
- `claudecode -> claude-code`

Runtime detection priority:
1. CLI explicit `--platform`
2. `TRENDR_PLATFORM`
3. `OPENCLAW_SESSION_ID`
4. any `CODEX_*` env key
5. any `CLAUDE_CODE_*` env key
6. `cli`

For every SKILL.md:
- Execute only the command block for the current runtime.
- All non-target runtime blocks must be treated as `dormant` and explicitly skipped.

## Quick Start

Before any research task, read the relevant skill file:
- Literature review: `skills/paper-scout/SKILL.md` → `skills/paper-analyzer/SKILL.md` → `skills/review-writer/SKILL.md`
- Verification: `skills/verifier/SKILL.md`
- Platform hotspots: `skills/platform-hotspots/SKILL.md`
- Chrome automation: `skills/chrome-cdp-setup/SKILL.md`

Runtime-specific authority files:
- `codex` → additionally read `CODEX.md`, `skills/*/codex.md`, `agents/*/codex.md`
- `claude-code` → additionally read `CLAUDE.md`, `skills/*/claude-code.md`, `agents/*/claude-code.md`
- `openclaw` → continue using `SKILL.md` + `SOUL.md`

## Agent Roles

TrendR defines 4 agent roles. In multi-agent runtimes, dispatch them as subagents.
In single-agent runtimes (Codex), execute their responsibilities sequentially.

### review-lead (Orchestrator)
- Reads: `skills/review-writer/SKILL.md`, `skills/trendr-watchdog/SKILL.md`
- Coordinates the v2 state-machine pipeline
- Writes the final review — never delegates writing
- See: `agents/review-lead/SOUL.md` / `agents/review-lead/codex.md` / `agents/review-lead/claude-code.md`

### paper-scout (Search)
- Reads: `skills/paper-scout/SKILL.md`
- Searches 3-5 academic APIs based on research domain
- Outputs: `candidates.csv`, `search_log.md`
- See: `agents/paper-scout/SOUL.md` / `agents/paper-scout/codex.md` / `agents/paper-scout/claude-code.md`

### paper-analyzer (Analysis)
- Reads: `skills/paper-analyzer/SKILL.md`
- Deep-reads papers with relevance >= 4
- Outputs: `notes/*.md`, `matrix.csv`
- See: `agents/paper-analyzer/SOUL.md` / `agents/paper-analyzer/codex.md` / `agents/paper-analyzer/claude-code.md`

### verifier (Verification)
- Reads: `skills/verifier/SKILL.md`
- Runs after WRITING in v2: citation existence, citation reality, claim support, coverage, taxonomy consistency, bib quality
- Outputs: `verify.json`
- See: `agents/verifier/SOUL.md` / `agents/verifier/codex.md` / `agents/verifier/claude-code.md`

## Engine Runtime

TrendR v2 adds a standalone engine layer under `engine/`:

- `engine/state_machine.py`: drives `INIT → DISCOVERY → ANALYSIS → GAP_CHECK → WRITING → VERIFY → DONE`
- `engine/validators.py`: validates file contracts before transitions
- `engine/watchdog.py`: monitors `heartbeat.json` + `run_state.json` and writes `resume_request.json`
- `engine/adapters/`: platform adapters (`openclaw`, `cli`) implementing the runtime bridge

If the project directory contains `run_state.json` with `version: 2`, follow the engine state instead of manually deciding the next phase.

For `codex` / `claude-code` / `cli` runtimes:
- Default to sequential execution (stability first)
- Enable subagent parallelism only in DISCOVERY/ANALYSIS when task boundaries are clear
- Use file-based watchdog recovery (`engine/watchdog.py` + `resume_request.json`)

## Tool Mapping

TrendR skills use OpenClaw tool names. Map them to your runtime:

| OpenClaw tool | Generic equivalent |
|--------------|-------------------|
| `web_fetch <url>` | HTTP GET (curl, fetch, requests) |
| `web_search <query>` | Web search API |
| `exec: <cmd>` | Shell command execution |
| `read <path>` | File read |
| `write <path>` | File write |
| `sessions_spawn` | Subagent dispatch (or sequential execution) |
| `sessions_yield` | Wait for subagent result |
| `openclaw browser ...` | Chrome CDP / Playwright / Puppeteer |

## 9 Academic API Sources

All free, no keys required (Semantic Scholar key recommended):

1. **arXiv** — `export.arxiv.org/api/query` (3s rate limit)
2. **Semantic Scholar** — `api.semanticscholar.org/graph/v1`
3. **OpenAlex** — `api.openalex.org/works`
4. **PubMed** — `eutils.ncbi.nlm.nih.gov/entrez`
5. **CrossRef** — `api.crossref.org/works`
6. **DBLP** — `dblp.org/search/publ/api`
7. **Europe PMC** — `www.ebi.ac.uk/europepmc/webservices/rest`
8. **bioRxiv** — `api.biorxiv.org/details`
9. **Papers with Code** — `paperswithcode.com/api/v1`

Full URL templates and response parsing rules are in `skills/paper-scout/SKILL.md`.

## 9 Platform Hotspot Sources

Requires Chrome CDP or browser automation for JS-heavy sites:

1. Zhihu Hot List
2. Zhihu Tech Search
3. 
4. X (Twitter) AI/Tech Search
5. Reddit AI/Tech Search
6. YouTube Trending/AI
7. GitHub Trending
8. Hacker News
9. Product Hunt

Extraction selectors and commands are in `skills/platform-hotspots/SKILL.md`.
Chrome CDP setup guide is in `skills/chrome-cdp-setup/SKILL.md`.

## Output Convention

```
~/research/{project-name}/
├── candidates.csv          # Paper search results
├── search_log.md           # Search strategy log

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gy-hou/trendr](https://github.com/gy-hou/trendr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
