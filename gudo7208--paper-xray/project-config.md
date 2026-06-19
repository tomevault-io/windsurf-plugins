---
trigger: always_on
description: |
---


# Paper X-Ray: Paper Analysis & Topic Research

> Four modes: Single deep analysis / Batch parallel / Quick triage / Topic auto-survey.

## Configuration

Customize these paths before first use. All paths are relative to your workspace root (current working directory).

| Variable | Description | Default |
|----------|-------------|---------|
| NOTES_DIR | Paper notes directory | `papers/notes/` |
| READING_LIST | Reading list file path | `papers/reading-list.md` |
| INDEX_FILE | Paper notes index file | `papers/notes/index.md` |
| COGNITIVE_BASELINE | Cognitive baseline files for personal reflection (optional) | Not enabled |
| RESEARCH_FOCUS | Default research direction for triage mode | Must be provided by user |

## Usage

```
# Single mode
/paper-xray <paper name | arxiv ID | DOI | URL | PDF path>

# Batch mode
/paper-xray batch: <id1> <id2> <id3> ... [category: <topic>]

# Triage mode (quick assessment, no notes generated)
/paper-xray triage: <id1> <id2> ... [context: <research direction>]

# Topic research mode
/paper-xray topic: <research topic> [scope: <constraints>] [n: <paper count>]
```

Single mode examples:
- `/paper-xray Attention Is All You Need`
- `/paper-xray 2301.07041`
- `/paper-xray https://arxiv.org/abs/2301.07041`
- `/paper-xray ~/Downloads/paper.pdf`

Batch mode examples:
- `/paper-xray batch: 2603.04756 2504.05738 2603.05344 category: Agent`

Triage mode examples:
- `/paper-xray triage: 2603.04756 2504.05738 context: AI-assisted software engineering`

Topic mode examples:
- `/paper-xray topic: Agent in Software Engineering`
- `/paper-xray topic: Code Generation Evaluation Benchmarks scope: 2024-2026 n: 8`

## Mode Routing

Auto-detect mode based on input:
- Contains `batch:` prefix → Batch mode
- Contains `triage:` prefix → Triage mode
- Contains `topic:` prefix → Topic research mode
- Matches arxiv ID / DOI / URL / PDF path / specific paper name → Single mode
- Ambiguous (e.g., "latest LLM Agent progress") → Default to Topic mode

## Paper Type Detection

After fetching a paper, auto-detect type and use corresponding template:

| Type | Identification | Template |
|------|---------------|----------|
| Research Paper | Has clear Method + Experiments, proposes new method/framework | `references/template.md` |
| Technical Report | Title contains model name (GPT/Claude/Qwen etc.), architecture+training+eval focused, no controlled experiments | `references/tech-report-template.md` |
| Survey Paper | Title contains Survey/Review/Taxonomy, primarily categorization | `references/survey-analysis-template.md` |

Priority: Technical Report > Survey > Research Paper (default).

## Principles

- **Objectivity first**: The analysis subject is the paper itself, not personal opinions. Deconstruct the paper first, then discuss personal insights.
- **Precise terminology**: Target audience is researchers/engineers. Use proper terms, but define key concepts clearly.
- **Honest uncertainty**: Distinguish "experimentally proven" from "author speculation". Mark insufficient information, don't fabricate.
- **Structured, not narrative**: Use tables, bullet points, comparisons. No essays.

---

## Single Mode Execution Flow

### Step 1: Fetch Paper

Route based on input type:

| Input Format | Recognition Rule | Fetch Strategy |
|---------|---------|---------|
| `2301.07041` or `arxiv:2301.07041` | Digits+dot or arxiv: prefix | → `https://arxiv.org/html/{id}` WebFetch |
| `https://arxiv.org/abs/...` | arxiv abs URL | → Replace with `/html/` WebFetch |
| `https://arxiv.org/pdf/...` | arxiv pdf URL | → Replace with `/html/` WebFetch |
| `https://arxiv.org/html/...` | arxiv html URL | → Direct WebFetch |
| `10.xxxx/...` | DOI format | → WebSearch for arxiv/html version |
| `~/xxx.pdf` or `/xxx.pdf` | Local path | → Read tool to read PDF |
| Other URL | Generic URL | → WebFetch |
| Paper name (natural language) | No match above | → WebSearch, prefer arxiv html |

**Fetch priority**: arxiv HTML > other HTML > PDF.

**Robust fetch strategy** (arxiv papers):
1. Try `https://arxiv.org/html/{id}`
2. If HTML unavailable (404 or incomplete), try `https://arxiv.org/abs/{id}` for abstract page
3. If abs page also fails, WebSearch `arxiv {id}` for alternative sources
4. Last resort: WebSearch paper title for any available version

**Long paper handling**: Fetch Abstract + Introduction + Conclusion first for global understanding, then fetch Method, Experiments etc. as needed. Don't dump everything at once.

**Metadata extraction**: Title, authors, publication date, venue, arxiv ID (if available).

**arxiv ID year parsing**: ID format `YYMM.NNNNN`, first two digits = year. `2602.09540` = Feb 2026, `2301.07041` = Jan 2023.

### Step 2: Problem & Motivation Analysis

- **Problem definition**: 1-2 sentences precisely describing the core problem
- **Existing method limitations**: Where previous work got stuck — theoretical/engineering/data bottleneck?
- **Key Insight**: What did the authors see that others missed?

### Step 3: Technical Approach Deconstruction

- **Overall architecture**: Mermaid diagram or structured description of the system/method flow
- **Key technical contributions**: Break down 2-3 core contributions, each explaining: what it is, why, and how it fundamentally differs from existing methods

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gudo7208/paper-xray](https://github.com/gudo7208/paper-xray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
