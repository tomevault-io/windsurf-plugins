---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Project: CCProxy – OpenAI-compatible proxy for Anthropic Messages API

Common commands
- Install deps: uv pip install -r requirements.txt (includes asyncer for async operations and aiofiles for async file I/O)
- Run dev (uvicorn): python main.py
- Run via script (env checks): ./run-ccproxy.sh
- Docker build/run (compose): ./docker-compose-run.sh up -d
- Docker logs: ./docker-compose-run.sh logs -f
- Lint (ruff check): ./start-lint.sh --check
- Lint fix + format: ./start-lint.sh --all or ./start-lint.sh --fix
- Typecheck: mypy . (strict mode enabled)
- Tests (all): ./run-tests.sh or uv run pytest -q
- Tests with coverage: ./run-tests.sh --coverage
- Single test file: uv run pytest -q test_optimized_client.py
- Single test by node: uv run pytest -q test_optimized_client.py::test_name

Environment configuration
Required (via .env or environment)
- OPENAI_API_KEY or OPENROUTER_API_KEY
- BIG_MODEL_NAME
- SMALL_MODEL_NAME
Optional
- OPENAI_BASE_URL (default https://api.openai.com/v1)
- HOST (default 127.0.0.1)
- PORT (default 11434)
- LOG_LEVEL (default INFO)
- LOG_FILE_PATH (default log.jsonl)
- ERROR_LOG_FILE_PATH (default error.jsonl)
- WEB_CONCURRENCY (for multi-worker Uvicorn deployments)
Thread Pool Configuration (all optional)
- THREAD_POOL_MAX_WORKERS (default None - auto-calculates based on CPU cores, max 40)
- THREAD_POOL_HIGH_CPU_THRESHOLD (default None - auto-calculates based on CPU count: 60% + 2.5% per core, max 90%)
- THREAD_POOL_AUTO_SCALE (default False - enable dynamic scaling based on CPU contention)
Cache Warmup (all optional)
- CACHE_WARMUP_ENABLED (default False)
- CACHE_WARMUP_FILE_PATH (default cache_warmup.json)
- CACHE_WARMUP_MAX_ITEMS (default 100)
- CACHE_WARMUP_ON_STARTUP (default True)
- CACHE_WARMUP_PRELOAD_COMMON (default True)
- CACHE_WARMUP_AUTO_SAVE_POPULAR (default True)
- CACHE_WARMUP_POPULARITY_THRESHOLD (default 3)
- CACHE_WARMUP_SAVE_INTERVAL_SECONDS (default 3600)
Cython Optimization (all optional)
- CCPROXY_ENABLE_CYTHON (default True - enable Cython-compiled modules for 15-35% performance improvement)
- CCPROXY_BUILD_CYTHON (default True - build Cython extensions during installation)
Scripts create .env.example and validate env where helpful.

Run options
- Local dev: python main.py (FastAPI with uvicorn; auto-reload per Settings.reload)
- Production: ./run-ccproxy.sh (Uvicorn with multi-worker support; workers = CPU × 2 + 1)
- Docker: docker build -t ccproxy:latest -f Dockerfile .; docker-compose up -d
Health/metrics
- Health: GET / (root) returns {status: ok}
- Metrics: GET /v1/metrics; cache stats: GET /v1/cache/stats; clear caches: POST /v1/cache/clear

Big-picture architecture (Hexagonal/Clean Architecture)

## Domain Layer (ccproxy/domain/)
- Domain models and core business logic
- ccproxy/domain/models.py: Core domain entities and data structures
- ccproxy/domain/exceptions.py: Domain-specific exceptions and error handling

## Application Layer (ccproxy/application/)
- Use cases and application services
- ccproxy/application/converters.py: Message format conversion between Anthropic and OpenAI (exports async converters)
- ccproxy/application/converters_module/: Modular converter implementations with specialized processors
  - async_converter.py: AsyncMessageConverter and AsyncResponseConverter for parallel processing
  - Uses Asyncer library for improved async operations (asyncify for CPU-bound operations, anyio.create_task_group for parallel execution)
  - Optimized for high-throughput with parallel message and tool call processing
- ccproxy/application/tokenizer.py: Advanced async-aware token counting with TTL-based cache (300s expiry); uses anyio.create_task_group for parallel token encoding with asyncified tiktoken operations; includes OpenAI request counting via count_tokens_for_openai_request for precise integration with tiktoken encoders.
- ccproxy/application/model_selection.py: Model mapping (opus/sonnet→BIG, haiku→SMALL)
- ccproxy/application/request_validator.py: LRU cache (10,000 capacity) with cryptographic hashing
- ccproxy/application/response_cache.py: Response caching abstraction (delegates to cache implementations)
- ccproxy/application/cache/: Advanced caching with circuit breaker pattern, memory management, streaming de-duplication
  - warmup.py: CacheWarmupManager for preloading popular requests and common prompts; uses anyio.Path for async file operations and parallel warmup item loading
- ccproxy/application/error_tracker.py: Comprehensive error tracking and monitoring system with async JSON serialization and parallel redaction processing using asyncer
- ccproxy/application/thread_pool.py: Intelligent thread pool management for CPU-bound operations
  - Auto-detects multi-worker deployment via WEB_CONCURRENCY and adjusts accordingly
  - Prevents resource exhaustion: reduces threads per worker in multi-worker mode
  - Target total threads = CPU_count × 5 (distributed across workers)
  - Single worker: up to 40 threads; Multi-worker: 4-20 threads per worker
- ccproxy/application/type_utils.py: Type utilities and helper functions (uses Cython optimizations for type checking)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OCWorkforces) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
