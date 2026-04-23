---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

This repository provides a Claude Code plugin marketplace with skills (agents) for rigorous social science research analysis. Each skill guides users through a structured, phased workflow with pauses for user input at each stage.

## Installation

Users can install this plugin marketplace and individual plugins with:

```bash
# Add the marketplace
/plugin marketplace add nealcaren/social-data-analysis

# Install only the plugins you need
/plugin install r-analyst@social-data-analysis
/plugin install stata-analyst@social-data-analysis
/plugin install interview-analyst@social-data-analysis
/plugin install interview-writeup@social-data-analysis
/plugin install dag-development@social-data-analysis
/plugin install abductive-analyst@social-data-analysis
/plugin install text-analyst@social-data-analysis
/plugin install lecture-designer@social-data-analysis
/plugin install lit-search@social-data-analysis
/plugin install lit-synthesis@social-data-analysis
/plugin install lit-writeup@social-data-analysis
/plugin install interview-bookends@social-data-analysis
/plugin install genre-skill-builder@social-data-analysis
/plugin install methods-writer@social-data-analysis
/plugin install case-justification@social-data-analysis
/plugin install revision-coordinator@social-data-analysis
```

## Available Skills

After installation, invoke skills with:

| Skill | Purpose | Invocation |
|-------|---------|------------|
| **R Analyst** | Statistical analysis in R for publication | `/r-analyst` |
| **Stata Analyst** | Statistical analysis in Stata for publication | `/stata-analyst` |
| **Interview Analyst** | Qualitative analysis of interview data | `/interview-analyst` |
| **Interview Write-Up** | Write-up support for interview methods and findings | `/interview-writeup` |
| **DAG Development** | Develop causal diagrams and render publication-ready figures | `/dag-development` |
| **Abductive Analyst** | Abductive analysis (Timmermans & Tavory) | `/abductive-analyst` |
| **Text Analyst** | Computational text analysis (R/Python) | `/text-analyst` |
| **Lecture Designer** | Transform chapters into engaging lectures | `/lecture-designer` |
| **Lit Search** | Build literature databases via OpenAlex | `/lit-search` |
| **Lit Synthesis** | Deep reading, theoretical mapping, debate identification | `/lit-synthesis` |
| **Lit Write-Up** | Draft publication-ready Theory sections | `/lit-writeup` |
| **Interview Bookends** | Draft introductions and conclusions from theory/findings | `/interview-bookends` |
| **Genre Skill Builder** | Create genre-analysis-based writing skills from a corpus | `/genre-skill-builder` |
| **Methods Writer** | Draft Methods sections for interview articles (3 pathways) | `/methods-writer` |
| **Case Justification** | Draft case justification sections (5 clusters) | `/case-justification` |
| **Revision Coordinator** | Orchestrate manuscript revision by routing feedback to skills | `/revision-coordinator` |

## Unified Phased Architecture

All skills follow the same phased structure with pauses between phases:

### Statistical Analysis (R & Stata)

| Phase | Goal | Pause Point |
|-------|------|-------------|
| **0: Research Design** | Establish identification strategy | User confirms design |
| **1: Data Familiarization** | Understand data before modeling | User reviews descriptives |
| **2: Model Specification** | Fully specify before estimation | User approves specification |
| **3: Main Analysis** | Run primary models | User reviews results |
| **4: Robustness** | Stress-test findings | User assesses robustness |
| **5: Output** | Publication-ready outputs | Analysis complete |

### Interview Analysis

| Phase | Goal | Pause Point |
|-------|------|-------------|
| **0: Theory Synthesis** | Build theoretical sensitivity (Track A only) | User confirms framework |
| **1: Immersion** | Deep familiarity with data | User reviews observations |
| **2: Coding** | Systematic categorization | User reviews coding structure |
| **3: Interpretation** | Move from "what" to "why" | User reviews explanations |
| **4: Quality Check** | Evaluate against quality indicators | User addresses gaps |
| **5: Synthesis** | Integrate into coherent argument | Analysis complete |

### Abductive Analysis (Timmermans & Tavory)

| Phase | Goal | Pause Point |
|-------|------|-------------|
| **0: Theoretical Preparation** | Build theoretical sensitivity | User confirms theoretical map |
| **1: Familiarization** | Open coding, flag surprises | User reviews initial codes |
| **2: Theoretical Casing** | Apply multiple lenses to excerpts | User reviews casings |
| **3: Anomaly Analysis** | Identify contradictions and puzzles | User confirms focus |
| **4: Memo Writing** | Develop tentative theory | User tests interpretations |
| **5: Integration** | Test against full dataset | User reviews synthesis |
| **6: Writing Up** | Rhetorical abduction for publication | Analysis complete |

### Text Analysis (R/Python)

| Phase | Goal | Pause Point |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nealcaren/social-data-analysis](https://github.com/nealcaren/social-data-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
