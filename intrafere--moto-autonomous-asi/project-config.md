---
trigger: always_on
description: LM Studio and its pre-loaded models can be reached at "http://127.0.0.1:1234".
---

## LM Studio Server Information
LM Studio and its pre-loaded models can be reached at "http://127.0.0.1:1234".
**NOTE:** The system works without LM Studio. If LM Studio is unavailable, users can configure OpenRouter for all roles.

## Complete Project Directory Structure and File Descriptions
project-root/
├── .github/                            # GitHub community health files
│   ├── ISSUE_TEMPLATE/                 # Public issue forms + security contact routing
│   └── pull_request_template.md        # Default pull request template
├── backend/
│   ├── shared/                          # SHARED RESOURCES
│   │   ├── __init__.py                  # Package initialization
│   │   ├── config.py                    # RAGConfig, SystemConfig
│   │   ├── models.py                    # Pydantic models (includes ModelConfig, BoostConfig, WorkflowTask)
│   │   ├── lm_studio_client.py          # LM Studio HTTP API client
│   │   ├── openrouter_client.py         # OpenRouter HTTP API client (credit exhaustion detection)
│   │   ├── api_client_manager.py        # Unified API router (OpenRouter/LM Studio fallback + boost)
│   │   ├── boost_manager.py             # Singleton boost manager (tracks boost modes: next-count, always-prefer, category)
│   │   ├── boost_logger.py              # Boost API call logger (persists to boost_api_log.txt)
│   │   ├── workflow_predictor.py        # Predicts next 20 API calls (mode-specific algorithms)
│   │   ├── free_model_manager.py        # Free model rotation/cooldown singleton (looping + auto-selector backup)
│   │   ├── rag_lock.py                  # Global RAG operation lock (prevents Aggregator/Compiler collision)
│   │   ├── token_tracker.py             # Cumulative input/output token tracker singleton with per-model breakdown and research timer
│   │   ├── wolfram_alpha_client.py      # Wolfram Alpha API client
│   │   ├── utils.py                     # Common utilities
│   │   ├── json_parser.py               # JSON parsing with sanitization for LLM quirks
│   │   ├── critique_memory.py           # Paper critique persistence (saves up to 10 validator critiques per paper)
│   │   ├── critique_prompts.py          # Default critique prompt and builder function for validator critiques
│   │   ├── secret_store.py              # Secure API key persistence via OS keyring (OpenRouter, Wolfram Alpha)
│   │   └── path_safety.py               # Safe path resolution helpers (realpath/normpath containment checks)
│   ├── aggregator/                      # AGGREGATOR 
│   │   ├── __init__.py
│   │   ├── core/
│   │   │   ├── __init__.py
│   │   │   ├── rag_manager.py           # 4-stage RAG pipeline orchestrator
│   │   │   ├── coordinator.py           # Manages 1-10 submitters + 1 validator (default 3, configurable per-submitter)
│   │   │   ├── queue_manager.py         # Submission queue. Monitors queue size to trigger submitter pause when ≥10 submissions.
│   │   │   └── context_allocator.py     # Direct injection vs RAG routing (tries direct first, offloads to RAG only when doesn't fit). Includes allocate_cleanup_review_context() which NEVER skips due to size - uses RAG when database too large.
│   │   ├── ingestion/
│   │   │   ├── __init__.py
│   │   │   ├── chunker.py               # Multi-config chunking (256/512/768/1024)
│   │   │   ├── pipeline.py              # Document ingestion pipeline
│   │   │   ├── normalizer.py            # Text normalization
│   │   │   └── metadata_extractor.py    # Extract metadata from chunks
│   │   ├── validation/
│   │   │   ├── __init__.py
│   │   │   ├── contradiction_checker.py # Detect contradictions
│   │   │   └── json_validator.py        # Validate JSON responses
│   │   ├── memory/
│   │   │   ├── __init__.py
│   │   │   ├── shared_training.py       # Validator-distributed database (accepted submissions)
│   │   │   ├── local_training.py        # Per-submitter rejection logs (last 5)
│   │   │   └── event_log.py             # Persistent event log (acceptances, rejections, cleanup removals)
│   │   ├── agents/
│   │   │   ├── __init__.py
│   │   │   ├── submitter.py             # Submitter agent (parallel, cyclic chunk sizes)
│   │   │   └── validator.py             # Validator agent (sequential validation)
│   │   └── prompts/
│   │       ├── __init__.py
│   │       ├── submitter_prompts.py     # Submitter system prompts + JSON schemas
│   │       └── validator_prompts.py     # Validator system prompts + JSON schemas
│   │
│   ├── compiler/                        # COMPILER (Phase 2)
│   │   ├── __init__.py                  # Package initialization
│   │   ├── core/
│   │   │   ├── __init__.py              # Package initialization
│   │   │   ├── compiler_coordinator.py  # Orchestrates sequential Markov chain workflow
│   │   │   └── compiler_rag_manager.py  # Compiler-specific RAG wrapper (user-configurable context per role)
│   │   ├── agents/
│   │   │   ├── __init__.py              # Package initialization
│   │   │   ├── high_context_submitter.py # 3 modes: construction, outline, review
│   │   │   ├── high_param_submitter.py   # Rigor enhancement mode
│   │   │   └── critique_submitter.py    # Critique phase submitter (peer review)
│   │   ├── validation/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Intrafere/MOTO-Autonomous-ASI](https://github.com/Intrafere/MOTO-Autonomous-ASI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
