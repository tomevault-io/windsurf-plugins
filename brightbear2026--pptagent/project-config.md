---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PPT Agent generates consulting-grade `.pptx` presentations from text/documents. The system emphasizes high information density, professional visual design, and transparent human control at 2 mandatory checkpoints.

**Core Goals:**
- Argument-driven outlines (Pyramid Principle), not document chapter mapping
- Support multiple input formats: DOCX / XLSX / CSV / PPTX / TXT / MD
- 2 mandatory user review checkpoints (outline + content)
- Quality over speed: 3-10 minute generation is acceptable

## Architecture

### 6-Agent Pipeline

```
parse → analyze → outline[checkpoint1] → content[checkpoint2] → design → render
```

Each agent's result is persisted in `pipeline_stages` PostgreSQL table. Users can edit any completed stage and re-run from that point via `POST /api/task/{id}/resume`.

| Stage | Agent | Description |
|-------|-------|-------------|
| `parse` | ParseAgent (code-only) | DOCX/XLSX/CSV/PPTX/TXT/MD → RawContent |
| `analyze` | AnalyzeAgent (LLM) | Document strategy + audience analysis + **chunk generation** |
| `outline` | **PlanAgent** (LLM) | Pyramid Principle → DeckPlan (framework structure + argument slides) |
| `content` | ContentAgent (LLM, per-slide parallel) | Each slide's text blocks + chart/diagram specs |
| `design` | **HTMLDesignAgent** (LLM, default) or DesignAgent | HTML-first rendering path; fallback to legacy python-pptx |
| `render` | html2pptx.js + ChartRenderer (default) or RenderAgent | Node/Playwright → pptxgenjs + native chart injection |

### 2 Mandatory Checkpoints

```
CHECKPOINT_AGENTS = {"outline", "content"}
```

| # | Checkpoint | After Stage | User Reviews | User Can Edit |
|---|---|---|---|---|
| 1 | Outline Confirmation | `outline` | SCQA structure, all slides with takeaway messages, narrative flow | Edit any field, reorder, delete pages |
| 2 | Content Confirmation | `content` | Per-slide text blocks, chart data, diagram specs | Edit text, modify chart data, rerun single page |

After checkpoint 2: design → render run automatically without pausing.

### PlanAgent — Pyramid Principle Outline (key design)

**Why PlanAgent exists**: The old OutlineAgent used `recommended_structure` (a natural-language string like "开篇→问题→分析→方案→行动") split by "→" to allocate pages uniformly. This produced chapter-mapping outlines ("materials decomposition") instead of argument sequences.

**PlanAgent approach**:
- User's `scenario` choice (季度汇报/战略提案/竞标pitch etc.) hard-maps to a narrative framework (SCR/SCQA/AIDA), not LLM-inferred
- System prompt emphasizes: "PPT is an argument, not a table of contents; each slide's takeaway_message must be a complete sentence with a verb"
- Outputs SCQA structure + slides array where each slide has a clear claim (action title)
- Rule-based verification + 1 LLM fix pass for quality

**Output format** (compatible with existing ContentAgent):
```json
{
  "narrative_logic": "SCQA框架：...",
  "scqa": {"situation": "...", "complication": "...", "question": "...", "answer": "..."},
  "root_claim": "顶层结论",
  "items": [OutlineItem dicts...],
  "data_gap_suggestions": []
}
```

### Data Model

`SlideSpec` is the core object flowing through design → render. Key fields set by each layer:

| Layer | Fields Set |
|-------|-----------|
| parse | RawContent (source_pages, tables, raw_text) |
| analyze | StrategyInsight, chunks (for PlanAgent), derived_metrics |
| outline | OutlineItem list (slide_type, takeaway_message, supporting_hint, narrative_arc) |
| content | SlideContent (text_blocks, chart_suggestion, diagram_spec, visual_block) |
| design | SlideSpec fully populated (layout_template, visual_theme, charts, diagrams) |
| render | .pptx file |

## Technology Stack

| Component | Technology |
|-----------|-----------|
| Backend | FastAPI + Uvicorn |
| Frontend | React 18 + TypeScript + Vite + Ant Design |
| Database | PostgreSQL (Alembic migrations) |
| LLM | Any OpenAI-compatible API (SiliconFlow / DeepSeek / Tongyi / Zhipu etc.) |
| PPT Generation | python-pptx (native vector chart objects) |
| Encryption | Fernet + PBKDF2HMAC per-user key derivation |
| Deployment | Docker + docker-compose |

## LLM Configuration

All providers use OpenAI-compatible protocol via `base_url` switch. Per-stage model config allows different models per pipeline stage.

### Provider Config in `models/model_config.py`

```python
class StageModelConfig:
    provider: str      # "openai_compat" | "zhipu"
    model: str
    api_key: str       # encrypted in DB; plaintext in memory only
    base_url: str
    temperature: float
    max_tokens: int

class PipelineModelConfig:
    # Per-stage configs; get_stage_config(stage) returns the right one
    analyze: StageModelConfig
    outline: StageModelConfig
    content: StageModelConfig
    design: StageModelConfig
```

### LLM Client Architecture

```
llm_client/
  base.py          # LLMClient ABC: chat(messages, tools, ...) -> ChatResponse
  openai_compat.py # OpenAI-compatible adapter (covers DeepSeek, Tongyi, SiliconFlow, etc.)
  glm_client.py    # Zhipu GLM proprietary SDK adapter
  factory.py       # get_client(provider, ...) -> LLMClient
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brightbear2026/PPTagent](https://github.com/brightbear2026/PPTagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
