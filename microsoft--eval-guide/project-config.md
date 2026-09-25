---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **content/plugin repository**, not an application. It ships an AI-agent evaluation toolkit for Copilot Studio in two parallel forms:

- **Claude Code plugin** — skills under `skills/*/SKILL.md`, registered via `.claude-plugin/plugin.json` and `marketplace.json`.
- **GitHub Copilot prompts** — equivalent prompts under `.github/prompts/*.prompt.md`, with always-on instructions in `.github/copilot-instructions.md`.

There is no build, no test runner, no lint step. Changes are validated by running the skills end-to-end against an agent description.

`AGENTS.md` is the master cross-tool instruction file. `README.md` is the canonical user-facing docs. When changing user-visible behavior, keep all three (`README.md`, `AGENTS.md`, `.github/copilot-instructions.md`) in sync — they each describe the same routing table for a different audience.

## High-level architecture

### The 6 skills form a pipeline, not a flat catalog

`/eval-guide` is the orchestrator that walks the customer through the operational workflow (Discover → Plan → Generate → Run → Interpret) over Microsoft's ***Practical Guidance on Agent Evaluation* — 10-step playbook**. The canonical methodology spine is `skills/eval-guide/playbook.md`; every skill points to it rather than restating the methodology. The other 5 skills are extracted stages that can be invoked directly:

| Skill | Role |
|---|---|
| `eval-guide` | Orchestrator. Owns the dashboard workflow and stage transitions. |
| `eval-suite-planner` | Stage 1 standalone. Produces eval-suite workbook plus an interactive HTML review page. |
| `eval-generator` | Stage 2 standalone. Produces test-case CSVs / conversation blueprints. |
| `eval-result-interpreter` | Stage 4 standalone. SHIP/ITERATE/BLOCK verdict from results. |
| `eval-triage-and-improvement` | Stage 4 deep-dive. Interactive remediation. |
| `eval-faq` | Methodology Q&A grounded in Microsoft's eval ecosystem. |

When editing one stage's behavior, check whether the corresponding standalone skill needs the same change. The orchestrator and standalones share methodology but have separate SKILL.md files.

### The dashboard is the review checkpoint

`/eval-guide` uses interactive review surfaces instead of asking "does this look right?" in chat. Stage 1 Plan produces a populated workbook plus `eval-suite-<agent>-<date>-review.html`. Generate and Interpret use the dashboard server flow:

1. Skill writes stage data to `stage-N-data.json`.
2. Skill launches `python skills/eval-guide/dashboard/serve.py --stage <name> --data stage-N-data.json`.
3. `serve.py` injects the JSON into a template (`dashboard/templates/<stage>.html` composed into `base.html`), writes a standalone HTML file next to the data file, and opens it in the browser.
4. The user edits inline; on Confirm/Request Changes, a feedback JSON downloads. They save it next to the data file.
5. `serve.py` detects `<stage>-feedback.json` on disk and exits.
6. Skill reads the feedback file. **No `.docx` or `.csv` is generated until the user confirms via the dashboard.**

Stage names map to file names for served dashboards: `generate` (2), `interpret` (4). Stage 3 (Run) executes tests directly with no dashboard. The planner HTML review page follows `skills/eval-guide/plan-review-page.md`; served dashboard templates live in `dashboard/templates/`, with example stage data in `dashboard/examples/`.

### Eval execution path (Stage 3)

`skills/eval-guide/scripts/eval-runner.js` is a Node script that talks to a live Copilot Studio agent over DirectLine, runs CSV test cases, and uses the Anthropic SDK as an LLM judge for `Compare meaning` / `General quality` methods. It requires `ANTHROPIC_API_KEY` and either `--token-endpoint` or `--directline-secret`.

```bash
node skills/eval-guide/scripts/eval-runner.js --token-endpoint <url> --csv-dir <dir>
```

This is the only stage that requires a running agent.

### Versioning and self-upgrade

Every `/eval-guide` invocation runs `bin/eval-guide-update-check` from the SKILL.md `preamble:` block. The script compares `VERSION` against `microsoft/eval-guide@main` on GitHub and prints `UPGRADE_AVAILABLE <old> <new>` / `JUST_UPGRADED <old> <new>` / nothing. The skill body has explicit handling instructions for each output. State (config, snooze, just-upgraded marker) lives in `~/.eval-guide/`.

When bumping a release: edit `VERSION`, update `.claude-plugin/plugin.json` and `marketplace.json` versions, and merge to `main` so the remote check picks up the new version.

## Conventions to preserve

These are non-obvious invariants enforced across all 6 skills — keep them consistent when editing:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/eval-guide](https://github.com/microsoft/eval-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
