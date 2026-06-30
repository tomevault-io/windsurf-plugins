---
trigger: always_on
description: Discover an LLM-maintained financial analysis repository for systematic company research, fundamental analysis, and market intelligence in structured markdown files.
---


**Inherits:** Financial KB standards from [CLAUDE](economics/CLAUDE.md) and content infrastructure from [CLAUDE](CLAUDE.md)

**LLM-maintained financial analysis repository** - Systematic company research, fundamental analysis, and market intelligence stored as structured markdown files.

## Core Purpose

This is an **LLM-maintained financial knowledge base** where Claude actively:

- Performs fundamental and technical analysis of companies
- Creates detailed company research reports as markdown files
- Maintains sector comparisons and peer analysis
- Tracks financial metrics, shareholding patterns, and market trends
- Builds cross-referenced investment research library

**Human:** Request analysis, provide company names, direct research focus.
**LLM:** Research execution, markdown file creation, cross-referencing, consistency checks.

## Critical Requirement: Everything as Markdown Files

**ALL analysis outputs MUST be saved as standalone markdown files:**

- ✅ Each company analysis → Separate `.md` file (e.g., `reliance-industries-fundamental-analysis.md`)
- ❌ Never provide analysis only in chat - always create a markdown file

**File naming convention:** `{company-name}-{analysis-type}.md`

- Examples: `reliance-industries-fundamental.md`, `tcs-technical-analysis.md`

## Content Catalog

**See [readme.md](readme.md) for full content catalog** - Company reports, sector analyses, market intelligence, peer comparisons, investment frameworks.

## Why LLM-Maintained Financial Knowledge Base?

*"The tedious part of maintaining investment research is not the analysis — it's organizing reports, tracking updates, and cross-referencing across companies."*

**Humans struggle with:** Updating 10-15 company reports when sector dynamics change, maintaining peer comparisons, tracking shareholding pattern changes, linking related analyses.

**LLMs excel at:** Fetching real-time financial data, systematic report generation, multi-file updates, cross-sector pattern recognition, consistent formatting.

**Not traditional screener queries:** This knowledge base sits between you and raw financial data as a curated, analytical layer.

- ❌ Traditional: Query screener → Get raw data → Manually analyze → Forget analysis
- ✅ LLM Knowledge Base: Query → Fetch data → Analyze → **Save as markdown** → Cross-reference → Build knowledge library

## Three-Layer Architecture

**Layer 1: Raw Data Sources** (real-time via MCPs and web)

- Financial APIs (Groww MCP, Zerodha Kite MCP)
- Web sources (Screener.in, MoneyControl, NSE/BSE, Economic Times)
- Regulatory filings (SEBI, company investor presentations)
- Analyst reports and news
- Market data providers

**Layer 2: The Knowledge Base** (LLM-maintained markdown files)

- Company analysis reports (fundamental, technical, comprehensive)
- Sector overview documents
- Peer comparison matrices
- Shareholding pattern trackers
- Valuation summaries with cross-references
- All markdown files in `/docs/economics/company-analysis/`

**Layer 3: The Schema** (this file)

- Defines report templates, financial analysis standards
- Guides how Claude structures analysis and creates markdown files
- Sets taxonomy for sectors, analysis types, and file organization

## Core Operations

### Analyze: Creating Company Reports

When analyzing a company:

1. **Fetch Data:** Use MCP tools (Groww/Zerodha) and web sources for financials, prices, shareholding
2. **Perform Analysis:** Fundamental/technical/comprehensive based on request
3. **Create Markdown File:** Save complete analysis as `.md` file with proper frontmatter
4. **Cross-Reference:** Link to related sector reports, peer analyses, previous reports
5. **Update Catalog:** Add entry to `readme.md` content catalog

### Update: Refreshing Existing Reports

When updating analysis:

1. Read existing markdown file
2. Fetch latest data
3. Update metrics, ratios, trends
4. Note changes in shareholding, financials, or outlook
5. Append update log at bottom
6. Preserve historical analysis for comparison

### Query: Synthesizing Investment Intelligence

1. Search existing markdown files (Obsidian CLI or grep)
2. Synthesize insights across multiple companies/sectors
3. Create new synthesis document if valuable
4. Cross-reference related analyses
5. Update sector-level markdown files

### Lint: Quality Checks

Periodically verify: stale analyses (`>3` months old), broken cross-references, orphaned company reports, missing peer comparisons, inconsistent valuations.

**Commands:** See parent `/docs/CLAUDE.md` for Obsidian CLI reference (orphans, deadends, unresolved).

## Target Taxonomy

**Folder structure for financial analysis:**

- **companies/** - Individual company deep-dives

- **sectors/** - Sector-level analysis and trends
  - `sectors/oil-gas-sector-overview.md`
  - `sectors/banking-sector-npa-analysis.md`
  - `sectors/it-services-margin-trends.md`

- **peer-comparisons/** - Comparative analyses
  - `peer-comparisons/telecom-big-3-comparison.md`
  - `peer-comparisons/private-banks-valuation.md`

- **frameworks/** - Investment methodologies and checklists
  - `frameworks/fundamental-analysis-checklist.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deepaksood619/deepaksood619.github.io](https://github.com/deepaksood619/deepaksood619.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
