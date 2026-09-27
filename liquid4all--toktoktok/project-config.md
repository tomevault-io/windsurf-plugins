---
trigger: always_on
description: TokTokTok is a high-performance Byte-Pair Encoding (BPE) tokenizer trainer that produces vocabularies compatible with OpenAI's tiktoken library.
---

# TokTokTok BPE Tokenizer Trainer Specification

TokTokTok is a high-performance Byte-Pair Encoding (BPE) tokenizer trainer that produces vocabularies compatible with OpenAI's tiktoken library.

## Overview

The trainer uses an iterative, multi-phase approach where each phase can train on different data sources with a specified merge budget. The output is a `.tiktoken` file that can be loaded directly by the tiktoken library.

### Key Design Principles

- **Memory-bounded operation**: Training operates within a configurable memory budget using reservoir sampling
- **Multi-threaded**: Parallel pair counting and merging across available CPU cores
- **Iterative processing**: Files are streamed and processed incrementally, not loaded entirely into memory
- **Phase-based training**: Multiple training phases allow controlled vocabulary allocation across different data domains

## Vocabulary Structure

The final vocabulary is composed of:

```
Total Vocab = 256 (base bytes) + 1,161 (hardcoded merges) + Σ(phase merges) + special tokens
```

| Component | Token ID Range | Description |
|-----------|---------------|-------------|
| Base bytes | 0-255 | Raw byte values (UTF-8 compatible) |
| Hardcoded merges | 256-1416 | Pre-defined merges for numbers, whitespace, operators |
| Trained merges | 1417+ | Corpus-learned merges from training phases |
| Special tokens | End of vocab | User-defined special tokens |

## Configuration Format (YAML)

Training is configured via a YAML file. Run with: `toktoktok -c config.yaml`

### YAML Schema

```yaml
# Output file path for the trained vocabulary
output: <path>                    # Required: .tiktoken output file

# Memory management
working_set_mb: <integer>         # Optional: Max memory in MB (default: 1024)

# Parallelization
threads: <integer>                # Optional: Thread count, -1 for auto (default: -1)

# Logging
verbose: <boolean>                # Optional: Detailed logging (default: false)

# Special tokens (added at end of vocabulary)
special_tokens:                   # Optional: List of special token strings
  - "<token1>"
  - "<token2>"

# Training phases (executed sequentially)
phases:                           # Required: At least one phase
  - name: <string>                # Required: Phase name for logging
    merges: <integer>             # Required: Number of merges for this phase
    sources:                      # Required: At least one source
      - path: <directory>         # Directory (recursive scan for .txt/.parquet)
      - file: <filepath>          # OR single file path
```

### Example Configuration

```yaml
output: ./my_tokenizer.tiktoken
working_set_mb: 4096
threads: -1
verbose: false

special_tokens:
  - "<|endoftext|>"
  - "<|pad|>"
  - "<|startoftext|>"
  - "<|im_start|>"
  - "<|im_end|>"

phases:
  - name: "English General"
    merges: 30000
    sources:
      - path: /data/corpus/english/wikipedia
      - path: /data/corpus/english/books
      - file: /data/corpus/english/common_crawl.parquet

  - name: "Programming"
    merges: 15000
    sources:
      - path: /data/corpus/code/python
      - path: /data/corpus/code/javascript

  - name: "Multilingual"
    merges: 5000
    sources:
      - path: /data/corpus/german
      - path: /data/corpus/french
```

## Supported Input Formats

| Format | Extension | Description |
|--------|-----------|-------------|
| Plain text | `.txt` | UTF-8 encoded text files |
| Parquet | `.parquet` | Apache Parquet files with a `text` column |

When specifying a directory path, the trainer recursively scans for all `.txt` and `.parquet` files.

## Tokenization Regex

The trainer uses the GPT-4 / cl100k_base compatible regex pattern for pre-tokenization:

```regex
(?i:'s|'t|'re|'ve|'m|'ll|'d)|[^\r\n\p{L}\p{N}]?\p{L}+|\p{N}{1,3}| ?[^\s\p{L}\p{N}]+[\r\n]*|\s*[\r\n]+|\s+(?!\S)|\s+
```

### Pattern Breakdown

| Pattern | Description | Examples |
|---------|-------------|----------|
| `(?i:'s\|'t\|'re\|'ve\|'m\|'ll\|'d)` | English contractions (case-insensitive) | `'s`, `'T`, `'re`, `'VE` |
| `[^\r\n\p{L}\p{N}]?\p{L}+` | Optional non-letter/digit followed by letters | `Hello`, ` world`, `.test` |
| `\p{N}{1,3}` | 1-3 digit numbers | `1`, `42`, `999` |
| ` ?[^\s\p{L}\p{N}]+[\r\n]*` | Punctuation with optional leading space | ` !!!`, `...`, `->` |
| `\s*[\r\n]+` | Whitespace before newlines | `\n`, `  \n\n` |
| `\s+(?!\S)` | Trailing whitespace | Spaces at end of text |
| `\s+` | Other whitespace | Spaces between words |

This regex ensures tokenization boundaries match tiktoken's behavior for seamless compatibility.

## Hardcoded Merges

Before corpus-based training begins, 1,161 merges are applied for common patterns. These ensure efficient encoding of numbers, whitespace, and programming constructs regardless of training data.

### Hardcoded Token Categories

#### Two-Digit Numbers (100 merges, IDs 256-355)
All combinations `"00"` through `"99"`, each formed by merging two single-digit byte tokens.

| Token | ID | Composition |
|-------|-----|-------------|
| `"00"` | 256 | `'0'` + `'0'` |
| `"01"` | 257 | `'0'` + `'1'` |
| ... | ... | ... |
| `"99"` | 355 | `'9'` + `'9'` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Liquid4All/toktoktok](https://github.com/Liquid4All/toktoktok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
