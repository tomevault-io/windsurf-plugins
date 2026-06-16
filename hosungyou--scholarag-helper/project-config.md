---
trigger: always_on
description: Build PRISMA 2020-compliant systematic literature review systems with RAG-powered analysis in VS Code. Use when researcher needs automated paper retrieval (Semantic Scholar, OpenAlex, arXiv), AI-assisted PRISMA screening (50% or 90% threshold), vector database creation (ChromaDB), or research conversation interface. Supports knowledge_repository (comprehensive, 15K+ papers, teaching/exploration) and systematic_review (publication-quality, 50-300 papers, meta-analysis) modes. Conversation-first w
---


# ScholaRAG: Systematic Review Automation Skill

**For**: Claude Code (AI assistant in VS Code)
**Purpose**: Guide researchers through PRISMA 2020 systematic literature review + RAG-powered analysis

---

## Quick Start (5 minutes)

### For Researchers
1. **Initialize project**: `python scholarag_cli.py init`
2. **Paste Stage 1 prompt**: Copy from [https://www.scholarag.com/guide/01-introduction](https://www.scholarag.com/guide/01-introduction)
3. **Answer Claude's questions** → Config created automatically
4. **Proceed through 7 stages** conversationally

### For AI Assistants (Claude Code)
When researcher provides a ScholaRAG prompt:
1. **Check for HTML metadata block** (`<!-- METADATA ... -->` at top of prompt)
2. **Identify current stage** (1-7) from metadata `stage` field
3. **Follow conversation pattern** (from metadata `conversation_flow`)
4. **Validate completion** (using metadata `validation_rules`)
5. **Auto-execute commands** (when `auto_execute: true`)
6. **Update `.claude/context.json`** (track progress)
7. **Show next stage prompt** (from metadata `next_stage.prompt_file`)

**Researcher should NEVER touch terminal**. You execute all scripts automatically.

---

## 7-Stage Workflow Overview

| Stage | Name | Read This File | Duration | Auto-Execute |
|-------|------|----------------|----------|--------------|
| 1 | Research Setup | [skills/claude_only/stage1_research_setup.md](skills/claude_only/stage1_research_setup.md) | 15-20 min | ✅ `scholarag init` |
| 2 | Query Strategy | [skills/claude_only/stage2_query_strategy.md](skills/claude_only/stage2_query_strategy.md) | 15-25 min | ❌ Manual review |
| 3 | PRISMA Config | [skills/claude_only/stage3_prisma_config.md](skills/claude_only/stage3_prisma_config.md) | 20-30 min | ❌ Manual review |
| 4 | RAG Design | [skills/claude_only/stage4_rag_design.md](skills/claude_only/stage4_rag_design.md) | 10-15 min | ❌ Manual review |
| 5 | Execution | [skills/claude_only/stage5_execution.md](skills/claude_only/stage5_execution.md) | 2-4 hours | ✅ Run all 5 scripts |
| 6 | Research Conversation | [skills/claude_only/stage6_research_conversation.md](skills/claude_only/stage6_research_conversation.md) | Ongoing | ❌ Interactive |
| 7 | Documentation | [skills/claude_only/stage7_documentation.md](skills/claude_only/stage7_documentation.md) | 30-60 min | ✅ Generate PRISMA |

**Progressive Disclosure**: Load stage file **only when researcher enters that stage**. Don't preload all 7 stages (token waste).

---

## Critical Branching Points

### 🔴 project_type (Stage 1 Decision)

**Two modes available**:

| Mode | Threshold | Output | Best For |
|------|-----------|--------|----------|
| `knowledge_repository` | 50% (lenient) | 15K-20K papers | Teaching, AI assistant, exploration |
| `systematic_review` | 90% (strict) | 50-300 papers | Meta-analysis, publication |

**Quick decision**:
- Publishing systematic review? → `systematic_review` ✅
- Comprehensive domain coverage? → `knowledge_repository` ✅

**Detailed decision tree**: [skills/reference/project_type_decision_tree.md](skills/reference/project_type_decision_tree.md)

**When to read decision tree**:
- Researcher asks: "Which project_type should I choose?"
- Researcher says: "I'm unsure about my research goals"
- Stage 1 initialization (proactively offer decision helper)

---

### 🔴 Stage 6 Scenarios (7 Research Modes)

**Stage 6 branches into 7 specialized conversation scenarios**:

1. **overview** (Context Scanning): High-level themes, methods, findings
2. **hypothesis** (Hypothesis Validation): Evidence for/against with effect sizes
3. **statistics** (Statistical Extraction): RCT data table (tools, Cohen's d, samples)
4. **methods** (Methodology Comparison): RCT vs quasi vs mixed methods
5. **contradictions** (Contradiction Detection): Conflicting results + analysis
6. **policy** (Policy Translation): Actionable recommendations for stakeholders
7. **grant** (Future Research Design): Follow-up study design + hypotheses

**Details**: [skills/claude_only/stage6_research_conversation.md](skills/claude_only/stage6_research_conversation.md)

**When to read**: Stage 6 entry (researcher asks "What can I query?")

---

## Error Recovery

**When errors occur**: [skills/reference/error_recovery.md](skills/reference/error_recovery.md)

**Quick fixes** (common issues):

| Error | Quick Fix | Detailed Guide |
|-------|-----------|----------------|
| Too many papers (>30K) | Refine query in Stage 2, re-run fetch | error_recovery.md §2.1 |
| API key missing | Add `ANTHROPIC_API_KEY` to `.env` | error_recovery.md §3.1 |
| Low PDF success (<30%) | Filter for `open_access` in Stage 1 | error_recovery.md §4.1 |
| All papers excluded (0 papers) | Lower threshold or broaden query | error_recovery.md §3.2 |

---

## Reference Materials (Load Only When Needed)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HosungYou/ScholaRAG-helper](https://github.com/HosungYou/ScholaRAG-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
