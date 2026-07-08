---
trigger: always_on
description: **Last Updated:** 2026-05-11
---

# AGENTS.md - Guide for AI Agents Working on OPAL

**Last Updated:** 2026-05-11
**Purpose:** Essential information for AI agents (like Claude, GPT, etc.) working on the OPAL simulator codebase

---

## Table of Contents
1. [Quick Start](#quick-start)
2. [Project Overview](#project-overview)
3. [Code Structure](#code-structure)
4. [Running the Simulator](#running-the-simulator)
5. [Configuration System](#configuration-system)
6. [Key Components](#key-components)
7. [Development Workflow](#development-workflow)
8. [Testing](#testing)
9. [Important Patterns](#important-patterns)
10. [Common Tasks](#common-tasks)
11. [Debugging Tips](#debugging-tips)

---

## Quick Start

### Environment Setup
```bash
# Clone repository
git clone git@github.com:IBM/opal-sim.git
cd opal-sim

# setup or run from uv directly 
uv venv --python 3.11
source .venv/bin/activate
uv pip install -e .

# (Optional) enable the repo's pre-commit hook (Black formatter)
git config core.hooksPath .githooks
```

### Run Basic Simulation
```bash
# From project root (main.py self-extends sys.path; PYTHONPATH not required)
python ./opal/main.py

# With custom config and graphs
python ./opal/main.py -c ./configs/defaults.json -g

# With debug logging
OPAL_LOG_LEVEL=DEBUG python ./opal/main.py
```

### Run Tests
```bash
# All tests
pytest

# Specific test with verbose output
OPAL_LOG_LEVEL=DEBUG pytest -s -v ./tests/test_configs.py

# With output shown
pytest -s -v
```

---

## Project Overview

**OPAL** (Open simulator Platform for distributed AI and LLM workflows) is a discrete-event simulator for LLM inference platforms written in Python using SimPy.

### Key Features
- **Discrete-event simulation** using SimPy framework
- **vLLM worker modeling** with batching and scheduling
- **Distributed KV cache** management with tiering
- **Storage backends** (DFS, fixed latency, custom)
- **Workload generation** (uniform, exponential, trace replay)
- **Router with autoscaling** support
- **GPU modeling** with utilization tracking

### Why Simulation?
- **Cost:** Avoid expensive GPU infrastructure for exploration
- **Speed:** Fast iteration on design decisions
- **Complexity:** Explore policy space without full implementation

---

## Code Structure

```
opal-sim/
├── opal/                          # Main package
│   ├── __init__.py
│   ├── main.py                    # Entry point
│   ├── opal.py                    # OpalSimulator class
│   ├── opal_base.py               # Base classes (NoDynamicAttributes)
│   ├── environment.py             # OpalSimulatorEnvironment (SimPy env wrapper)
│   ├── opal_config.py             # Configuration system (ConfigProxy)
│   ├── opal_logging.py            # Logging utilities
│   ├── opal_profile.py            # Performance profiling decorator
│   ├── opal_registery.py          # OpalRegistry (worker singleton registry)
│   ├── defaults.py                # Reserved for future default values
│   │
│   ├── vllm_worker.py             # ⭐ LLMWorkerVLLMScheduler (1700+ lines)
│   ├── router.py                  # Router with policies
│   ├── autoscaling.py             # Autoscaling logic
│   ├── workload_orchestrator.py   # WorkloadOrchestrator (multi-stage)
│   ├── kvc_manager.py             # KV cache: OpalStorageBackend, OpalStorageManager
│   ├── kvbm.py                    # KV block manager
│   ├── gpu_model.py               # GPUModel (roofline/synthetic inference)
│   ├── io_model.py                # I/O: CPUMemory, LocalNVMe, DistributedFS devices
│   ├── llm_model.py               # OpalModelConfig (HF or local config loading)
│   │
│   ├── request.py                 # LLMRequest, LLMRequestStats, IORequest
│   ├── datatypes.py               # STR_DTYPE_TO_BYTES (dtype → byte-size map)
│   ├── events.py                  # KVCEvent, SystemEvent, OpalInfraEvent
│   ├── stage_statistics.py        # StageStatistics collection
│   ├── plot.py                    # Plotting utilities
│   ├── util.py                    # safe_process(), check_and_create_directory(), etc.
│   │
│   ├── workloads/                 # Workload generators
│   │   ├── abstract_workload.py   # AbstractWorkload base class
│   │   ├── workload.py            # UniformReqRate, ExponentialReqRate, Trace
│   │   └── sc25_blog.py           # SC25Workload
│   │
│   └── regression-fitting/        # Model calibration (a, b parameters)
│       ├── offline_calc_a_b.py
│       ├── online_calibration.py
│       └── README.md
│
├── configs/                       # Configuration files
│   ├── defaults.json              # Default configuration (local model dir)
│   └── hf.json                    # Variant that pulls model config from Hugging Face
│
├── model-configs/                 # Local model configurations
│   └── granite-3.3-8b-instruct/
│       └── config.json
│
├── traces/                        # Trace files for replay workloads
│   └── hello.jsonl                # Example trace
│
├── tests/                         # Unit tests
│   └── test_configs.py            # Parametrized config loading + run tests
│
├── wiki/                          # Documentation (cloned from GitHub wiki)
│   ├── Configuration-Simulation.md
│   ├── vllm-worker.md
│   ├── vLLM-modeling.md
│   ├── KVCache-manager.md
│   ├── Router.md
│   ├── Worker.md
│   ├── Workload-generation.md
│   ├── Running.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IBM/opal-sim](https://github.com/IBM/opal-sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
