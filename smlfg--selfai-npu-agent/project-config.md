---
trigger: always_on
description: This is a sophisticated **AI-powered terminal chatbot with multi-backend inference support** designed for Windows on ARM with Snapdragon X Elite NPU acceleration. The project implements a three-phase intelligent pipeline (SelfAI) with fallback mechanisms, memory management, and agent-based task execution.
---

# AI NPU Agent Project - Architecture & Documentation

## Project Overview

This is a sophisticated **AI-powered terminal chatbot with multi-backend inference support** designed for Windows on ARM with Snapdragon X Elite NPU acceleration. The project implements a three-phase intelligent pipeline (SelfAI) with fallback mechanisms, memory management, and agent-based task execution.

**Key Purpose**: Enable efficient local AI inference with automatic fallback from NPU hardware acceleration to CPU execution, all managed through a configuration-driven system with optional planning and merge phases.

---

## Architecture Overview

### High-Level System Design

The system implements a **three-phase pipeline**:

```
┌─────────────────────────────────────────────────────────────────┐
│                        SelfAI Pipeline                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. PLANNING PHASE (Ollama-based)                               │
│     ├─ Accepts user goal/request                                │
│     ├─ Generates DPPM plan (Distributed Planning Problem Model) │
│     └─ Creates subtasks with dependencies & merge strategy      │
│                                                                  │
│  2. EXECUTION PHASE (Multi-backend LLM inference)               │
│     ├─ Executes subtasks sequentially/parallel (per plan)       │
│     ├─ Uses AgentManager to route to specialized agents         │
│     ├─ Falls back between backends: AnythingLLM → QNN → CPU    │
│     └─ Saves results and tracks status                          │
│                                                                  │
│  3. MERGE PHASE (Result synthesis)                              │
│     ├─ Collects all subtask outputs                             │
│     ├─ Synthesizes into coherent final answer                   │
│     └─ Falls back gracefully with internal summary              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Multi-Backend Inference Strategy

The system supports **three execution backends in priority order**:

1. **AnythingLLM (NPU)** - Primary: Hardware-accelerated inference via Snapdragon X NPU
   - Communicates via HTTP API to AnythingLLM server
   - Configured in `config.yaml` under `npu_provider`
   - Supports streaming output

2. **QNN (Qualcomm Neural Network)** - Secondary: Direct NPU model execution
   - Automatically discovered from `models/` directory
   - Uses QAI Hub models (e.g., Phi-3.5-Mini-Instruct)
   - Optimized for on-device inference

3. **CPU Fallback** - Tertiary: Local CPU inference via llama-cpp-python
   - Uses GGUF quantized models (e.g., Phi-3-mini-4k-instruct.Q4_K_M.gguf)
   - Pure CPU execution, no GPU/NPU required
   - Guarantees functionality even without specialized hardware

**Automatic Failover**: If AnythingLLM fails, system automatically tries QNN, then CPU.

---

## Directory Structure

```
AI_NPU_AGENT_Projekt/
├── CLAUDE.md                          # This file - architecture documentation
├── README.md                           # User-facing project overview
├── UI_GUIDE.md                        # Terminal UI features & customization
├── config.yaml.template               # Configuration template
├── config_extended.yaml               # Extended configuration example
├── .env.example                       # Environment variables template
├── requirements.txt                   # Main dependencies
├── requirements-core.txt              # Core CPU dependencies
├── requirements-npu.txt               # NPU-specific dependencies
│
├── config_loader.py                   # Configuration loading & validation
├── main.py                            # Entry point: Agent initialization
├── llm_chat.py                        # QNN-based chat interface
│
├── selfai/                            # Main SelfAI package
│   ├── __init__.py
│   ├── selfai.py                      # Main CLI loop with full pipeline
│   ├── core/
│   │   ├── agent.py                   # Basic agent with tool-calling loop
│   │   ├── agent_manager.py           # AgentManager: manages multiple agents
│   │   ├── model_interface.py         # Base interface for LLM models
│   │   ├── anythingllm_interface.py   # AnythingLLM HTTP client
│   │   ├── npu_llm_interface.py       # QNN/NPU model interface
│   │   ├── local_llm_interface.py     # CPU fallback (llama-cpp-python)
│   │   ├── planner_ollama_interface.py# Ollama planner client
│   │   ├── merge_ollama_interface.py  # Ollama merge provider
│   │   ├── execution_dispatcher.py    # Subtask execution orchestrator
│   │   ├── memory_system.py           # Conversation & plan storage
│   │   ├── context_filter.py          # Smart context relevance filtering
│   │   ├── planner_validator.py       # Plan schema validation
│   │   └── smolagents_runner.py       # Smolagents integration
│   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smlfg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
