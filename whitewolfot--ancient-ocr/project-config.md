---
trigger: always_on
description: This file tells Claude Code everything it needs to build and extend this project correctly.
---

# CLAUDE.md — Ancient-OCR (Arabic OCR + Lexicon-Augmented Intelligence)

This file tells Claude Code everything it needs to build and extend this project correctly.
Read it fully before writing any code. Follow the rules below without exception.

---

## What this system does

Ingests scanned Arabic PDFs/images → runs an OCR ensemble → normalizes noisy output → resolves uncertain words using classical Arabic lexicons + morphology → returns corrected Arabic text with full per-token provenance.

Three output modes: `clean` | `annotated` | `debug`.
Two interfaces: FastAPI API + CLI. No separate frontend required.

---

## Repo structure

```
/
├── CLAUDE.md                  ← you are here
├── main.py                    ← pipeline orchestrator (wire-up only)
├── config.yaml                ← all settings; no magic numbers in code
├── requirements.txt
├── pyproject.toml             ← optional extras: [layout], [trocr], [lm]
├── README.md
├── io/                        ← document_loader.py (PDF+image → pages)
├── preprocessing/             ← image_pipeline, deskew, thresholding, denoise, layout_detection
├── ocr_engine/                ← schema, base, paddle, tesseract, trocr, ensemble
├── alignment/                 ← token_matcher, bbox_alignment, string_similarity
├── normalization/             ← arabic_normalizer, noise_filter
├── morphology/                ← root_extractor, pattern_analyzer, camel_adapter
├── lexicon_ingestion/         ← sources, parser, storage, index_builder
├── lexicon_engine/            ← query_engine, candidate_generator, context_scorer, scorer, ranker
├── confidence_engine/         ← scoring, decision, state
├── output/                    ← formatter, json_export, markdown_export, debug_export
├── api/                       ← server.py, routes.py, schemas.py
├── cli/                       ← main.py, commands.py
├── utils/                     ← config.py, logging.py, cache.py
├── eval/                      ← metrics.py, benchmark.py
├── tests/
├── data/                      ← lexicon raw + built indexes (gitignored)
└── models/                    ← downloaded/finetuned weights (gitignored)
```

---

## Build order — mandatory, do not skip phases

```
1.  Skeleton: all __init__.py, config.yaml, utils/config.py, utils/logging.py
2.  main.py orchestrator (safe imports — fail with clear message if module missing)
3.  Shared contracts: ocr_engine/schema.py + confidence_engine/state.py  ← build first, never change shape
4.  io/document_loader.py
5.  preprocessing/: denoise → thresholding → deskew → layout_detection → image_pipeline
6.  ocr_engine/: base → paddle_backend → tesseract_backend → trocr_backend
7.  alignment/: string_similarity → bbox_alignment → token_matcher
8.  ocr_engine/ensemble.py (uses alignment)
9.  normalization/: noise_filter → arabic_normalizer
10. lexicon_ingestion/: sources → parser → storage → index_builder
11. morphology/: root_extractor → pattern_analyzer → camel_adapter
12. lexicon_engine/: query_engine → candidate_generator → context_scorer → scorer → ranker
13. confidence_engine/: scoring → decision  (state already done in step 3)
14. output/: formatter → json_export → markdown_export → debug_export
15. api/: server → routes → schemas
16. cli/: main → commands
17. tests/ + eval/
18. README.md
```

---

## Shared data contracts — define once, obey everywhere

Use **Pydantic v2**. All models must be JSON-serializable. Build these in step 3 and never change the field names downstream.

```python
class WordToken(BaseModel):
    text: str
    confidence: float           # 0..1
    bbox: tuple[int,int,int,int]  # x, y, w, h
    page_index: int
    source: str                 # "paddle"|"tesseract"|"trocr"|"ensemble"

class OCRResult(BaseModel):
    text: str
    words: list[WordToken]
    confidence: float
    page_index: int
    source: str
    raw: dict = {}              # engine-specific payload for debug

class LexiconEntry(BaseModel):
    lemma: str
    root: str | None
    pattern: str | None
    gloss: str
    source: str
    era: str                    # "classical" | "modern"
    domain: str | None
    examples: list[str] = []
    priority: int

class Candidate(BaseModel):
    text: str
    reason: str                 # "spelling_variant"|"normalization"|"root_alt"|"morph_alt"|"identity"
    lexicon_entries: list[LexiconEntry] = []
    features: dict = {}         # lexicon_score, morph_score, ocr_score, context_score
    score: float | None = None

class TokenState(BaseModel):
    original: str
    normalized: str
    normalization_log: list[dict]        # per-step changes (traceability)
    candidates: list[Candidate]
    selected: str
    confidence: float
    sources: list[str]
    decision: str               # "accept"|"accept_with_note"|"uncertain"|"review_required"
    reason_code: str
```

---

## Non-negotiable design rules

1. **Never overwrite a token without storing the original + alternatives.**
2. **Every correction must carry its source(s) and a reason code.**
3. **Lexicon evidence > morphology > model guessing.** Morphology supports; never overrules.
4. **Raw per-engine OCR stays accessible throughout** (needed for debug mode).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whitewolfOT/Ancient-OCR](https://github.com/whitewolfOT/Ancient-OCR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
