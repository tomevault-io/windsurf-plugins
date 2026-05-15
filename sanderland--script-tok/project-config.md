---
trigger: always_on
description: This guide provides a high-level architectural overview of the script_bpe repository to help AI Coding Assistants work effectively in this codebase.
---

# AGENTS.md - Codebase Architecture Guide

This guide provides a high-level architectural overview of the script_bpe repository to help AI Coding Assistants work effectively in this codebase.

## Project Overview

This repository implements **SCRIPT encoding-based pre-tokenization and BPE/Unigram tokenization** for multilingual text processing. The core innovation is using Unicode script properties to create more efficient and robust tokenizers for multilingual data.

**Papers**:
- [BPE Stays on SCRIPT: Structured Encoding for Robust Multilingual Pretokenization](https://arxiv.org/abs/2505.24689)
- [Which Pieces Does Unigram Tokenization Really Need?](https://arxiv.org/abs/2512.12641)

## High-Level Architecture

The codebase follows a **layered architecture** with clear separation of concerns:

```
Text Input
    ↓
[Pretokenization Layer] - Chunks text and encodes to base tokens
    ↓
[Corpus Layer] - Pretokenized, partitioned training data
    ↓
[Training Layer] - BPE or Unigram algorithm
    ↓
[Tokenizer Layer] - Trained model for encode/decode
```

## Module Structure

### 1. Pretokenization (`script_bpe/pretokenize/`)

**Purpose**: Transform raw text into sequences of "base tokens" (atomic units for tokenization)

**Key Concepts**:
- **Base tokens**: Atomic units - either bytes (UTF-8) or script/index pairs
- **Pretokenization**: Chunking text into segments before encoding
- **Encoding**: Converting characters to base token sequences

**Two Main Approaches**:

1. **UTF-8 Pretokenizer** (`UTF8Pretokenizer`):
   - Encodes each byte as a base token
   - Uses regex patterns (GPT-4, GPT-4o) for chunking
   - Base token = single byte

2. **Script Pretokenizer** (`ScriptPretokenizer`):
   - Uses Unicode script properties to encode characters
   - Each character → (script_block_id, index_within_block)
   - Base token = pair of script tokens
   - Can optionally chunk by script boundaries instead of regex

**Configuration System**:
- `PretokenizerConfig`: Base configuration (normalization, regex, etc.)
- `ScriptPretokenizerConfig`: Script-specific settings
- `UTF8PretokenizerConfig`: UTF-8-specific settings
- Registry in `pretokenize/__init__.py` maps names → configs

**Key Variants** (see `PRETOKENIZER_REGISTRY`):
- `bytes_gpt4`: Classic UTF-8 + GPT-4 regex
- `bytes_gpt4o_cb`: UTF-8 + GPT-4o regex + character boundaries
- `scriptenc_cb`: Script encoding with character boundaries (PROPOSED METHOD for BPE)
- `scriptenc_cbi`: Script encoding with character boundaries + inherited enforcement
- `scriptenc_gpt4o_cb`: Hybrid (regex chunking + script encoding)
- `scriptenc_nosplit_cb`: No regex chunking (very slow, for ablations)
- `scriptenc2_cb`, `scriptenc3_cb`: V2/V3 encoding variants

**Character Boundary Enforcement** (`enforce_char_boundaries`):
- Prevents merges that would create invalid UTF-8 sequences
- Three levels:
  - `False`: No restrictions (can create partial characters)
  - `True` (cb): Only complete characters allowed
  - `enforce_inherited=True` (cbi): Also prevents inherited scripts at start

**Script Encoding** (`scriptencoding.py`):
- `ScriptConfig`: Defines how Unicode scripts/categories map to blocks
- `ScriptBlock`: A group of characters sharing script+category
- Three encoding versions (`ScriptEncodingV1`, `ScriptEncodingV2`, `ScriptEncodingV3`):
  - V1: More granular script categories
  - V2: High-resource scripts get categories, low-resource collapse to ALL
  - V3: Like V2, but with broader space-combining behavior (all scripts get PSF)
- Special handling: Hiragana merged with Han, space-combining scripts
- Registry variants: `scriptenc_cb` (V1), `scriptenc2_cb` (V2), `scriptenc3_cb` (V3)

### 2. Corpus Management (`script_bpe/corpus/`)

**Purpose**: Efficiently store and access pretokenized training data

**Architecture**:
- `PretokenizedCorpus`: Represents a pretokenized dataset
  - Stored as partitioned Parquet files for parallel access
  - Format: `(chunk_bytes, count)` pairs
  - Metadata includes pretokenizer hash for validation
  
**Storage Structure**:
```
results/corpora/
  └── {corpus_name}/
      └── {pretokenizer_hash}/
          ├── metadata.json
          ├── part_0000.parquet
          ├── part_0001.parquet
          └── ...
```

**Workflow**:
1. Raw text → `from_texts()` → pretokenize with workers
2. Group identical chunks, count frequencies
3. Partition into ~128 files for parallel loading
4. Workers can iterate their partition independently

**Registry** (`registry.py`):
- `load_corpus_by_name()`: Lazy-loads or creates corpora
- Supports HuggingFace datasets (OSCAR, CulturaX, finewiki)
- `MONOLINGUAL_DATASETS`: List of 12 language-script pairs (e.g., `eng_latn_300mb`, `kor_hang_300mb`, `zho_hans_300mb`) with ~300MB each, ordered by bytes/char

### 3. Tokenizers (`script_bpe/tokenizers/`)

Two tokenization algorithms, sharing a common base:

#### Base Layer (`base.py`)
- `BaseToken`: Token with id and atomic_tokens sequence
- `BaseTokenizer`: Abstract interface for encode/decode/save/load
- `BaseTrainer`: Training configuration and logging
- Common stats: compression metrics, Renyi entropy, etc.

#### BPE Implementation (`tokenizers/bpe/`)

**Training** (`trainer.py`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanderland/script_tok](https://github.com/sanderland/script_tok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
