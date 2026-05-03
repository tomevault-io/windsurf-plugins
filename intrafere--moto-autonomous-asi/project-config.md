---
trigger: always_on
description: Enables OpenRouter integration with automatic LM Studio fallback, plus boost controls and research metrics in the workflow panel.
---

# API Key Controls & Workflow Management System

## Overview

Enables OpenRouter integration with automatic LM Studio fallback, plus boost controls and research metrics in the workflow panel.

**Key Features:**
- **Per-Role OpenRouter Selection**: Each role independently uses LM Studio or OpenRouter
- **Global OpenRouter API Key**: Single key for all per-role OpenRouter selections. Boost can reuse it when no explicit boost-only override key is provided.
- **LM Studio Fallback**: Optional fallback per role on credit exhaustion
- **Free Model Cooldown Handling**: SERIAL BOTTLENECK pause, free model looping, and auto-selector backup (see below)
- **Boost Mode**: Selective task acceleration via two modes, using either an explicit boost override key or the active global OpenRouter key:
  - **Boost Next X Calls**: Counter-based, next X API calls regardless of task ID
  - **Category Boost**: Role-based, boosts all calls for specific role categories (Aggregator and Compiler only; Autonomous agents inherit from their parent roles automatically)
- **System works without LM Studio**: Defaults to OpenRouter when LM Studio unavailable

---

## Architecture Components

### Boost and Parallel Execution

**Boost is a ROUTING decision, NOT a CONCURRENCY decision.**
- Boost affects which API endpoint is used, NOT whether submitters run in parallel or serial
- Aggregation submitters ALWAYS run in parallel regardless of boost status (unless single-model mode)
- Single-model mode: triggered when all submitters AND validator use the SAME configured model ID. Boost routing does NOT trigger single-model mode.

### Backend Core

#### OpenRouterClient (`backend/shared/openrouter_client.py`)
- Async HTTP client. Base URL: `https://openrouter.ai/api/v1`
- App Attribution Headers: `HTTP-Referer: https://intrafere.com/moto-autonomous-home-ai/`, `X-Title: MOTO Deep Research Harness`
- Credit exhaustion detection: HTTP 402 OR error messages containing "credit", "insufficient", "balance", "quota", "key limit", "limit exceeded"
- Raises `CreditExhaustionError` on exhaustion (no retries). Retries transient errors (max 3).
- Temperature=0.0 default. No stop sequences (removed — caused premature truncation with certain models).

#### APIClientManager (`backend/shared/api_client_manager.py`)
- Central router for all API calls: boost check → role's OpenRouter (with resettable fallback) → LM Studio
- Tracks fallback state per role: `_role_fallback_state: Dict[str, str]`
- `reset_openrouter_fallbacks()`: Resets all roles originally configured for OpenRouter back from LM Studio fallback. Called automatically on API key set, or manually via reset endpoint.
- Lazy initialization: OpenRouter client initializes from `rag_config.openrouter_api_key` when first needed

**CRITICAL REQUIREMENT - Role Configuration:**
- **EVERY role calling `api_client_manager.generate_completion()` MUST be configured via `api_client_manager.configure_role()`**
- This includes: aggregator submitters/validator, compiler submitters/validator/critique, autonomous agents, Tier 3 final answer agents

**Boost Mode Priority** (`should_use_boost(task_id)`):
1. Boost Next X: `boost_next_count > 0` → True
2. Category Boost: `_extract_role_prefix(task_id) in boosted_categories` → True

**Counter Decrement:** `boost_next_count` decrements ONLY on successful boost API calls. Failed/exhausted calls do NOT decrement.

**Resettable Fallback:** When a role hits credit exhaustion, it falls back to LM Studio for subsequent calls. User can reset all fallen-back roles via `POST /api/openrouter/reset-exhaustion` or by re-setting the API key (auto-resets). Each role has independent fallback state. If no fallback configured: raises RuntimeError.

**Categories from role_id:**
- `aggregator_submitter_*` → "Aggregator Submitters"
- `aggregator_validator` → "Aggregator Validator"
- `compiler_high_context` → "Compiler High-Context"
- `compiler_high_param` → "Compiler High-Param"
- `compiler_validator` → "Compiler Validator"
- `autonomous_*` → "Autonomous"

#### BoostManager (`backend/shared/boost_manager.py`)
- Singleton. Key methods: `set_boost_config`, `clear_boost`, `set_boost_next_count`, `toggle_category_boost`, `should_use_boost` (main check for coordinators), `consume_boost_count` (only after successful boost call)
- Boost can use an **explicit override** OpenRouter API key, or it falls back to the active global OpenRouter key. A temporary `OpenRouterClient` is created per boosted task and closed immediately after.
- **Autonomous agent task ID inheritance**: All autonomous orchestration agents use parent role task ID prefixes — Topic Selector/Completion Reviewer/Reference Selector/Paper Title Selector/Tier 3 agents use `agg_sub1_*`; Topic Validator/Redundancy Checker use `agg_val_*`. Boosting a parent role automatically covers all autonomous agents that run on that model.

#### BoostLogger (`backend/shared/boost_logger.py`)
- Singleton. Log file: `backend/data/boost_api_log.txt`
- Methods: `log_api_call`, `get_logs(limit)`, `clear_logs`, `get_stats`
- Boost logs are merged into the main API call log view; boost endpoints remain available for boost-only debugging.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Intrafere/MOTO-Autonomous-ASI](https://github.com/Intrafere/MOTO-Autonomous-ASI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
