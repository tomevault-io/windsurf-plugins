---
trigger: always_on
description: Dingo is a comprehensive AI data quality evaluation tool for ML practitioners, data engineers, and AI researchers. It systematically assesses training data, fine-tuning datasets, and production AI systems using rule-based, LLM-based, and agent-based evaluation methods.
---

# Dingo — Agent Instructions

## Project Overview

Dingo is a comprehensive AI data quality evaluation tool for ML practitioners, data engineers, and AI researchers. It systematically assesses training data, fine-tuning datasets, and production AI systems using rule-based, LLM-based, and agent-based evaluation methods.

**Repository**: https://github.com/MigoXLab/dingo
**PyPI**: `pip install dingo-python`
**License**: Apache 2.0

## Tech Stack

| Layer | Technology |
|-------|------------|
| Language | Python 3.10+ |
| Data Models | Pydantic (BaseModel, extra="allow") |
| LLM Integration | OpenAI SDK (supports any compatible API) |
| MCP Server | FastMCP + SSE transport |
| Distributed | PySpark (optional) |

## Directory Structure

```
dingo/
├── AGENTS.md                ← this file (agent instructions)
├── setup.py                 ← package config (extras_require for optional deps)
├── mcp_server.py            ← MCP server entry point (legacy, use `dingo serve` instead)
├── requirements/
│   ├── runtime.txt          ← core dependencies (minimal)
│   ├── datasource.txt       ← optional datasource deps (S3, SQL, Parquet, etc.)
│   ├── optional.txt         ← heavy optional deps (torch, pyspark, etc.)
│   └── agent.txt            ← agent evaluation deps (langchain, tavily)
│
├── SKILL.md                 ← AI agent skill definition (symlink → clawhub/SKILL.md)
├── dingo/                   ← core Python package
│   ├── config/
│   │   └── input_args.py    ← InputArgs, EvalPiplineConfig, EvaluatorGroupConfig
│   ├── io/
│   │   ├── input/data.py    ← Data model (Pydantic, extra="allow")
│   │   └── output/          ← ResultInfo, EvalDetail, SummaryModel (+ cross-layer analysis)
│   ├── data/
│   │   ├── datasource/      ← LocalDataSource, SQLDataSource, S3DataSource, HFDataSource
│   │   ├── dataset/         ← Dataset implementations per source
│   │   └── converter/       ← Format converters (JSON, JSONL, CSV, Parquet, etc.)
│   ├── model/
│   │   ├── model.py         ← Model registry (rule_register, llm_register)
│   │   ├── rule/            ← Rule-based evaluators (80+ built-in)
│   │   │   ├── base.py      ← BaseRule
│   │   │   ├── rule_common.py ← Common rules (text quality, format, PII, etc.)
│   │   │   └── utils/       ← Shared utilities (normalize, ngrams, etc.)
│   │   └── llm/             ← LLM-based evaluators
│   │       ├── base_openai.py ← BaseOpenAI (base class for all LLM evaluators)
│   │       ├── text_quality/  ← Text quality evaluators (V4, V5)
│   │       ├── rag/          ← RAG metrics (Faithfulness, Precision, Recall, etc.)
│   │       ├── hhh/          ← 3H evaluators (Honest, Helpful, Harmless)
│   │       ├── compare/      ← Document comparison evaluators
│   │       └── agent/        ← Agent-based evaluators
│   │           ├── base_agent.py  ← BaseAgent
│   │           ├── tools/         ← Tool registry + implementations
│   │           ├── agent_fact_check.py
│   │           └── agent_hallucination.py
│   ├── exec/
│   │   ├── local.py         ← LocalExecutor (single machine, cross-layer conflict detection)
│   │   └── spark.py         ← SparkExecutor (distributed)
│   └── run/
│       └── cli.py           ← CLI entry point (subcommands: eval, info)
│
├── examples/                ← Usage examples (SDK, CLI, various scenarios)
├── test/                    ← Test suite
│   ├── data/                ← Test data files
│   ├── env/                 ← Test environment configs
│   └── scripts/             ← Test scripts (pytest)
└── docs/                    ← Documentation
```

## Core Concepts

### Data Flow

```
Data Input → Interface (SDK/CLI/MCP) → Datasource → Dataset → Converter → Evaluator → Executor → Report
```

### Registration System

All evaluators use decorator-based registration:

```python
# Rule evaluator
@Model.rule_register('QUALITY_BAD_COMPLETENESS', ['default', 'pretrain'])
class MyRule(BaseRule):
    @classmethod
    def eval(cls, input_data: Data) -> EvalDetail: ...

# LLM evaluator
@Model.llm_register('MyLLMEvaluator')
class MyLLMEvaluator(BaseOpenAI):
    prompt = "..."
    @classmethod
    def build_messages(cls, input_data: Data) -> List: ...

# Agent evaluator
@Model.llm_register('MyAgent')
class MyAgent(BaseAgent):
    available_tools = ["tavily_search"]
    @classmethod
    def eval(cls, input_data: Data) -> EvalDetail: ...
```

### Data Model

`Data` uses `extra = "allow"` — any field can be set dynamically:

```python
class Data(BaseModel):
    class Config:
        extra = "allow"
```

Common fields: `data_id`, `prompt`, `content`, `image`, `context`, `raw_data`, `reference`, `user_input`, `response`, `retrieved_contexts`.

### InputArgs Configuration

```python
input_data = {
    "input_path": "data.jsonl",
    "dataset": {"source": "local", "format": "jsonl"},
    "executor": {"max_workers": 4, "batch_size": 10, "result_save": {"bad": True, "good": True}},
    "evaluator": [
        {
            "fields": {"content": "text_field", "prompt": "question_field"},
            "evals": [
                {"name": "RuleAbnormalChar"},

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MigoXLab/dingo](https://github.com/MigoXLab/dingo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
