---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RapidFire AI is an experiment execution framework for LLM fine-tuning and post-training that enables hyperparallelized training, dynamic real-time experiment control (IC Ops), and automatic multi-GPU orchestration. The system uses chunk-based scheduling to allow concurrent training of multiple configurations even on a single GPU.

## Key Commands

### Development Setup

```bash
# Create and activate virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies from source
pip install -r requirements.txt

# Install Node.js 22.x and build frontend
cd rapidfireai/frontend
node ./yarn/releases/yarn-4.9.1.cjs install
node ./yarn/releases/yarn-4.9.1.cjs build
cd ../..

# Start all services in development mode
chmod +x ./rapidfireai/start_dev.sh
./rapidfireai/start_dev.sh start

# Stop services
./rapidfireai/start_dev.sh stop
```

### Running from Installed Package

```bash
# Initialize RapidFire (installs dependencies, copies tutorials)
rapidfireai init

# Start RapidFire servers (dispatcher, mlflow, frontend)
rapidfireai start

# Stop all servers
rapidfireai stop

# System diagnostics (GPU, CUDA, Python env)
rapidfireai doctor

# Check version
rapidfireai --version
```

### Testing

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_chunks.py

# Run with verbose output
pytest -v
```

### Code Quality

```bash
# Format code with ruff (line-length: 120)
ruff format .

# Run linter
ruff check .

# Fix auto-fixable issues
ruff check --fix .
```

### Building and Releasing

```bash
# Build PyPI package (requires frontend build first)
rm -rf dist/ *.egg-info/ .eggs/ && python -m build

# Bump version (creates commit and tag)
./bump_version.sh patch  # 0.10.1 → 0.10.2
./bump_version.sh minor  # 0.10.1 → 0.11.0
./bump_version.sh major  # 0.10.1 → 1.0.0

# Push version tag to trigger TestPyPI deployment
git push origin test0.10.2
```

### Port Management

```bash
# Kill services on specific ports if conflicts occur
lsof -t -i:8851 | xargs kill -9  # dispatcher
lsof -t -i:8852 | xargs kill -9  # mlflow
lsof -t -i:8853 | xargs kill -9  # frontend
```

## Architecture

RapidFire AI uses a microservices-inspired distributed architecture:

### Core Components

1. **Experiment** (`experiment.py`): Top-level API for users. Manages experiment lifecycle, creates database tables, sets up logging and signal handlers. Entry point for `run_fit()` and `get_results()`.

2. **Controller** (`backend/controller.py`): Orchestrates the entire training lifecycle. Runs in the user's process. Responsible for:
   - Creating models from parameter configurations
   - Initializing and managing Workers
   - Running the Scheduler to assign chunks to workers
   - Handling Interactive Control Operations (IC Ops)
   - Monitoring training progress

3. **Scheduler** (`backend/scheduler.py`): Pure scheduling logic that assigns runs to available workers for specific chunks. Uses round-robin and fairness algorithms to ensure optimal GPU utilization. Tracks which runs have completed which chunks.

4. **Worker** (`backend/worker.py`): Separate GPU processes that execute actual training. Each worker:
   - Polls database for assigned tasks
   - Loads model checkpoints from shared memory or disk
   - Trains on assigned data chunks
   - Saves checkpoints back to shared memory/disk
   - Reports progress to MLflow

5. **Dispatcher** (`dispatcher/dispatcher.py`): Flask-based REST API for UI communication. Provides endpoints for:
   - Viewing experiment status
   - Interactive Control Operations (stop, resume, clone, delete runs)
   - Real-time run metrics

6. **Database** (`db/rf_db.py`): SQLite-based persistence layer with async operations. Stores:
   - Experiment metadata
   - Run configurations and status
   - Task scheduling state
   - Checkpoint locations

7. **Frontend** (`frontend/`): React-based dashboard (MLflow fork) with IC Ops panel. Displays live experiment tracking and enables dynamic control.

### Data Flow

1. User creates `Experiment` and calls `run_fit()` with configs and datasets
2. Controller creates runs in database and spawns Worker processes
3. Controller runs Scheduler loop to assign (run_id, chunk_id) to available workers
4. Workers poll database, load models, train on chunks, save checkpoints
5. Workers report metrics to MLflow and update database task status
6. Scheduler continues until all runs complete all chunks (epochs)
7. User can invoke IC Ops through UI to stop/resume/clone runs mid-training

### Shared Memory System

RapidFire uses shared memory (`utils/shm_manager.py`) to avoid disk I/O bottlenecks:
- Model checkpoints stored in shared memory between chunks (configurable via `USE_SHARED_MEMORY`)
- Registry tracks which models are in memory
- Process locks prevent concurrent access issues
- Fallback to disk for larger models

### Interactive Control (IC Ops)

Unique feature enabling real-time experiment control:
- **Stop**: Pause a run, saves checkpoint
- **Resume**: Restart a stopped run from checkpoint
- **Clone**: Create new run from existing, optionally warm-start from parent's weights

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RapidFireAI/rapidfireai](https://github.com/RapidFireAI/rapidfireai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
