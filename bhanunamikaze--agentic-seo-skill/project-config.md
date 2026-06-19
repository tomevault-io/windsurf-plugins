---
trigger: always_on
description: >
---


# SEO Skill (Agentic / Claude / Codex)

LLM-first SEO analysis skill with 16 specialized sub-skills, 10 specialist agents, and 89 scripts for website, blog, and GitHub repository optimization.

## Deterministic Trigger Mapping

For prompt reliability in Codex/agent IDEs, map common user wording to a fixed workflow:

- If user says `perform seo analysis on <url>` (or similar generic SEO request with a URL), treat it as a **single-URL full audit**.
- If no explicit sub-skill is specified, run the full/page audit path with **LLM-first reasoning** and script-backed evidence.
- For full/page audits, always produce:
  - `FULL-AUDIT-REPORT.md` (detailed findings)
  - `ACTION-PLAN.md` (prioritized fixes)
- If `generate_report.py` is run, also return the saved HTML path (for example `SEO-REPORT.html`).

## Available Commands

| Command | Sub-Skill | Description |
|---------|-----------|-------------|
| `seo audit <url>` | [seo-audit](resources/skills/seo-audit.md) | Full website audit with scoring |
| `seo page <url>` | [seo-page](resources/skills/seo-page.md) | Deep single-page analysis |
| `seo technical <url>` | [seo-technical](resources/skills/seo-technical.md) | Technical SEO checks |
| `seo content <url>` | [seo-content](resources/skills/seo-content.md) | Content quality & E-E-A-T |
| `seo schema <url>` | [seo-schema](resources/skills/seo-schema.md) | Schema detection/validation/generation |
| `seo sitemap <url>` | [seo-sitemap](resources/skills/seo-sitemap.md) | Sitemap analysis & generation |
| `seo images <url>` | [seo-images](resources/skills/seo-images.md) | Image optimization audit |
| `seo geo <url>` | [seo-geo](resources/skills/seo-geo.md) | AI search optimization (GEO) |
| `seo programmatic <url>` | [seo-programmatic](resources/skills/seo-programmatic.md) | Programmatic SEO safeguards |
| `seo competitors <url>` | [seo-competitor-pages](resources/skills/seo-competitor-pages.md) | Comparison/alternatives pages |
| `seo hreflang <url>` | [seo-hreflang](resources/skills/seo-hreflang.md) | International SEO validation |
| `seo plan <url>` | [seo-plan](resources/skills/seo-plan.md) | Strategic SEO planning |
| `seo github <repo_or_url>` | [seo-github](resources/skills/seo-github.md) | GitHub repository discoverability, README, topics, community health, and traffic archival |
| `seo article <url>` | [seo-article](resources/skills/seo-article.md) | Article data extraction & LLM optimization |
| `seo links <url>` | [seo-links](resources/skills/seo-links.md) | External backlink profile & link health |
| `seo aeo <url>` | [seo-aeo](resources/skills/seo-aeo.md) | Answer Engine Optimization (Featured Snippets, PAA, Knowledge Panel) |

---

## Orchestration Logic

When the user requests SEO analysis, follow this routing:

### Step 1 — Identify the Task

Parse the user's request to determine which sub-skill(s) to activate:

- **Full audit**: Read `resources/skills/seo-audit.md` — crawl multiple pages, delegate to agents, score and report
- **Single page**: Read `resources/skills/seo-page.md` — deep dive on one URL
- **Specific area**: Read the matching `resources/skills/seo-*.md` file
- **Strategic plan**: Read `resources/skills/seo-plan.md` and the matching `resources/templates/*.md` for the detected industry
- **GitHub repository SEO**: Read `resources/skills/seo-github.md` and use GitHub scripts with `--provider auto` for API/`gh` fallback.
- **Generic `perform seo analysis on <url>` request**: treat as single-page full audit, read `resources/skills/seo-page.md`, and generate `FULL-AUDIT-REPORT.md` + `ACTION-PLAN.md`.

### Step 2 — Collect Evidence

**Primary method (LLM-first)** — use the built-in `read_url_content` tool first:
```
read_url_content(url)  →  returns parsed HTML content directly
```
Use this as the baseline evidence for reasoning.

**Deterministic verification (recommended when script execution is available)**:
```bash
# Fetch/parse raw HTML for structured checks
python3 <SKILL_DIR>/scripts/fetch_page.py <url> --output /tmp/page.html
python3 <SKILL_DIR>/scripts/parse_html.py /tmp/page.html --url <url> --json

# Optional: generate shareable HTML dashboard artifact
python3 <SKILL_DIR>/scripts/generate_report.py <url> --output SEO-REPORT.html
```

> **Do not use third-party mirrors (e.g., `r.jina.ai`) as primary evidence when direct site fetch or bundled scripts are available.**
> `<SKILL_DIR>` = absolute path to this skill directory (the folder containing this SKILL.md).

### Step 3 — Perform LLM-First Analysis

Use the LLM as the primary SEO analyst:

1. Synthesize evidence from page content, metadata, and optional script outputs.
2. Produce findings with explicit proof:
   - `Finding`
   - `Evidence` (specific element, metric, or snippet)
   - `Impact` (why it matters for ranking/indexing/UX)
   - `Fix` (clear implementation step)
3. Prioritize by impact and implementation effort.
4. Separate confirmed issues, likely issues, and unknowns (missing data).

Always read and apply `resources/references/llm-audit-rubric.md` to keep scoring, severity, confidence, and output structure consistent across audit types.

### Step 4 — Run Baseline Verification Scripts (When execution is available)

For full/page audits, run baseline checks to avoid hypothesis-only reporting. Do not replace LLM reasoning with script-only scoring.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bhanunamikaze/Agentic-SEO-Skill](https://github.com/Bhanunamikaze/Agentic-SEO-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
