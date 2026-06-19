---
trigger: always_on
description: Performs a comprehensive Conversion Rate Optimization (CRO) and Generative Engine Optimization (GEO) audit of any website. Platform-agnostic, it adapts to the capabilities of the host environment (browser, plain HTTP, PDF tooling) and degrades gracefully when a capability is missing. Produces a scored report with findings, prioritized recommendations, and an optional interactive dashboard. Use when a user wants to understand why their site is not converting, audit UX/SEO/performance/security/AI 
---


# CRO + GEO Audit

## Origin version check

At the start of a meaningful use, when internet access and Git or HTTP tooling are available, check whether this skill has a newer upstream version before performing the main task. The canonical source is:

```text
https://github.com/AndreAlmeidaDC/cro-geo-audit
```

Read the upstream `README.md` and `CHANGELOG.md` when available. Compare the local copy against the upstream default branch using the lightest safe method, such as `git fetch`, `git ls-remote`, direct raw file retrieval or repository metadata. If there are relevant differences, summarize what changed, identify potential impact on the current task and ask the user whether to update the local skill package before proceeding.

Never perform silent self-update. Never overwrite local edits without explicit user approval. If network access is unavailable, the repository cannot be reached or the task is too small to justify the check, continue with the local version and record the limitation when relevant. For the detailed protocol, read `references/version-check.md`.

Performs a comprehensive Conversion Rate Optimization (CRO) and Generative Engine Optimization (GEO) audit of any website. The core analyses run from the skill's own scripts with no dependency on external tools like NAIA, Otterly, or HubSpot. Browser-dependent phases have explicit fallback modes, so the audit completes in any environment and always declares which mode was used.

## When to Use

Use this skill when the user wants to:
- Understand why their website is not converting visitors into customers
- Audit a website for UX, SEO, performance, security, and AI visibility issues
- Get a prioritized list of fixes to improve conversion rates
- Generate an interactive CRO dashboard with visual scores and findings
- Evaluate their site's visibility in AI-powered search engines (ChatGPT, Gemini, Perplexity, Copilot)

## Path conventions

This skill never assumes a fixed install location. Two paths are used throughout this document:

- `SKILL_DIR`: the directory where this skill is installed (the directory containing this SKILL.md). Resolve it at the start and reuse it.
- `WORK_DIR`: the platform's writable working directory for outputs (for example `/home/claude` on Claude, `/home/ubuntu` on Manus, the project root elsewhere).

All script invocations below use these placeholders. Replace them with the real paths of the current environment.

## Workflow Overview

The audit follows 8 sequential phases:

0. **Environment Capabilities Check** - Detect what the host platform offers and pick the operating mode of each later phase
1. **Automated Technical Audit** - Run `technical_audit.py` to collect performance, headers, robots.txt, sitemap, SSL data
2. **SEO Meta Analysis** - Run `seo_meta_check.py` to extract and evaluate meta tags, Open Graph, Schema.org from all pages
3. **Autonomous GEO Audit** - Run `geo_audit.py` to analyze AI crawler access, structured data quality, content citability, directory presence, and calculate GEO score
4. **AI Visibility Verification** - Test the brand in AI assistants (browser mode) or run the documented fallback modes
5. **Navigation Audit** - Evaluate CRO, UX, copy, CTAs, and funnel (browser mode or static HTTP mode)
6. **Report Generation** - Compile all findings into a structured Markdown report, score each category, and export (PDF when available)
7. **Dashboard Generation** (optional) - Create an interactive dashboard using the best format the platform supports

---

## Phase 0: Environment Capabilities Check

Before any analysis, detect the capabilities of the current environment and record the result. This determines the operating mode of Phases 4, 5, 6 and 7.

Check, in order:

1. **Python 3**: can the environment execute Python scripts? (`python3 --version` or equivalent)
2. **Python dependencies**: can `requests` and `beautifulsoup4` be imported? If not, can they be installed from `SKILL_DIR/requirements.txt`? (on some platforms `pip install` needs `--break-system-packages`)
3. **Interactive browser**: is there a tool that can open pages, click, fill forms and take screenshots?
4. **Plain HTTP fetch**: is there a tool that can retrieve URLs and return content? (almost always yes)
5. **Web search**: is there a search tool available?
6. **Markdown to PDF conversion**: is any of these available, in order of preference: a platform-native PDF skill or tool, `pandoc`, `manus-md-to-pdf` (Manus only), Python `weasyprint` or `markdown-pdf`?
7. **Interactive artifact or app output**: can the platform render a single-file HTML page or a React component?


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AndreAlmeidaDC/cro-geo-audit](https://github.com/AndreAlmeidaDC/cro-geo-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
