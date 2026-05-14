---
trigger: always_on
description: AI-native scientific research pipeline distributed as a host-agnostic skills package. Each pipeline stage is a `SKILL.md` folder that runs on any AI coding agent supporting the [skills convention](https://github.com/vercel-labs/skills) — Cursor, OpenAI Codex CLI, Cline, Continue, Gemini CLI, Copilot, Windsurf, Claude Code, and 40+ others. Takes a research goal — optionally with a research paper — and autonomously runs a multi-step research loop. Ships with reference files for cryptography and di
---

# Reaper

AI-native scientific research pipeline distributed as a host-agnostic skills package. Each pipeline stage is a `SKILL.md` folder that runs on any AI coding agent supporting the [skills convention](https://github.com/vercel-labs/skills) — Cursor, OpenAI Codex CLI, Cline, Continue, Gemini CLI, Copilot, Windsurf, Claude Code, and 40+ others. Takes a research goal — optionally with a research paper — and autonomously runs a multi-step research loop. Ships with reference files for cryptography and distributed systems, but the skills themselves are domain-agnostic — swap the reference files to adapt to other research domains.

## Project structure

- `skills/` — 10 composable skills (each has a `SKILL.md` defining its behavior; the `/<skill>` form is the canonical display convention used in all user-facing docs)
  - `/reaper` — Main orchestrator that chains all other skills
  - `/clarify-goal` — Interactive goal clarification (asks user targeted questions before pipeline runs)
  - `/analyze-paper`, `/review-literature`, `/formalize-problem`, `/brainstorm`, `/investigate`, `/critique`, `/synthesize` — Pipeline stages
  - `/search-paper` — Academic search + citation graph + venue resolution. Bundles five Python drivers (`arxiv.py`, `iacr.py`, `semantic_scholar.py`, `dblp.py`, `openalex.py`); the `SKILL.md` itself orchestrates the layered venue lookup.
- `tests/` — Python tests for skill structure, search scripts, and L1 eval graders
- `evals/` — Layered evaluation system. L1 code-based graders (`graders/`), L2 Claude-CLI LLM judges (`judge/`), per-skill rubrics (`rubrics/`), and fixtures with reference + planted-negative variants. Orchestrator: `python3 -m evals.run_evals`. See `evals/README.md`.
- `dev/` — Development docs including `ROADMAP.md` (full methodology and design)
- `.claude-plugin/` — Claude-Code-specific plugin manifest (`plugin.json`, `marketplace.json`); other hosts ignore this directory
- `.github/workflows/` — CI (pytest + strict `npx skills` discovery check that asserts every expected skill, script, and reference file is present after installation)

## Commands

```bash
# Run tests (includes L1 structural eval graders)
pytest tests/

# Run the layered evals
python3 -m evals.run_evals --layer structural                  # L1 only — no LLM, what CI runs
python3 -m evals.run_evals --layer all --skill analyze-paper   # L1 + L2 (uses local `claude` CLI)

# Python dependencies for search skills + evals
pip install arxiv requests beautifulsoup4 pyyaml
```

## Key conventions

- Skills follow the [Agent Skills specification](https://agentskills.io/specification). Each skill directory contains a `SKILL.md` with YAML frontmatter. Required fields: `name` (1–64 chars, lowercase alphanumeric + hyphens, no leading/trailing/consecutive hyphens, must match the parent directory name) and `description` (1–1024 chars, describes both what the skill does and when to use it). Recognized optional fields: `license`, `compatibility`, `metadata`, `allowed-tools`. All skills in this repo set `license: Apache-2.0`.
- Skill authoring follows the [best practices](https://agentskills.io/skill-creation/best-practices), [description guidance](https://agentskills.io/skill-creation/optimizing-descriptions), and Anthropic's [Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf):
  - Keep `SKILL.md` under 500 lines / ~5000 tokens; use `references/` for detail loaded on demand, and tell the agent *when* to load each reference file.
  - Spend context wisely: add what the agent lacks, omit what it knows. Provide a clear default rather than a menu of options.
  - Match instruction specificity to task fragility — be prescriptive for fragile/destructive operations, descriptive (with the *why*) for flexible ones.
  - Descriptions use imperative phrasing ("Use when…"), focus on user intent, and stay under 1024 chars.
- The orchestrator skill (`/reaper`) runs the full pipeline: clarify → analyze → literature → formalize → brainstorm → investigate ↔ critique → synthesize. After delivery, users can iterate by re-invoking the `/critique` skill with feedback.
- Runtime state goes in `reaper-workspace/` (gitignored). Never commit workspace artifacts.
- The six methodology principles (separation of concerns, fixed evaluation signal, structured results log, keep-or-discard loop, never stop, clarity and simplicity) govern how skills behave.
- Domain-specific content (impossibility results, trust model checklists, venue tiers, definitional standards) lives in `skills/reaper/references/`, not inline in skills. Skills reference these files but remain domain-agnostic — the reference files can be swapped for a different research domain.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SebastianElvis/reaper](https://github.com/SebastianElvis/reaper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
