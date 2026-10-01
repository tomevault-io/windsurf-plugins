---
trigger: always_on
description: Manages downloading and access to HuggingFace repositories.
---

# Developer Summary for AI Agents (`go-huggingface`)

Welcome to the `go-huggingface` codebase. This document is a comprehensive guide for AI developer agents modifying, extending, or debugging this repository.

---

## 1. Project Overview & Architectural Philosophy

[`go-huggingface`](file:///home/janpf/Projects/gomlx/go-huggingface) provides Go-native tools to interact with [HuggingFace](https://huggingface.co):
- **Hub Client (`hub`)**: Download and cache model, dataset, or generic repository files, inspect repo metadata, or work with local and embedded files.
- **Tokenizers (`tokenizers`)**: Pure Go implementations of HuggingFace tokenizers (WordPiece, BPE, Unigram) reading directly from `tokenizer.json`, plus SentencePiece model support.
- **Streaming Bucketing (`tokenizers/bucket`)**: Grouping sentences by length into discrete buckets (Power-of-2, Two-Bits) with padding minimization and latency limits.
- **Model Formats & Execution (`models/`)**: 
  - Safetensors parsing and zero-copy memory-mapped loading into GoMLX backends (`models/safetensors`).
  - GGUF binary format parsing and pure-Go dequantization (`models/gguf`).
  - HuggingFace transformer config translation to GoMLX computation graphs (`models/transformer`).
  - Segment Anything 2 (SAM2) image backbone and mask decoder (`models/sam2`).
- **Parquet Datasets (`datasets`)**: On-demand downloading, schema inspection, and stream-reading of Parquet-backed HuggingFace datasets with automatic Go struct generation.

### Modularity & Dependency Decoupling
The packages are strictly decoupled:
- `hub` has **no** dependency on GoMLX or Parquet.
- `tokenizers` has **no** dependency on GoMLX, Parquet, or external CGo/Python libraries (unless using optional third-party wrappers like `daulet/tokenizers`).
- GoMLX (`github.com/gomlx/gomlx` and `github.com/gomlx/compute`) is only imported by `models/*` and example packages.

---

## 2. Package Directory Map

```
go-huggingface/
├── cmd/
│   ├── dataset_download/        # CLI tool to inspect, download, list, and delete dataset files
│   ├── generate_dataset_structs/ # CLI tool to generate Go structs from Parquet dataset schemas
│   └── hubinfo/                 # CLI tool to inspect metadata, save local copies, and manage cache
├── datasets/                    # HuggingFace datasets client & Parquet scanning
├── docs/
│   └── CHANGELOG.md             # Project change log (MANDATORY UPDATE ON EVERY FIX)
├── examples/
│   ├── BAAI-bge-small-en-v1.5/  # Example & test suite for BERT-based sentence embeddings
│   ├── gemma4-e4bit/            # Gemma 4 E4B model loading example
│   ├── kalmgemma3/              # KaLM-Gemma3 12B sentence embedding example & tests
│   └── msmarco/                 # MS MARCO dataset reader and embedding benchmark
├── hub/                         # Core HuggingFace Hub client (remote, local, and embed modes)
├── internal/
│   ├── downloader/              # Parallel download manager with FIFOSemaphore & atomic file locking
│   ├── files/                   # File utilities (tilde expansion, flock-based file locking)
│   ├── py/                      # Python reference scripts for generating golden test tensors
│   └── testing/                 # Shared test validation utilities (tensor comparison tolerances)
├── models/
│   ├── gguf/                    # GGUF parser, metadata extraction & k-quant dequantization
│   ├── image/                   # Image preprocessing graph operations (resize, normalize)
│   ├── safetensors/             # Safetensors reader, mmap integration & tensor iterators
│   ├── sam2/                    # Segment Anything 2 (SAM2) model architecture & Segmenter API
│   └── transformer/             # Transformer graph builder for GoMLX
└── tokenizers/
    ├── api/                     # Tokenizer interface, TokenSpan, AnnotatedEncoding, EncodeOptions
    ├── bucket/                  # Sentence length bucketing (TwoBitBucket, ByPower, ByPowerBudget)
    ├── hftokenizer/             # Native Go tokenizer engine parsing tokenizer.json (BPE/WordPiece/Unigram)
    └── sentencepiece/           # SentencePiece processor wrapper (via eliben/go-sentencepiece)
```

---

## 3. Public Packages Deep-Dive

### 3.1. `hub`
Manages downloading and access to HuggingFace repositories.
- **Three Repository Modes**:
  1. **Remote (`hub.New(id)`)**: Downloads from HuggingFace Hub into the local cache (`~/.cache/huggingface/hub/` or `$XDG_CACHE_HOME/huggingface/hub/`).
  2. **Local (`hub.NewLocal(dir)`)**: Reads directly from a local folder on disk without any network requests. Useful for offline environments, `git clone` checkouts, or Docker images.
  3. **Embedded (`hub.NewEmbed(fsys, subDir)`)**: Reads directly from an `fs.FS` (e.g., `//go:embed`). Operates fully in-memory.
- **Preferred File Access APIs**:
  - Use `repo.Open(fileName)` (returns `fs.File`) or `repo.ReadFile(fileName)` (returns `[]byte`). These stream data directly from memory for embedded repositories without disk writes.
  - Avoid `repo.DownloadFile(s)` unless an OS disk path string is strictly necessary: for embedded repos, `DownloadFile` forces temporary extraction into `os.TempDir()`.
  - Use `repo.FetchFiles(...)` to pre-download/cache files without requesting their disk paths.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gomlx/go-huggingface](https://github.com/gomlx/go-huggingface) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
