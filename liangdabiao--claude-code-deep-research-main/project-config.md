---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview


This is a **Claude Code Deep Research Agent** framework that implements a sophisticated multi-agent research system to conduct comprehensive, citation-backed research projects. It replicates and enhances OpenAI's Deep Research and Google Gemini's Deep Research capabilities using Claude Code's native features.

**Core Technology:**
- **Graph of Thoughts (GoT)** Framework - Intelligent research path management with graph-based reasoning
- **7-Phase Deep Research Process** - Structured methodology for quality research
- **Multi-Agent Architecture** - Parallel research agents with specialized roles
- **Citation Validation System** - A-E source quality ratings with chain-of-verification

## Quick Start Commands

### Primary Research Command

```bash
/deep-research [research topic]
```

This single command executes the complete research workflow:
1. Question refinement (asks clarifying questions)
2. Research planning with subtopics
3. Multi-agent parallel research deployment
4. Source triangulation and synthesis
5. Citation validation
6. Output to `RESEARCH/[topic]/` directory

### Step-by-Step Commands

```bash
/refine-question [raw question]     # Generate structured research prompt
/plan-research [structured prompt]   # Create execution plan
/synthesize-findings [directory]     # Combine research outputs
/validate-citations [file]           # Verify citation quality
```

## Architecture

### 7-Phase Research Process

1. **Question Scoping** - Define boundaries, output format, constraints
2. **Retrieval Planning** - Break topic into subtopics, plan agent deployment
3. **Iterative Querying** - Deploy parallel research agents (3-8 agents)
4. **Source Triangulation** - Cross-validate findings across sources
5. **Knowledge Synthesis** - Combine into coherent narrative with citations
6. **Quality Assurance** - Fact-check, validate citations, detect hallucinations
7. **Output & Packaging** - Generate structured research documents

### Skills System

The `.claude/skills/` directory contains modular research capabilities:

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `question-refiner` | Transform raw questions into structured prompts | Vague topics, undefined scope |
| `research-executor` | Execute full 7-phase research process | Structured research prompts |
| `got-controller` | Manage Graph of Thoughts for complex topics | Multi-faceted research, quality-critical |
| `citation-validator` | Verify citation accuracy and quality | Final reports, publishing preparation |
| `synthesizer` | Combine findings from multiple agents | Contradictory sources, comprehensive reports |

### Graph of Thoughts Operations

The GoT framework manages research as a graph with these transformations:

- **Generate(k)**: Spawn k parallel research paths from a node
- **Aggregate(k)**: Merge k findings into stronger synthesis
- **Refine(1)**: Improve existing finding without new research
- **Score**: Rate quality (0-10) based on citations, accuracy, completeness
- **KeepBestN(n)**: Prune to top n nodes per level

**Research Patterns:**
- **Balanced**: Generate(4-5) → Score best → Deepen top paths → Aggregate
- **Depth-first**: Generate(3) → Take best → Generate(3) from it → Continue
- **Breadth-first**: Generate(8) → KeepBestN(5) → Generate(2) from each → Aggregate

### Multi-Agent Deployment Strategy

When executing research, deploy agents in parallel:

```
Phase 3: Iterative Querying
├── Web Research Agents (3-5): Current info, trends, news
├── Academic/Technical Agents (1-2): Papers, specifications
└── Cross-Reference Agent (1): Fact-checking, verification
```

Each agent receives:
- Clear description of research focus
- Specific search queries
- Expected output format
- Citation requirements

## Output Structure

Research outputs are created in `RESEARCH/[topic_name]/`:

```
RESEARCH/[topic_name]/
├── README.md                    # Overview and navigation
├── executive_summary.md         # 1-2 page key findings
├── full_report.md               # Complete analysis (20-50 pages)
├── data/
│   └── statistics.md            # Key numbers, facts
├── visuals/
│   └── descriptions.md          # Chart/graph descriptions
├── sources/
│   ├── bibliography.md          # Complete citations
│   └── source_quality_table.md  # A-E ratings
├── research_notes/
│   └── agent_findings_summary.md # Raw agent outputs
└── appendices/
    ├── methodology.md           # Research methods
    └── limitations.md           # Unknowns, gaps
```

## Citation Requirements

**Every factual claim must include:**
1. Author/Organization name
2. Publication date
3. Source title
4. Direct URL/DOI
5. Page numbers (if applicable)

**Source Quality Ratings:**
- **A**: Peer-reviewed RCTs, systematic reviews, meta-analyses
- **B**: Cohort studies, clinical guidelines, reputable analysts
- **C**: Expert opinion, case reports, mechanistic studies
- **D**: Preprints, preliminary research, blogs
- **E**: Anecdotal, theoretical, speculative

## Tool Permissions

The `.claude/settings.local.json` file configures allowed tools:
- **WebSearch**: General web searches

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liangdabiao/Claude-Code-Deep-Research-main](https://github.com/liangdabiao/Claude-Code-Deep-Research-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
