---
trigger: always_on
description: > Full architecture documentation is in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).
---

# Frieren DAST-AI — Claude Code Instructions

> Full architecture documentation is in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Product Vision

Frieren DAST-AI is a proxy + AI-driven scanner built to be simpler to use and smarter than
existing DAST tools. The bar is: better and easier than the alternatives.

- **Deep contextual reasoning**: the coordinator understands what an endpoint does before
  deciding what to test, not just pattern-matching URL structure.
- **Better than Aikido**: fewer false positives, more actionable findings, real exploit evidence.
- **Simple by default**: intercept traffic, get findings — no manual configuration required.

Every decision in the codebase should be evaluated against this bar: does this make the tool
smarter, more accurate, or easier to use? If not, don't add it.

## Project Goal

**Find real, exploitable vulnerabilities in running web applications.**

Highest true positive rate, lowest false positive rate. A finding must survive the full
pipeline (agent detection → LLM validation, or deterministic evidence) before it reaches
the dashboard. A false positive that wastes a developer's time is a failure.

## Project Overview

Frieren DAST-AI is an AI-driven DAST tool combining:
- HTTPS MITM proxy + real-time dashboard (FastAPI + WebSocket)
- Multi-agent scanner: canary pre-probe → LLM Coordinator → parallel VulnAgents → Red-Team Validator
- Tech-stack-aware payload filtering: Wappalyzer (~3000 signatures) + manual rules
- Data-driven passive scanner: 62+ rules in YAML under `dast/passive_rules/`
- Tiered model support: Haiku (planning), Opus (validation), configurable per-tier

**Primary Users:** Security engineers
**Primary Use Case:** Active DAST scanning of web applications via proxy interception

---

## Important Conventions

### Code Style
1. **All code, comments, and docs in English** — no Portuguese
2. **No emojis** in code or documentation
3. **Type hints** on all function signatures
4. **Descriptive variable names** — no abbreviations
5. **Error handling** — log and continue; never crash the scan on one endpoint
6. **Modularity first** — keep modules small and single-responsibility. Never add a feature
   to an existing module when it fits better in a new one. Complexity should stay local;
   cross-module coupling should be explicit and minimal.
7. **Understand before acting** — agents, plugins, and LLM prompts must read and interpret
   the actual request/response context before testing anything. Never assume a vulnerability
   category from the URL path alone; use params, body, headers, and response content as
   evidence. A tool that spams payloads without understanding the endpoint is noise, not signal.
8. **Logging required everywhere** — every module must have `logger = get_logger(__name__)`.
   Log at the appropriate level:
   - `logger.debug` — probe attempts, payload choices, iteration steps (high volume, off by default)
   - `logger.info` — agent start/done, scan start/result, session load/save, findings recorded
   - `logger.warning` — recoverable errors, LLM call failures, unexpected responses, FP filter hits
   - `logger.error` — unrecoverable errors, exceptions that abort an operation
   Never swallow exceptions silently (`except Exception: pass`) — always log them with `error=str(exc)`.

### Package Manager
**ALWAYS use uv:**
```bash
uv run dast-ai proxy          # start proxy + dashboard
uv run pytest
```
Never use `python3` directly. Never activate venv manually.

### Agent Development
- Add a new file in `dast/agents/` inheriting from `VulnAgent`
- Call `get_filtered_payloads(attack_type, target)` from `dast/agents/payload_filter.py` instead of `get_payloads()` directly
- Load extra payloads from YAML in `dast/payloads/` via `loader.py`
- Call `Coordinator.register(YourAgent)` at the bottom of the module
- Import the module in `dast/agents/__init__.py`
- Safety policy: no destructive payloads; SLEEP/WAITFOR max 5s; detection only
- Auth agent: use `urlparse(target.url).path` for path-based bypass headers, not hardcoded paths

### Payload Development
- Add or edit `dast/payloads/*.yaml` — no code changes needed
- Use `get_payloads(category, group)`, `get_all_payloads(category)`,
  `get_detection_payloads(category)` from `dast/payloads/loader.py`
- Document intent with comments in the YAML

### Passive Scanner Rule Development
- Add a `.yaml` file anywhere under `dast/passive_rules/` — it is auto-discovered at startup
- Follow the rule schema in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- `one_per_host: true` in conditions prevents noisy per-request repetition
- `needs_ai_validation: true` routes the finding through the LLM before storing it
- Run `uv run python -c "from dast.plugins.passive_scanner import _load_all_rules; print(len(_load_all_rules()), 'rules')"` to verify

### Content Discovery (forced browsing)
- `dast/wordlists/*.txt` — plain-text wordlists (one entry per line, `#`-comments
  ignored), SecLists-derived (MIT, see each file header). `directories.txt` (~100),
  `files.txt` (~100), `graphql.txt` (endpoint paths). Add a list by dropping a
  `<name>.txt` — load it with `load_wordlist("<name>")` from `dast/wordlists/loader.py`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [knowbe4/frieren-dast-ai](https://github.com/knowbe4/frieren-dast-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
