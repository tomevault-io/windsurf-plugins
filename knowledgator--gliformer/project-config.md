---
trigger: always_on
description: Multi-task information extraction model built on top of GLiNER, supporting:
---

# GLiNExT — Multi-task Information Extraction Framework

Multi-task information extraction model built on top of GLiNER, supporting:
- Named-entity recognition (NER)
- Relation extraction (open & joint)
- Text classification
- Embedding (pairwise similarity)
- Grouping / JSON schema extraction (structuring)
- Instance counting

## Architecture Overview

### Three-component task design

Each task is a self-contained module with three components:
- **Processor** — data preparation, prompt contribution, label creation
- **Model (Head)** — task-specific neural layers, forward pass, loss computation
- **Decoder** — post-processing logits into structured predictions

### High-level orchestration

Four top-level classes coordinate the task modules:

| Class | File | Role |
|-------|------|------|
| `GLiNExT` | `glinext.py` | User-facing API: inference, training, model loading (extends `BaseEncoderGLiNER`) |
| `GLiNExTModel` | `model.py` | Shared encoder + task heads orchestrator (`nn.Module`) |
| `GLiNextProcessor` | `processor.py` | Delegates prompt/label work to per-task processors |
| `GLiNExTDecoder` | `decoder.py` | Factory assembling per-task decoders from config |
| `GLiNExTDataCollator` | `collator.py` | Bridges raw data → model-ready batches via processor |
| `GLiNExTSchema` | `schema.py` | Fluent builder for multi-task inference schemas |
| `GLiNExTTrainer` | `training.py` | Extends GLiNER Trainer for multi-task label handling |

### Model flow

1. **Encode text** — shared transformer backbone (`Encoder` or `BiEncoder`; task-specific labels batched into a single `encode_labels()` pass)
2. **Extract task-specific prompt embeddings** — [ENT], [CAT], [REL], [PARENT], [CHILD] tokens from encoder output
3. **Route to task-specific heads** — each head receives `SharedRepresentations` (word embeddings + prompt embeddings)
4. **Decode** — general method assembles outputs from all active heads into `GLiNExTOutput`

### Head execution order & dependencies

```
NER (no deps)
Joint Relex (no external deps — NER is built-in via inheritance from NERHead)
Open Relex, Classification, Count, Structuring, Embedding (no deps, run independently)
```

## Abstract Base Classes (`tasks/__init__.py`)

| ABC | Purpose | Key abstract method |
|-----|---------|-------------------|
| `TaskHead` | Neural module base | `forward(shared, dependency_outputs, **batch)` |
| `TaskProcessor` | Data processor base | `get_classes_mapping()`, `create_labels()` |
| `TaskDecoder` | Output decoder base | `decode(model_output, classes_mapping)` |

### Shared base classes

Extraction tasks share span resolution and decoding via intermediate bases:

```
TaskProcessor (ABC)                    TaskDecoder (ABC)
└── SpanProcessor                      └── SpanDecoder
    ├── NERProcessor                       ├── NERDecoder
    │   └── JointRelexProcessor            │   └── JointRelexDecoder
    ├── OpenRelexProcessor                 ├── OpenRelexDecoder
    └── StructuringProcessor               └── StructuringDecoder
```

**SpanProcessor** (`tasks/span_processor.py`):
- `_resolve_entity_spans()` — text mention → token index resolution
- `_resolve_text_span()` — single mention resolution
- `_tokenize_text()` — tokenize + cache
- `_generate_negative_spans()` — random non-overlapping negative sampling
- `_collect_span_candidates()` — positive + negative span collection

**SpanDecoder** (`tasks/span_decoder.py`):
- `decode_bio_spans()` — single-sample BIO (L, C, 3) → List[Span]
- `decode_bio_spans_single_class()` — single-class BIO (L, 3) → List[Span]
- `decode_bio_spans_batch()` — batched BIO decoding
- `decode_span_level()` — pre-computed span representations decoding
- `greedy_search()` — overlap removal (flat/nested modes)
- `resolve_span_text()` — token indices → text

## Task Modules

| Module | Path | Description |
|--------|------|-------------|
| **NER** | `tasks/ner/` | Named entity recognition via span scoring |
| **Joint Relex** | `tasks/joint_relex/` | Joint NER + relation extraction; inherits NERHead, adjacency-based entity pair scoring (GLiNER-relex style) |
| **Open Relex** | `tasks/open_relex/` | Anchor-based relation extraction; dual AnchoredSpanScorers, no NER dependency (GLiNER2-style) |
| **Classification** | `tasks/classification/` | Text classification via anchor paradigm + pooling (GLiClass-style) |
| **Embedding** | `tasks/embedding/` | Text pair similarity with configurable pooling and loss |
| **Structuring** | `tasks/structuring/` | Group entities into clusters for JSON schema extraction (GLiNER2-style) |
| **Count** | `tasks/count/` | Predict instance counts per parent group |

## File Structure

```
glinext/
├── __init__.py        — Public API re-exports
├── glinext.py         — GLiNExT user-facing class (inference, training, model I/O)
├── config.py          — GLiNextConfig with per-task sub-configs (NERHeadConfig, etc.)
├── model.py           — GLiNExTModel orchestrator, SharedRepresentations, GLiNExTOutput
├── training.py        — GLiNExTTrainer (multi-task label handling, OOM recovery)
├── utils.py           — Utilities
├── processing/        — Data preparation, collation, decoding, schema
│   ├── __init__.py
│   ├── processor.py       — GLiNextProcessor orchestrator, per-task processor delegation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Knowledgator/GLiFormer](https://github.com/Knowledgator/GLiFormer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
