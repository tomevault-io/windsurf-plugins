---
trigger: always_on
description: DataTrove is a library to process, filter, and deduplicate text data at very large scale. It provides prebuilt pipeline blocks with a framework to add custom functionality. Pipelines are platform-agnostic, running locally, on Slurm, or on Ray clusters.
---

# DataTrove

DataTrove is a library to process, filter, and deduplicate text data at very large scale. It provides prebuilt pipeline blocks with a framework to add custom functionality. Pipelines are platform-agnostic, running locally, on Slurm, or on Ray clusters.

See [README.md](README.md) for detailed documentation on terminology, pipeline blocks, executors, and usage examples.

## Python Environment

- **Python Version:** 3.10+
- **Package Manager:** `uv` for dependency management
- **Virtual Environment:** `.venv/` (created via `uv venv`)

## Core Commands

```bash
uv venv --python 3.12
uv sync --all-extras          # install all optional dependencies
make quality                  # lint changed files (ruff check + format)
make style                    # auto-fix changed files
make quality-full             # lint entire repo
make style-full               # auto-fix entire repo
make test                     # run tests
```

## Project Structure

```
├── src/datatrove/
│   ├── data.py                   # Document, Media dataclasses
│   ├── io.py                     # DataFolder, fsspec I/O helpers
│   ├── pipeline/
│   │   ├── base.py               # PipelineStep base class
│   │   ├── readers/              # JsonlReader, ParquetReader, CsvReader, WarcReader, HuggingFaceDatasetReader, IpcReader
│   │   ├── writers/              # JsonlWriter, ParquetWriter, HuggingFaceDatasetWriter
│   │   ├── extractors/           # Trafilatura, ReadabilityInscriptis
│   │   ├── filters/              # Gopher, FineWeb, C4, Language, URL, Regex, Sampler, FastText, UnigramLogProb, Lambda
│   │   ├── formatters/           # FTFYFormatter, PIIFormatter, SymbolLinesFormatter
│   │   ├── dedup/                # Exact, MinHash, Sentence, BloomFilter, ExactSubstrings
│   │   ├── stats/                # Doc, Word, Line, Paragraph, Sentence, Lang, Token, Perplexity, Contamination stats + StatsMerger
│   │   ├── tokens/               # DocumentTokenizer, MegatronDocumentTokenizer, TokensCounter, Merger, ContextShuffler
│   │   ├── decont/               # NGramsDecontIndexer, NGramsDecontFilter
│   │   ├── inference/            # InferenceRunner, servers (VLLM, SGLang, Endpoint, Custom, Dummy), progress monitor, dataset cards
│   │   └── media/                # Binary/WARC/Zstd readers, writers, MIME filters, HTTP fetch
│   ├── executor/
│   │   ├── base.py               # PipelineExecutor base
│   │   ├── local.py              # LocalPipelineExecutor
│   │   ├── slurm.py              # SlurmPipelineExecutor
│   │   └── ray.py                # RayPipelineExecutor
│   ├── tools/                    # CLI: check_dataset, merge_stats, launch_pickled_pipeline, failed_logs, inspect_data, jobs_status, track_jobs
│   └── utils/                    # Hashing, tokenization, text processing, LID, perplexity, logging, stats, batching, jobs
├── tests/
│   ├── pipeline/                 # Tests for filters, readers, writers, dedup, extractors, stats, tokens, inference
│   └── executor/                 # Tests for local, ray executors
├── examples/                     # FineWeb, CommonCrawl, C4 tokenization, dedup, stats, inference
│   └── inference/                # Synthetic data generation, benchmarking, progress monitoring
├── pyproject.toml
├── Makefile
└── README.md
```

## Code Style

- **Formatter/linter:** Ruff (line length 119)
- **Naming:** `snake_case` functions/variables, `PascalCase` classes, `UPPER_SNAKE_CASE` constants
- **Imports:** stdlib → third-party → local, absolute imports (`from datatrove.pipeline.filters import ...`). Two blank lines after imports (isort config).
- **Type hints:** Required on all function signatures
- **Docstrings:** Google style for public functions and classes
- **Logging:** Uses `loguru` — import via `from loguru import logger`
- **Dependencies:** Optional deps checked at runtime via `_requires_dependencies` on `PipelineStep`

## Git Workflow

- Atomic, focused commits
- Conventional commit messages: `<type>(<scope>): <summary>` (types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`)
- Rebase branches, not merge
- Always add tests for new functionality
- Run `make quality` before pushing

## Development Guidelines

**Adding a new pipeline block:**
1. Inherit from the appropriate base class (see `pipeline/base.py` and subclass bases in each subdirectory)
2. Implement the required method (`run`, `filter`, `extract`, `format`, `extract_stats`)
3. Add optional dependency checks via `_requires_dependencies`
4. Use `self.stat_update()` and `self.track_time()` for metrics
5. Add tests in `tests/pipeline/`
6. Export in the relevant `__init__.py`

**Key base classes** — always inherit from the appropriate one:
- `PipelineStep` → general blocks (`pipeline/base.py`)
- `BaseFilter` → filters returning `bool` (`pipeline/filters/base_filter.py`)
- `BaseReader` / `BaseDiskReader` → data readers (`pipeline/readers/base.py`)
- `DiskWriter` → data writers (`pipeline/writers/disk_base.py`)
- `BaseExtractor` → HTML extraction (`pipeline/extractors/base.py`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huggingface/datatrove](https://github.com/huggingface/datatrove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
