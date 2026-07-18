---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MostlyLucid-NMT is a production-ready FastAPI service providing an EasyNMT-compatible HTTP API for machine translation. It supports multiple model families:
- **Opus-MT** (Helsinki-NLP): 1200+ translation pairs for 150+ languages
- **mBART50** (Facebook): All-to-all translation for 50 languages
- **M2M100** (Facebook): All-to-all translation for 100 languages

The codebase has been completely refactored into a modular, tested, and maintainable structure.

## Code Architecture

### Directory Structure

```
mostlylucid-nmt/
├── src/                        # Main application package
│   ├── __init__.py
│   ├── app.py                  # FastAPI application with lifespan management
│   ├── config.py               # Centralized configuration (40+ env vars)
│   ├── models.py               # Pydantic request/response models
│   ├── exceptions.py           # Custom exception classes
│   ├── core/                   # Core infrastructure
│   │   ├── logging.py          # Structured logging setup
│   │   ├── cache.py            # LRU pipeline cache with GPU memory management
│   │   ├── chunk_cache.py      # LFU chunk translation cache
│   │   ├── device.py           # Device selection and management
│   │   ├── ct2_loader.py       # CTranslate2 model loading and conversion
│   │   ├── ct2_wrapper.py      # CTranslate2 pipeline-compatible wrapper
│   │   └── pi_optimizations.py # Raspberry Pi optimizations
│   ├── services/               # Business logic layer
│   │   ├── model_manager.py    # Model loading and caching (supports opus-mt, mbart50, m2m100)
│   │   ├── model_discovery.py  # Discovers available models from Hugging Face
│   │   ├── translation_service.py  # Translation pipeline
│   │   ├── language_detection.py   # Language detection
│   │   └── queue_manager.py    # Request queueing and backpressure
│   ├── utils/                  # Utility modules
│   │   ├── text_processing.py  # Sanitization, splitting, chunking
│   │   ├── symbol_masking.py   # Symbol masking/unmasking
│   │   └── markdown_sanitizer.py # Markdown depth/bracket sanitization
│   └── api/                    # API layer
│       └── routes/             # Route modules
│           ├── translation.py  # Translation endpoints
│           ├── language.py     # Language detection endpoints
│           ├── discovery.py    # Model discovery endpoints
│           └── observability.py # Health, cache, metrics endpoints
├── tests/                      # Comprehensive test suite
│   ├── conftest.py             # Pytest fixtures and configuration
│   ├── test_text_processing.py
│   ├── test_symbol_masking.py
│   ├── test_markdown_sanitizer.py
│   ├── test_cache.py
│   ├── test_chunk_cache.py     # LFU chunk cache tests
│   ├── test_config.py
│   └── test_api_integration.py
├── test_api_comprehensive.py   # HTTP API test suite (Python, comprehensive)
├── test_api_quick.sh           # Quick API test (Bash/Linux/Mac)
├── test_api_quick.bat          # Quick API test (Windows)
├── app.py                      # Entry point (imports src.app)
├── app_old.py                  # Original monolithic version (backup)
├── requirements.txt            # Dependencies with versions
├── pytest.ini                  # Pytest configuration
├── Dockerfile                  # CPU build
├── Dockerfile.gpu              # GPU build with CUDA 12.1
├── Dockerfile.min              # Minimal CPU build (no preloaded models)
├── Dockerfile.gpu.min          # Minimal GPU build (no preloaded models)
└── README.md                   # User documentation
```

### Dependency Injection Pattern

The application uses FastAPI's dependency injection for clean separation:
- `get_translation_service()` - Provides translation service with thread pool
- `get_frontend_executor()` - Provides ThreadPoolExecutor for lightweight tasks

### Lifespan Management

Application startup/shutdown is managed via FastAPI's `@asynccontextmanager`:
1. **Startup**: Initialize executors → create services → preload models → start maintenance task
2. **Shutdown**: Cancel tasks → shutdown executors → clear CUDA cache

## Translation Pipeline Architecture

### Request Flow (src/services/translation_service.py)

```
Input texts
  ↓
Noise detection (src/utils/text_processing.py:is_noise)
  ↓
Sentence splitting (split_sentences)
  ↓
Chunking (chunk_sentences)
  ↓
Symbol masking (src/utils/symbol_masking.py:mask_symbols)
  ↓
LFU chunk cache lookup (src/core/chunk_cache.py)
  ↓ (cache misses only)
Get model from cache (src/services/model_manager.py)
  ↓
Translate batches (EASYNMT_BATCH_SIZE)
  ↓
Store in chunk cache
  ↓
Unmask symbols (unmask_symbols)
  ↓
Post-process (remove_repeating_new_symbols)
  ↓
Markdown sanitization (src/utils/markdown_sanitizer.py:sanitize_translations)
  ↓
Return translations
```

### Queue Management (src/services/queue_manager.py)

- **TranslateSlot**: Context manager for semaphore-based slot acquisition
- **QueueManager**: Tracks waiting/inflight counts, estimates retry-after
- **Backpressure**: Returns 429 with `Retry-After` header when queue full

### Model Cache (src/core/cache.py)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scottgal/mostlyucid-nmt](https://github.com/scottgal/mostlyucid-nmt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
