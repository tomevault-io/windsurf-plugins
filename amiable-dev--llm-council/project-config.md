---
trigger: always_on
description: This file contains technical details, architectural decisions, and important implementation notes for future development sessions.
---

# CLAUDE.md - Technical Notes for LLM Council

This file contains technical details, architectural decisions, and important implementation notes for future development sessions.

## Project Overview

LLM Council is a 3-stage deliberation system where multiple LLMs collaboratively answer user questions. The key innovation is anonymized peer review in Stage 2, preventing models from playing favorites.

## Architecture

### Backend Structure (`backend/`)

**`tier_contract.py`** - ADR-022 Tier Contract
- `TierContract`: Frozen dataclass defining tier execution parameters
  - `tier`: Confidence level (quick|balanced|high|reasoning)
  - `deadline_ms`, `per_model_timeout_ms`: Timeout configuration
  - `token_budget`, `max_attempts`: Resource limits
  - `requires_peer_review`, `requires_verifier`: Stage flags
  - `allowed_models`, `aggregator_model`: Model configuration
  - `override_policy`: Escalation rules
- `create_tier_contract(tier, task_domain)`: Factory function to create contracts
  - **ADR-026**: Now accepts optional `task_domain` parameter for dynamic selection
  - Uses `select_tier_models()` when `LLM_COUNCIL_MODEL_INTELLIGENCE=true`
- `TIER_AGGREGATORS`: Speed-matched aggregator models per tier
- `DEFAULT_TIER_CONTRACTS`: Pre-built contracts for all tiers

**`metadata/`** - ADR-026 Model Intelligence Layer
- Provides model metadata abstraction for the Model Intelligence Layer
- Supports offline operation (StaticRegistryProvider) and dynamic metadata (DynamicMetadataProvider)

- **`types.py`**: Core types
  - `ModelInfo`: Frozen dataclass with id, context_window, pricing, modalities, quality_tier
  - `QualityTier`: Enum (FRONTIER, STANDARD, ECONOMY, LOCAL)
  - `Modality`: Enum (TEXT, VISION, AUDIO, etc.)
  - `ModelStatus`: Enum (AVAILABLE, DEPRECATED, PREVIEW, BETA) - ADR-028
- **`protocol.py`**: `MetadataProvider` Protocol (interface)
  - `get_model_info()`, `get_context_window()`, `get_pricing()`
  - `supports_reasoning()`, `list_available_models()`
- **`static_registry.py`**: `StaticRegistryProvider` (31 models from YAML)
  - Offline-safe provider using bundled registry + LiteLLM fallback
  - Registry at `src/llm_council/models/registry.yaml`
- **`dynamic_provider.py`**: `DynamicMetadataProvider` (ADR-026 Phase 1)
  - Fetches metadata from OpenRouter API with TTL caching
  - Falls back to StaticRegistryProvider when cache empty or offline
- **`cache.py`**: TTL caching infrastructure
  - `TTLCache`: Thread-safe cache with LRU eviction
  - `ModelIntelligenceCache`: Composite (registry 1hr, availability 5min TTL)
- **`openrouter_client.py`**: Async OpenRouter API client
  - `fetch_models()`: Fetches model metadata from OpenRouter
  - `transform_api_model()`: Transforms API response to ModelInfo
- **`selection.py`**: Tier selection algorithm (ADR-026 Phase 1)
  - `TIER_WEIGHTS`: Per-tier weight matrices (quick→latency, reasoning→quality)
  - `QUALITY_TIER_SCORES`: QualityTier → float mapping (ADR-030 benchmark-justified):
    - FRONTIER: 0.95 (MMLU 87-90%)
    - STANDARD: 0.85 (MMLU 80-86%, +0.10 from 0.75)
    - ECONOMY: 0.70 (MMLU 70-79%, +0.15 from 0.55)
    - LOCAL: 0.50 (MMLU 55-80%, +0.10 from 0.40)
  - `COST_REFERENCE_HIGH`: Reference price for cost normalization (0.015 per 1K tokens)
  - `ModelCandidate`: Dataclass for model scoring
  - `apply_anti_herding_penalty()`: Penalizes models with >30% traffic
  - `select_with_diversity()`: Enforces provider diversity (min 2 providers)
  - `select_tier_models()`: Main entry point for dynamic selection
  - **Metadata Integration Functions** (ADR-026 Phase 1 "Hollow" Fix):
    - `_get_provider_safe()`: Get provider without crashing (returns None on failure)
    - `_get_quality_score_from_metadata()`: Real QualityTier lookup (not regex)
    - `_get_cost_score_from_metadata()`: Real pricing data (not regex)
    - `_meets_context_requirement()`: Real context window filtering (not always True)
  - **Circuit Breaker Integration** (ADR-030):
    - `_is_circuit_breaker_enabled()`: Check if circuit breaker is enabled (env > config > default)
    - `_is_circuit_breaker_open(model_id)`: Check if model's circuit is open
    - `select_tier_models()` filters out models with open circuit breakers
- **`scoring.py`**: Cost scoring algorithms (ADR-030)
  - `MIN_PRICE`: Floor to avoid log(0) ($0.0001)
  - `CostScaleAlgorithm`: Literal["linear", "log_ratio", "exponential"]
  - `get_cost_score_log_ratio()`: Log-ratio scoring for exponential price differences
  - `get_cost_score_exponential()`: Exponential decay scoring
  - `get_cost_score_with_config()`: Uses algorithm from config/env var
  - `QUALITY_TIER_BENCHMARK_SOURCES`: Citation URLs for quality tier justification
- **`offline.py`**: Offline mode detection
  - `is_offline_mode()`: Checks `LLM_COUNCIL_OFFLINE` env var
- **`registry.py`**: ADR-028 Model Registry (Background Registry Pattern)
  - `ModelRegistry`: Thread-safe singleton cache with asyncio.Lock
  - `RegistryEntry`: Cached model info with timestamp
  - `get_registry()`: Singleton factory
  - `refresh_registry()`: Background refresh with stale-while-revalidate
  - `get_candidates()`: Read-only access for request-time filtering
  - `get_health_status()`: Health check for observability

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amiable-dev/llm-council](https://github.com/amiable-dev/llm-council) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
