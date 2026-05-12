---
trigger: always_on
description: This project is no longer maintained. See [ARCHIVED.md](ARCHIVED.md) for details.
---

# Prompt Refiner - Project Context

---

## ⚠️ PROJECT ARCHIVED (April 2026)

This project is no longer maintained. See [ARCHIVED.md](ARCHIVED.md) for details.

**Why archived:**
- Token costs dropped 12x (GPT-4: $30/M → $2.50/M)
- Context windows expanded 20-200x (now 200K-1M tokens)
- Better alternatives emerged (caching: 90%+ savings, ML compression: 20x)
- Rule-based optimization ceiling reached (5-15% is the limit)

**Key learning:** Rule-based token optimization has fundamental limits. Caching, ML-based compression, and model routing are more effective in 2026.

This documentation remains for reference.

---

This document provides context for Claude Code and developers working on this project.

## Project Purpose

Prompt Refiner is a Python library for building production LLM applications. It solves two core problems:

1. **Token Optimization** - Clean dirty inputs (HTML, whitespace, PII) to reduce API costs by 10-20%
2. **Context Management** - Pack system prompts, RAG docs, and chat history with automatic refinement and priority-based ordering

Perfect for RAG applications, chatbots, and any production system that needs to optimize LLM inputs efficiently.

## Architecture

The library is organized into 6 core transformation modules plus measurement utilities:

**Core Modules (Transform prompts):**
- **Cleaner**: Operations for cleaning dirty data (HTML, whitespace, Unicode, JSON)
- **Compressor**: Operations for reducing prompt size (truncation, deduplication)
- **Scrubber**: Operations for security and privacy (PII redaction)
- **Tools**: Operations for optimizing LLM tool schemas and responses (SchemaCompressor, ResponseCompressor) (v0.1.6+)
- **Packer**: Context composition with specialized packers and automatic refinement (v0.1.3+)
- **Strategy**: Benchmark-tested preset strategies (MinimalStrategy, StandardStrategy, AggressiveStrategy) (v0.1.5+)
  - **MessagesPacker**: For chat completion APIs (OpenAI, Anthropic)
  - **TextPacker**: For text completion APIs (Llama Base, GPT-3)
  - **Semantic roles for RAG**: ROLE_SYSTEM, ROLE_QUERY, ROLE_CONTEXT, ROLE_USER, ROLE_ASSISTANT
  - **Smart priority defaults**: Role automatically infers priority (PRIORITY_SYSTEM, PRIORITY_QUERY, PRIORITY_HIGH, PRIORITY_LOW)
  - **Default refining strategies**: Automatic cleaning (MinimalStrategy for system/query, StandardStrategy for context/history) (v0.2.1+)
  - Priority-based ordering with insertion order preservation
  - Grouped MARKDOWN sections for base models
  - Token savings tracking for optimization impact measurement

**Measurement Utilities (Analyze, don't transform):**
- **Analyzer**: Operations for measuring optimization impact (token counting, cost savings)

Each core module contains specialized operations that can be composed into pipelines using the `Refiner` class. The `Packer` module provides higher-level functionality for managing complex context budgets with support for both plain text and structured message formats. The Analyzer module provides measurement tools to track token savings and demonstrate ROI, but does not transform prompts.

## Development Philosophy

- Keep it lightweight - minimal dependencies (zero by default, optional for advanced features)
- Focus on performance - cleaning should be fast
- Make it configurable - users should control cleaning behavior
- Start simple - add features incrementally
- Graceful degradation - advanced features degrade gracefully when optional dependencies unavailable

## Version History

### v0.2.2 (Current) - Code Cleanup & Documentation Polish

**Non-Breaking Changes:**
- **Removed unused `model` parameter**: Cleaned up packer constructors
  - Removed from `BasePacker.__init__()`, `MessagesPacker.__init__()`, `MessagesPacker.quick_pack()`, `TextPacker.__init__()`, `TextPacker.quick_pack()`
  - Model parameter was never used internally, only stored
  - Follows YAGNI principle (You Aren't Gonna Need It)
  - Note: `model` parameter still exists in `CountTokens` operation for precise token counting

- **Removed unused TYPE_CHECKING blocks**: Cleaned up empty type checking imports in packer files

- **Documentation updates**:
  - Updated all examples to remove `model` parameter
  - Fixed README.md quickstart example
  - Updated API reference documentation
  - Clarified that `model` parameter is only for `CountTokens` operation

**Benefits:**
- Simpler, cleaner API
- No functional changes - purely cleanup
- Better code maintainability
- Reduced confusion about unused parameters

### v0.2.1 - Packer Simplification & Default Strategies
**BREAKING CHANGES:**

**Packer Simplification**
- **Removed `max_tokens` parameter**: Packers now include all items without token budget constraints
  - LLM APIs handle final token limits
  - Simpler API - no more budget management complexity
  - All items are included and ordered by priority, then insertion order
- **Removed overhead calculations**: No more `_calculate_overhead()`, `PER_MESSAGE_OVERHEAD`, `PER_REQUEST_OVERHEAD`
- **Renamed internal method**: `_greedy_select()` → `_select_items()` (simpler logic)
- **File renames**: `messages_packer.py` → `messages.py`, `text_packer.py` → `text.py`

**Default Refining Strategies (NEW)**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JacobHuang91/prompt-refiner](https://github.com/JacobHuang91/prompt-refiner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
