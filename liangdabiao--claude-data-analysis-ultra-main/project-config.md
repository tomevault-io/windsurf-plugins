---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Data Analysis Assistant is an intelligent data analysis platform that leverages Claude Code's sub-agents, slash-commands, skills, and hooks to provide a comprehensive data analysis workflow. This project combines custom skills for specialized analysis (RFM segmentation, attribution modeling, funnel analysis, etc.) with a flexible agent-based architecture.

## Quick Start

### ⭐ NEW: Automatic Multi-Skill Analysis
The easiest way to analyze your data:

```bash
# 1. Place data files in data_storage/
# 2. Run this single command:
/do-more

# That's it! It will:
# - Auto-discover all data files
# - Match relevant analysis skills
# - Execute skills in optimal order
# - Generate comprehensive HTML report
# - Takes 2-5 minutes, no human intervention needed
```

### Traditional Usage
1. Place your data files in the `data_storage/` directory
2. Use specialized skills directly (e.g., `/rfm-customer-segmentation`) or universal commands
3. Use `/analyze [filename]` for general data analysis
4. Use `/visualize [filename]` to create visualizations

### ⚡ Quick Decision Guide: Which Command Should I Use?

**Choose `/do-more` when you want:**
- ✅ Fast, automated analysis (2-5 minutes)
- ✅ No configuration or parameters
- ✅ Quick insights and overview
- ✅ Interactive HTML report
- ✅ Set-it-and-forget experience

**Choose `/do-all` when you want:**
- ✅ Thorough, deep analysis
- ✅ Human oversight and feedback at key stages
- ✅ Interactive hypothesis generation
- ✅ Custom code generation
- ✅ Multiple output formats (HTML, PDF, DOCX)
- ✅ Complete documentation pipeline
- ✅ Time for interactive session (10-30 minutes)

**Example comparison:**
```bash
# Quick insights (2-5 min, automatic)
/do-more

# OR thorough analysis (10-30 min, interactive)
/do-all
# → [checkpoint 1] Review data quality
# → [checkpoint 2] Approve hypotheses
# → [checkpoint 3] Review visualizations
# → Receive complete report with code
```

## Core Architecture

### Skills System (Primary Analysis Tools)
The project uses 12 specialized skills for different analysis types:

**Customer Analysis Skills:**
- `rfm-customer-segmentation`: RFM analysis for e-commerce customer segmentation
- `user-profiling-analysis`: Comprehensive user behavior analysis and profiling
- `ltv-predictor`: Customer lifetime value prediction
- `retention-analysis`: User retention and churn analysis

**Marketing Analysis Skills:**
- `attribution-analysis-modeling`: Multi-touch attribution using Markov chains and Shapley values
- `growth-model-analyzer`: Growth hacking and user acquisition analysis
- `ab-testing-analyzer`: A/B test analysis and statistical validation
- `funnel-analysis`: Conversion funnel analysis with segmentation

**Data Analysis Skills:**
- `data-exploration-visualization`: Automated EDA and visualization
- `regression-analysis-modeling`: Predictive modeling and regression analysis
- `content-analysis`: Text and content analysis using NLP
- `recommender-system`: Recommendation engine implementation

### Universal Commands
- **`/do-more`**: ⭐ **RECOMMENDED** - Automatic multi-skill analysis (no parameters, observes data_storage/)
- **`/do-all`**: Complete interactive workflow with human feedback checkpoints (no parameters, uses data from data_storage/)
- `/analyze [dataset] [analysis_type]`: General data analysis (exploratory, statistical, predictive, complete)
- `/visualize [dataset] [chart_type]`: Create visualizations
- `/generate [language] [analysis_type]`: Generate analysis code
- `/report [dataset] [format]`: Generate analysis reports
- `/quality [dataset] [action]`: Data quality validation
- `/hypothesis [dataset] [domain]`: Generate research hypotheses

### /do-more Command Details

**Purpose**: Automatically observe data in `data_storage/`, intelligently match skills, execute analyses, and generate integrated reports

**Workflow**:
1. **Data Observation**: Scans all files in `data_storage/`
2. **Type Detection**: Identifies data types (e-commerce, user behavior, marketing, etc.)
3. **Skill Matching**: Selects relevant skills based on data characteristics
4. **Sequential Execution**: Runs skills in optimal order
5. **Result Integration**: Combines all outputs into comprehensive reports

**Output Structure**:
```
do_more_analysis/
├── skill_execution/
│   ├── data-exploration-visualization/
│   ├── rfm-customer-segmentation/
│   ├── ltv-predictor/
│   ├── retention-analysis/
│   ├── funnel-analysis/
│   ├── growth-model-analyzer/
│   └── content-analysis/
└── integrated_results/
    └── Comprehensive_Analysis_Report.html  # Interactive HTML report
```

**Supported Data Types**:
- **E-commerce**: Orders, Customers, Products, Reviews
- **User Behavior**: Clickstream, sessions, events
- **Marketing**: Campaigns, attributions, funnels
- **General**: Any CSV/JSON/Excel dataset

### Sub-Agents System
- **data-explorer**: Statistical analysis and pattern discovery
- **visualization-specialist**: Chart and graph creation
- **code-generator**: Analysis code generation
- **report-writer**: Comprehensive report generation
- **quality-assurance**: Data validation and quality control

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liangdabiao/claude-data-analysis-ultra-main](https://github.com/liangdabiao/claude-data-analysis-ultra-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
