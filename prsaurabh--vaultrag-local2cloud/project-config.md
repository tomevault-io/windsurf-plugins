---
trigger: always_on
description: DocuAssist is a Retrieval-Augmented Generation (RAG) system - Document Assistant with dual-mode operation. It combines OCR, vector search, and LLM generation with **dual-mode operation**: private (local models) or public (AWS Bedrock).
---

# DocuAssist Copilot Instructions

## Project Overview
DocuAssist is a Retrieval-Augmented Generation (RAG) system - Document Assistant with dual-mode operation. It combines OCR, vector search, and LLM generation with **dual-mode operation**: private (local models) or public (AWS Bedrock).

## Architecture & Data Flow

```
User Query → Retrieval (Hybrid Search) → Generation (LLM) → Response
                ↓
    1. Metadata Extraction (SpaCy/LLM)
    2. ChromaDB Vector Search (dense)
    3. BM25 Keyword Search (sparse)
    4. Ensemble Weighting & Deduplication
                ↓
            Context → AWS Bedrock Llama 3 (or local for private mode)
```

### Core Components
- **[api/main.py](api/main.py)**: FastAPI application with 4 endpoints (load, retrieve, generate, chat)
- **[Retrival.py](Retrival.py)**: Hybrid ensemble retrieval (ChromaDB + BM25) with metadata filtering
- **[Generation.py](Generation.py)**: LLM generation with conversation memory and PII masking (Presidio)
- **[DataLoader_Chunking.py](DataLoader_Chunking.py)**: OCR + chunking strategies (semantic/spacy/recursive)
- **[Config.py](Config.py)**: Pydantic Settings with `.env` loading

## Critical Patterns

### 1. Dual Mode Pattern (Private vs Public)
**Every core function** accepts a `private` boolean flag:
```python
# Private mode: Local models (SentenceTransformerEmbeddings, no AWS)
# Public mode: AWS Bedrock (BedrockEmbeddings, Llama 3)

if private:
    embedding_function = SentenceTransformerEmbeddings(model_name=config.sbert_model)
else:
    embedding_function = BedrockEmbeddings(client=client, model_id="amazon.titan-embed-text-v1")
```
**When modifying**: Always preserve both code paths. Never assume AWS availability.

### 2. Hybrid Retrieval with Weighted Scoring
[Retrival.py](Retrival.py) implements ensemble search with **hardcoded weights**:
```python
chroma_metadata_weight = 0.4      # ChromaDB with metadata filter
chroma_no_metadata_weight = 0.3   # ChromaDB without filter (fallback)
bm25_weight = 0.3                 # BM25 keyword search
```
- Scores are normalized and deduplicated via `seen_texts` set
- **Always** has a fallback chunk even if all searches fail
- Metadata extraction uses SpaCy (private) or LLM (public)

### 3. Chunking Strategies
Three strategies in [DataLoader_Chunking.py](DataLoader_Chunking.py):
- `semantic`: LangChain's `SemanticChunker` with embeddings (default, **6 chunks**, 0.7 percentile threshold)
- `spacy`: SpaCy sentence-based chunking (3-way split)
- `recursive`: LangChain's `RecursiveCharacterTextSplitter` (1000 chars, 200 overlap)

### 4. PII Protection Flow
[Generation.py](Generation.py) uses Presidio for anonymization:
1. `mask_pii()`: Detects entities, creates unique placeholders (`<PERSON_1>`), stores in global `pii_mapping`
2. Send masked prompt to LLM
3. `unmask_pii()`: Replaces placeholders with original values from mapping

**Important**: Uses global state (`pii_mapping`, `entity_counter`) - not thread-safe.

### 5. Configuration System
[Config.py](Config.py) uses Pydantic Settings with **backwards compatibility aliases**:
```python
# New style: config.sbert_model
# Old style: config.SBERT_MODEL (property alias)
```
**All paths are relative** to project root (`./Data/Images/`, `./Chroma/chroma_db`).

## Development Workflows

### Running Locally
```bash
# Windows quick start
start_api.bat

# Linux/Mac
chmod +x start_api.sh && ./start_api.sh

# Manual (development with reload)
uvicorn api.main:app --reload --host 0.0.0.0 --port 8000
```

### Docker Deployment
```bash
docker-compose up -d
```
- Mounts `./Data`, `./Chroma`, `./models` for persistence
- Sets `TESSERACT_PATH=/usr/bin/tesseract`
- Health check: `/health` endpoint

### Testing API
Interactive docs at `http://localhost:8000/api/v1/docs`

Example chat request:
```bash
curl -X POST "http://localhost:8000/api/v1/chat" \
  -H "Content-Type: application/json" \
  -d '{"query": "Who is the buyer?", "k": 5, "private_mode": false}'
```

### Gradio UI (Legacy)
```bash
python ChatBot_UI.py
# Two modes: "Private" (local) or "Public" (AWS Bedrock)
```

## Critical Files to Review Before Modifying

| Task | Read First | Why |
|------|-----------|-----|
| Add endpoint | [api/main.py](api/main.py), [api/models.py](api/models.py) | Understand request/response validation, error handling |
| Change retrieval logic | [Retrival.py](Retrival.py) lines 17-100 | Ensemble weighting, metadata filtering, deduplication |
| Modify chunking | [DataLoader_Chunking.py](DataLoader_Chunking.py) lines 70-150 | SemanticChunker config, chunking strategies |
| Update config | [Config.py](Config.py), `.env` | Environment variables, backwards compatibility |
| Fix AWS issues | [Config.py](Config.py) lines 49-60 | `validate_aws_credentials()` method |

## Common Gotchas

1. **Hardcoded AWS Credentials**: Found in [DataLoader_Chunking.py](DataLoader_Chunking.py), [Generation.py](Generation.py), [Retrival.py](Retrival.py). Should use `config.aws_access_key_id` instead.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prsaurabh/VaultRAG-local2cloud](https://github.com/prsaurabh/VaultRAG-local2cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
