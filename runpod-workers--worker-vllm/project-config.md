---
trigger: always_on
description: **worker-vllm** is a RunPod serverless worker that provides OpenAI-compatible endpoints for Large Language Model (LLM) inference, powered by the vLLM engine. It enables blazing-fast LLM deployment on RunPod's serverless infrastructure with minimal configuration.
---

# Worker vLLM - Development Conventions & Architecture Guide

## Project Overview

**worker-vllm** is a RunPod serverless worker that provides OpenAI-compatible endpoints for Large Language Model (LLM) inference, powered by the vLLM engine. It enables blazing-fast LLM deployment on RunPod's serverless infrastructure with minimal configuration.

### Core Purpose

- **Primary Function**: Deploy any Hugging Face LLM as an OpenAI-compatible API endpoint
- **Platform**: RunPod Serverless infrastructure
- **Engine**: vLLM (high-performance LLM inference engine)
- **Compatibility**: Drop-in replacement for OpenAI API (Chat Completions, Models)

## High-Level Architecture

### 1. **Entry Point & Request Flow**

```
RunPod Request → handler.py → JobInput → Engine Selection → vLLM Generation → Streaming Response
```

**Key Components:**

- `src/handler.py`: Main entry point using RunPod serverless framework
- `src/utils.py`: Request parsing and utility classes (`JobInput`, `BatchSize`)
- Two engine modes: OpenAI-compatible vs. standard vLLM

### 2. **Engine Architecture**

#### Core Classes:

- **`vLLMEngine`**: Base engine handling vLLM initialization and generation
- **`OpenAIvLLMEngine`**: Wrapper providing OpenAI API compatibility
- **Engine Selection**: Automatic routing based on `job_input.openai_route`

#### Key Design Patterns:

- **Dual API Support**: Same codebase serves both OpenAI-compatible and native vLLM APIs
- **Streaming by Default**: Token-level streaming with configurable batching
- **Dynamic Batching**: Adaptive batch sizes that grow from min → max for efficiency

### 3. **Configuration System**

#### Environment-Based Configuration:

- **Single Source of Truth**: All configuration via environment variables
- **Hierarchical Loading**: `DEFAULT_ARGS` → `os.environ` → `local_model_args.json` (for baked models)
- **vLLM Argument Mapping**: Automatic translation of env vars to vLLM `AsyncEngineArgs`

#### Key Configuration Files:

- `src/engine_args.py`: Centralized configuration management
- `src/constants.py`: Default values for core settings
- `.runpod/hub.json`: Hub UI configuration (CRITICAL: always update when changing defaults)
- `worker-config.json`: UI form generation for RunPod console (if exists)

## Core Development Concepts

### 1. **Deployment Models**

#### Option 1: Pre-built Images (Recommended)

- **Image**: `runpod/worker-v1-vllm:<version>` (see [GitHub Releases](https://github.com/runpod-workers/worker-vllm/releases))
- **Configuration**: Entirely via environment variables
- **Model Loading**: Downloads model at runtime from Hugging Face
- **Use Case**: Quick deployment, model experimentation

#### Option 2: Baked Model Images

- **Build Process**: Model downloaded during Docker build
- **Storage**: Model embedded in container image
- **Configuration**: Stored in `/local_model_args.json`
- **Use Case**: Production deployments, faster cold starts

### 2. **Request Processing Patterns**

#### Input Handling:

```python
class JobInput:
    - llm_input: str | List[Dict] (prompt or messages)
    - sampling_params: SamplingParams (generation settings)
    - stream: bool (streaming vs batch response)
    - openai_route: bool (API compatibility mode)
    - batch_size configs: Dynamic batching parameters
```

#### Response Streaming:

- **Batched Streaming**: Tokens grouped into configurable batch sizes
- **Dynamic Growth**: `min_batch_size * growth_factor^n` up to `max_batch_size`
- **Usage Tracking**: Input/output token counting for billing

### 3. **Model & Tokenizer Management**

#### Tokenizer Handling:

- **Wrapper Pattern**: `TokenizerWrapper` for consistent chat template application
- **Special Cases**: Mistral models use vLLM's native tokenizer
- **Chat Templates**: Automatic application for message-based inputs

#### Model Loading:

- **Multi-GPU Support**: Automatic tensor parallelism detection
- **Quantization**: Support for AWQ, GPTQ, BitsAndBytes
- **Caching**: Hugging Face cache management

## Development Patterns & Best Practices

### 1. **Code Organization**

#### File Structure:

```
src/
├── handler.py          # RunPod entry point
├── engine.py          # Core vLLM engines
├── engine_args.py     # Configuration management
├── utils.py           # Request parsing & utilities
├── tokenizer.py       # Tokenizer wrapper
├── constants.py       # Default constants
└── download_model.py  # Model downloading logic
```

#### Separation of Concerns:

- **Engine Logic**: Isolated in `engine.py` classes
- **Configuration**: Centralized in `engine_args.py`
- **Request Handling**: Abstracted via `JobInput` class
- **Platform Integration**: Contained in `handler.py`

### 2. **Error Handling & Logging**

#### Logging Strategy:

- **Structured Logging**: Consistent format across components
- **Performance Tracking**: Timer decorators for critical operations
- **Error Context**: Detailed error messages with configuration context


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runpod-workers/worker-vllm](https://github.com/runpod-workers/worker-vllm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
