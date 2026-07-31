---
trigger: always_on
description: This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **SkyPilot Spot Simulator** - a cloud computing simulation framework that models spot instance execution strategies for optimizing cost and reliability when running tasks on preemptible instances across multiple cloud regions. The project is designed for research into multi-region scheduling strategies for long-running tasks with deadlines.

### Historical Context
- **Single-region strategies** (e.g., `rc_cr_threshold`, `rc_cr`, etc.) are the original strategies from the "Can't Be Late" paper, designed for optimizing within a single cloud region.
- **Multi-region strategies** are the newer extension that enables scheduling across multiple regions for better cost optimization.

## Quick Start Guide

### Running Simulations

**Single simulation:**
```bash
# Single-region
python main.py --strategy=rc_cr_threshold --env=trace --trace-file data/real/ping_based/random_start_time/us-west-2a_k80_1/0.json --task-duration-hours=48 --deadline-hours=52 --restart-overhead-hours=0.2

# Multi-region
python main.py --strategy=multi_region_rc_cr_threshold --env=multi_trace --trace-files data/real/ping_based/random_start_time/us-west-2a_k80_1/0.json data/real/ping_based/random_start_time/us-west-2b_k80_1/0.json --task-duration-hours=48 --deadline-hours=52 --restart-overhead-hours=0.2
```

**Strategy development and testing:**
```bash
# Quick test new strategy
python scripts_multi/quick_test_strategy.py my_strategy

# Compare strategies across multiple traces  
python scripts_multi/batch_strategy_comparison.py 10

# Research-grade systematic evaluation
python scripts_multi/benchmark_multi_region_modular.py
```

### Environment Setup

**Note**: The exact environment setup is uncertain and may vary between systems. Some dependencies might have been installed using `uv` or other package managers. When encountering environment issues, diagnose the specific problem and use the appropriate installation method.

Basic requirements typically include:
- Python 3.8+
- NumPy, Pandas, Matplotlib
- Other dependencies as needed (check imports for specifics)

### Don't forget to activate the virtual environment
If you encounter a `ModuleNotFoundError` or errors indicating a missing package, or a SkyPilot error `sky.exceptions.APIVersionMismatchError: Client and local API server version mismatch`, your first step should be to verify that the project's virtual environment is activated. This ensures you are using the project-specific dependencies.

At project root, run:

```fish
source .venv/bin/activate.fish
```

### Logging Configuration

Logging is controlled by the `LOG_LEVEL` environment variable:
```bash
# Set logging level (DEBUG, INFO, WARNING, ERROR)
export LOG_LEVEL=INFO
python main.py ...

# Default is DEBUG if not set
python main.py ...  # Uses DEBUG level
```

## Data Organization

### Primary Data Sources

1. **Original single/two-region data**: `data/real/ping_based/random_start_time/`
   - Contains real trace data organized by region and instance type
   - Format: `{region}_{instance_type}_{count}/0.json`
   - Example: `us-west-2a_k80_1/0.json`

2. **Extended multi-region data (newer)**: `data/converted_multi_region/`
   - Contains traces for more regions
   - Used for comprehensive multi-region analysis

### Trace File Format
```json
{
    "metadata": {
        "gap_seconds": 600,  // Time step between data points
        "start_time": "2022-09-08 07:41:46.331298"  // Random start time
    },
    "data": [
        0, 0, 1, 1, 0, 0, 0, ...  // 1 = preempted, 0 = available
    ]
}
```

### Creating Traces

See `data/README.md` for comprehensive data generation guide. Key points:

1. **Raw data collection**: `scripts/availability/availability_trace.py` (runs on remote servers)
2. **Parsing**: `sky_spot/parsers/parse_availability.py`  
3. **Random start generation**: `data/real/ping_based/random_start_time/parse.py`
4. **Synthetic traces**: Use generators like `two_exp`, `two_gamma`, `poisson`

Example for synthetic traces:
```bash
python ./sky_spot/traces/generate.py --trace-folder data/two_exp \
    --generator two_exp --gap-seconds 600 --length 4320 \
    --num-traces 1 --alive-scale 24.4920 --wait-scale 6.4126984
```

## Architecture: Multi-Region Strategy Design

### Core Design Principle

The multi-region system uses a **yield-based generator pattern** that models real cloud constraints:

```python
# Strategy yields actions to environment
result = yield TryLaunch(region=0, cluster_type=ClusterType.SPOT)

# Environment returns results
# TryLaunch → LaunchResult(success=bool, region=int, cluster_type=ClusterType)
# Terminate → None
```

### Why This Design?

1. **Information Isolation**: Strategies should only receive information through the proper interface, not by accessing internal environment state. This ensures:
   - Strategies can't bypass billing rules
   - The environment maintains control over resource availability checks
   - Strategies remain portable across different environment implementations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UCB-ADRS/ADRS](https://github.com/UCB-ADRS/ADRS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
