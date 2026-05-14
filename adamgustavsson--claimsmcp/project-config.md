---
trigger: always_on
description: **ClaimsMCP** is a Model Context Protocol (MCP) server that implements the "Claimify" research methodology for extracting verifiable factual claims from text. This is a production-ready implementation using OpenAI's structured outputs API and follows the peer-reviewed approach from "Towards Effective Extraction and Evaluation of Factual Claims" by Metropolitansky & Larson (2025).
---

# AGENTS.md - Project Documentation for AI Agents

## Project Overview

**ClaimsMCP** is a Model Context Protocol (MCP) server that implements the "Claimify" research methodology for extracting verifiable factual claims from text. This is a production-ready implementation using OpenAI's structured outputs API and follows the peer-reviewed approach from "Towards Effective Extraction and Evaluation of Factual Claims" by Metropolitansky & Larson (2025).

**Key Insight**: This is NOT just another text extraction tool. It implements a sophisticated 4-stage academic pipeline that filters out opinions, resolves ambiguities, and produces atomic, self-contained claims.

## Architecture at a Glance

```
User Text → MCP Server → Pipeline → LLM Client → MCP Sampling (primary)
                            ↓                    ↓ (or fallback)
         [Splitting → Selection → Disambiguation → Decomposition]
                            ↓                    ↓
                    Structured Claims        OpenAI API
```

## Core Components

### 1. `claimify_server.py` - MCP Server Entry Point
- **Purpose**: Exposes claim extraction as an MCP tool via stdio
- **Protocol**: Uses Model Context Protocol for client communication
- **Tool**: `extract_claims(text_to_process, question)` - the main interface
- **Important**: Uses stdio (stdin/stdout), NOT HTTP - this is intentional for MCP
- **Async**: Built on asyncio, runs single-threaded event loop

**Key Implementation Details**:
- Returns JSON array of claims as string
- Error handling wraps all exceptions
- Validates API keys on startup
- Logs to stderr (stdout reserved for MCP protocol)

### 2. `pipeline.py` - Core Extraction Logic
- **Purpose**: Orchestrates the 4-stage Claimify methodology
- **Architecture**: Each stage is a separate function with structured I/O

**The 4 Stages**:

1. **Sentence Splitting** (`split_into_sentences`)
   - Uses NLTK's punkt tokenizer
   - Handles paragraphs and list items
   - Maintains sentence boundaries accurately

2. **Selection Stage** (`run_selection_stage`)
   - Filters for sentences with verifiable propositions
   - Removes opinions, speculation, generic statements
   - Uses surrounding context (p=5, f=5 sentences)
   - Returns: `('verifiable', sentence)` or `('unverifiable', None)`

3. **Disambiguation Stage** (`run_disambiguation_stage`)
   - Resolves pronouns, acronyms, partial names
   - Addresses referential and structural ambiguity
   - Only proceeds if ambiguity can be resolved with context
   - Returns: `('resolved', sentence)` or `('unresolvable', None)`

4. **Decomposition Stage** (`run_decomposition_stage`)
   - Breaks sentences into atomic claims
   - Adds clarifying context in [brackets]
   - Ensures each claim is self-contained
   - Returns: `List[str]` of claims

**Flow Control**:
- Each sentence processes through all stages sequentially
- Early exit if any stage filters out the sentence
- Final output is deduplicated list of all claims

**Context Windows**:
- Fixed at p=5, f=5 (preceding/following sentences)
- Based on paper's experimental findings
- Balance between context richness and token cost

### 3. `llm_client.py` - LLM Communication Layer
- **Purpose**: Handles all LLM communication via MCP sampling (primary) or OpenAI API (fallback)
- **Key Feature**: Tries MCP sampling first, falls back to OpenAI's `beta.chat.completions.parse` API
- **Type Safety**: All responses validated against Pydantic models (via JSON parsing for sampling, native for API)
- **Model Support**: MCP sampling works with any model; OpenAI API requires gpt-4o-2024-08-06, gpt-4o-mini, gpt-4o

**Critical Methods**:
- `make_structured_request(system_prompt, user_prompt, response_model, stage)` - Main entry point, tries sampling then API
- `_make_sampling_request()` - Uses MCP `session.create_message()` with retry logic for malformed JSON
- `_make_openai_request()` - Direct OpenAI API call with structured outputs
- `_extract_json_from_text()` - Parses JSON from text responses (handles markdown code blocks)
- Returns parsed Pydantic model or None on failure
- Handles refusals and validation errors explicitly

**Retry Logic**:
- MCP sampling responses are parsed as JSON and validated with Pydantic
- On validation failure, retries up to `SAMPLING_MAX_RETRIES` times with error hints
- Validation errors are included in retry prompts to guide LLM corrections

**Logging Strategy**:
- Controlled by `LOG_LLM_CALLS` env var
- Logs to stderr or file (configurable)
- Captures: method used (sampling/API), prompts, responses, token usage, duration
- Each call numbered for tracing through pipeline

### 4. `structured_models.py` - Pydantic Response Schemas
- **Purpose**: Define strict response formats for structured outputs
- **Type Safety**: Enforced by OpenAI API, not just validation

**Models**:

```python
SelectionResponse:
  - sentence: str
  - thought_process: str  # 4-step CoT

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdamGustavsson/ClaimsMCP](https://github.com/AdamGustavsson/ClaimsMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
