---
trigger: always_on
description: Scalene is a high-performance CPU, GPU, and memory profiler for Python with AI-powered optimization proposals. It runs significantly faster than other Python profilers while providing detailed performance information. See the paper `docs/osdi23-berger.pdf` for technical details on Scalene's design.
---

# Scalene Development Guide

## Project Overview

Scalene is a high-performance CPU, GPU, and memory profiler for Python with AI-powered optimization proposals. It runs significantly faster than other Python profilers while providing detailed performance information. See the paper `docs/osdi23-berger.pdf` for technical details on Scalene's design.

**Key features:**
- CPU, GPU (NVIDIA/Apple), and memory profiling
- AI-powered optimization suggestions (OpenAI, Anthropic, Azure, Amazon Bedrock, Gemini, Ollama)
- Web-based GUI and CLI interfaces
- Jupyter notebook support via magic commands (`%scrun`, `%%scalene`)
- Line-by-line profiling with low overhead
- Separates Python time from native/C time

**Platform support:** Linux, macOS, WSL 2 (full support); Windows (partial support)

## Build & Test Commands

```bash
# Install in development mode
pip install -e .

# Run all tests
python3 -m pytest tests/

# Run tests for a specific Python version
python3.X -m pytest tests/

# Run linters
mypy scalene
ruff check scalene

# Run a single test file
python3 -m pytest tests/test_coverup_83.py -v
```

## Project Structure

### Core Profiler Components (`scalene/`)

- **`scalene_profiler.py`** - Main profiler class (`Scalene`). Entry point for profiling. Uses signal-based sampling for CPU profiling. Coordinates all profiling subsystems.
- **`scalene_statistics.py`** - `ScaleneStatistics` class. Collects and aggregates profiling data. Key types: `ProfilingSample`, `MemcpyProfilingSample`. Uses `RunningStats` for statistical aggregation.
- **`scalene_output.py`** - Profile output formatting for CLI/HTML
- **`scalene_json.py`** - `ScaleneJSON` class for JSON output format
- **`scalene_analysis.py`** - Profile analysis logic

### Entry Points

- **`__main__.py`** - Entry point for `python -m scalene`
- **`profile.py`** - Entry point for `--on`/`--off` control of background profiling

### Configuration & Arguments

- **`scalene_config.py`** - Version info (`scalene_version`, `scalene_date`) and constants:
  - `SCALENE_PORT = 11235` - Default port for web UI
  - `NEWLINE_TRIGGER_LENGTH` - Must match `src/include/sampleheap.hpp`
- **`scalene_arguments.py`** - `ScaleneArguments` class (extends `argparse.Namespace`) with all profiler options and their defaults defined in `ScaleneArgumentsDict`
- **`scalene_parseargs.py`** - `ScaleneParseArgs.parse_args()` builds the argument parser. `RichArgParser` provides colored help output (uses Rich on Python < 3.14, native argparse colors on 3.14+)

### Signal Handling

- **`scalene_signals.py`** - Signal definitions for CPU sampling
- **`scalene_signal_manager.py`** - Manages signal handlers
- **`scalene_sigqueue.py`** - Signal queue management
- **`scalene_client_timer.py`** - Timer for periodic profiling

### GPU Support

- **`scalene_nvidia_gpu.py`** - NVIDIA GPU profiling via `pynvml`
- **`scalene_apple_gpu.py`** - Apple GPU profiling (Metal)
- **`scalene_accelerator.py`** - Generic accelerator interface
- **`scalene_neuron.py`** - AWS Neuron support

### Memory Profiling

- **`scalene_memory_profiler.py`** - Memory profiling logic
- **`scalene_leak_analysis.py`** - Memory leak detection (experimental, `--memory-leak-detector`)
- **`scalene_mapfile.py`** - `ScaleneMapFile` for memory-mapped communication with native extension
- **`scalene_preload.py`** - Sets up `LD_PRELOAD`/`DYLD_INSERT_LIBRARIES` for native memory tracking

### Jupyter Integration

- **`scalene_magics.py`** - Jupyter magic commands (`%scrun` for line mode, `%%scalene` for cell mode)
- **`scalene_jupyter.py`** - Jupyter notebook support utilities

### Replacement Modules (`replacement_*.py`)

These modules monkey-patch standard library functions to capture profiling data during blocking operations:
- **`replacement_fork.py`** - Tracks `os.fork()`
- **`replacement_exit.py`** - Tracks `sys.exit()`
- **`replacement_lock.py`**, **`replacement_mp_lock.py`**, **`replacement_sem_lock.py`** - Lock acquisition timing
- **`replacement_thread_join.py`**, **`replacement_pjoin.py`** - Thread/process join timing
- **`replacement_signal_fns.py`** - Signal function replacements
- **`replacement_poll_selector.py`** - I/O polling timing
- **`replacement_get_context.py`** - Multiprocessing context

### Utilities

- **`runningstats.py`** - `RunningStats` class for online statistical calculations (mean, variance)
- **`scalene_funcutils.py`** - Function utilities
- **`scalene_utility.py`** - General utilities
- **`sparkline.py`** - Sparkline generation for memory visualization
- **`syntaxline.py`** - Syntax-highlighted source code lines
- **`adaptive.py`** - Adaptive sampling logic
- **`time_info.py`** - Time measurement utilities
- **`sorted_reservoir.py`** - Reservoir sampling for bounded-size sample collection

### GUI (`scalene/scalene-gui/`)

Web-based GUI built with TypeScript, bundled with esbuild.

**Core Files:**
- **`index.html.template`** - Jinja2 template for main GUI page (rendered by `scalene_utility.py`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plasma-umass/scalene](https://github.com/plasma-umass/scalene) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
