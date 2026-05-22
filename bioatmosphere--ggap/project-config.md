---
trigger: always_on
description: Generates 4 plot types: forest_dynamics, soil_biogeochemistry, environmental_conditions, summary_dashboard.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GGap is a GPU-accelerated agent-based forest gap dynamics model. It combines the UVAFME (University of Virginia Forest Model Enhanced) forest simulation with the SAGESim agent-based modeling framework to create a scalable, GPU-enabled forest Gap model.

The project integrates three major components:
1. **SAGESim** - GPU-accelerated agent-based modeling framework using CuPy and MPI
2. **UVAFME** - Traditional forest gap model (Python translation from Fortran)
3. **GGap** - Integration layer combining UVAFME processes with SAGESim's agent framework

## Development Environment Setup

### Dependencies

Python 3.13+ is required. Install dependencies using uv:

```bash
uv sync
```

Key dependencies:
- `cupy` - GPU array operations (requires CUDA or ROCm)
- `mpi4py` - Parallel execution across multiple ranks
- `numpy` - Numerical operations
- `rasterio` - Geospatial data handling
- `earthengine-api` & `geemap` - NLCD forest data access

### GPU Requirements

- **NVIDIA GPU**: CUDA drivers required
- **AMD GPU**: ROCm 5.7.1+ required

Install CuPy and mpi4py according to your system's hardware before installing other dependencies.

## Running the Model

### Quick Demo (No GPU Required)

Show available species and model architecture:

```bash
uv run python main.py
```

Or with standard Python:
```bash
python main.py
```

This demo doesn't require GPU/CuPy - it just displays species data.

### GGap Single-Site Simulation (GPU Required)

The primary entry point runs a single-site simulation with CSV-initialized species and climate from UVAFME input files:

```bash
cd gap
python run_one_site.py --num_gaps 200 --pool_size 1000 --years 500
```

Command-line options:
- `--num_gaps`: Number of gaps per site (default: 200)
- `--pool_size`: Max tree slots per gap (default: 1000)
- `--years`: Simulation duration in years (default: 1000)
- `--report_interval`: Years between progress reports and CSV output (default: 10)
- `--site_id`: Site ID from UVAFME CSV files (default: 0)
- `--data_dir`: Directory containing UVAFME CSV files (default: input_data)
- `--prefix`: File prefix for UVAFME CSV files (default: UVAFME2012)
- `--output_dir`: Directory for CSV output files (default: output_data)
- `--no_tree_data`: Skip writing tree_data.csv (can be very large)

**Example runs:**
```bash
# Quick test (10 gaps, 50 years)
python run_one_site.py --num_gaps 10 --years 50

# Full simulation
python run_one_site.py --num_gaps 200 --pool_size 1000 --years 1000

# Different site
python run_one_site.py --site_id 1 --data_dir input_data --prefix UVAFME2012
```

### Plotting Output

After a simulation, generate plots from CSV output:

```bash
cd gap
python plot_outputs.py --output-dir ../output_data --format png
```

Options: `--plots-dir`, `--format` (png/pdf/svg), `--dpi`, `--style`, `--show/--no-show`

Generates 4 plot types: forest_dynamics, soil_biogeochemistry, environmental_conditions, summary_dashboard.

### SAGESim Examples (Reference)

SAGESim provides reference implementations in `SAGESim/examples/`:

**SIR Epidemic Model:**
```bash
cd SAGESim/examples/sir
mpirun -n 4 python run.py --num_agents 10000 --percent_init_connections 0.1 --num_nodes 1
```

**Forest Gap Model (SAGESim Reference):**
```bash
cd SAGESim/examples/forest_gap
mpirun -n 4 python run.py --num_trees 500 --forest_size 100 --years 100
```

### UVAFME Model (Original)

Run the standalone UVAFME forest model:

```bash
cd UVAFME
python main.py
```

### Testing

Test model initialization (no GPU):
```bash
python -c "from gap.gap_model import GAPModel; m = GAPModel(); s = m.initialize_site(); print(s['site_name'], len(s['species']), 'species')"
```

Run SAGESim tests:
```bash
cd SAGESim
pytest tests/
```

## Code Architecture

### SAGESim Framework (`SAGESim/sagesim/`)

SAGESim is a GPU-accelerated agent-based modeling framework with double-buffering for race condition prevention.

**Core Components:**
- `model.py` - Main Model class orchestrating simulation execution
- `breed.py` - Agent breed definitions with properties and step functions
- `agent.py` - AgentFactory for creating and managing agents
- `space.py` - Spatial structures (NetworkSpace, GridSpace)
- `utils.py` - GPU kernel utilities for agent data access

**Key Concepts:**

1. **Breeds**: Define agent types with properties and step functions
   - Register properties: `self.register_property(name, default_value)`
   - Register step functions: `self.register_step_func(func, filepath, priority)`

2. **Step Functions**: GPU kernels decorated with `@jit.rawkernel(device="cuda")`
   - Must follow strict CuPy JIT constraints (see CuPy Limitations below)
   - Executed in parallel across all agents on GPU
   - Use double-buffering pattern for race-free writes

3. **Model Execution Flow**:
   - `model.setup(use_gpu=True)` - Initialize breeds, analyze step functions, generate GPU kernels
   - `model.simulate(ticks, sync_workers_every_n_ticks)` - Run simulation for N ticks
   - MPI synchronization occurs every N ticks across workers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bioatmosphere/GGap](https://github.com/bioatmosphere/GGap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
