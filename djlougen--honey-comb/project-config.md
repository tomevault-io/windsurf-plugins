---
trigger: always_on
description: This document provides setup and usage instructions for AI agents working with the Honey-Comb project.
---

# Honey-Comb Agent Instructions

This document provides setup and usage instructions for AI agents working with the Honey-Comb project.

## Project Overview

Honey-Comb is a context compression system for AI agent workflows. It uses rule-based and ML-based classifiers to compress context (keep essential information, remove noise) before passing it to LLMs, reducing token usage and improving performance.

## Quick Start

### Prerequisites
- Python 3.12+
- pip package manager
- Git

### Installation

```bash
# Clone the repository
git clone https://github.com/DJLougen/honey-comb.git
cd honey-comb

# Install in development mode
pip install -e .

# Install development dependencies
pip install -e ".[dev]"

# Verify installation
python -c "import honeycomb; print(honeycomb.__version__)"
```

### Run Tests

```bash
# Run all tests
pytest tests/

# Run with coverage
pytest tests/ --cov=honeycomb --cov-report=term-missing

# Run specific test file
pytest tests/test_compressor.py
```

## Project Structure

```
honey-comb/
├── honeycomb/              # Main package
│   ├── __init__.py        # Package initialization, version
│   ├── classifier.py      # ML classifier (TF-IDF + VotingClassifier)
│   ├── cli_train.py       # Training CLI
│   ├── compressor.py      # Compression rules (test output, files, reasoning)
│   ├── features.py        # Feature extraction for ML model
│   ├── firewall.py        # Main orchestrator (hot loop + cool loop)
│   ├── io.py              # JSONL I/O utilities
│   ├── labels.py          # Compression label taxonomy (CORE, DISTILL, COMPACT, DROP, STALE, ESCALATE)
│   ├── observability.py   # Metrics, logging, health checks
│   ├── session.py         # Thread-safe session state management
│   ├── budget.py          # Token budget management
│   └── config.py          # Configuration management
├── scripts/               # Utility scripts
│   ├── benchmark.py                    # Performance benchmarks
│   ├── benchmark_statistical.py        # Statistical benchmarks
│   ├── demo_pollution.py              # Demo: context compression
│   ├── demo_production.py             # Demo: production features
│   ├── generate_synthetic.py          # Generate training data
│   └── generate_visuals.py            # Generate documentation charts
├── tests/                 # Test suite (129 tests)
├── models/                # Trained ML models
├── examples/              # Example data
├── docs/                  # Documentation
├── pyproject.toml         # Project configuration
└── README.md              # Main documentation
```

## Key Concepts

### Compression Labels

Honey-Comb uses a 6-label taxonomy for context compression:

- **CORE**: Keep verbatim (system prompts, user goal, critical context)
- **DISTILL**: Extract key information (test output → summary + failures)
- **COMPACT**: Keep structure only (file → "src/foo.py (200 lines): class Foo, def bar()")
- **DROP**: Remove entirely (stale context, duplicate information)
- **STALE**: Mark for removal (superseded by newer information)
- **ESCALATE**: Pass to LLM (ambiguous content requiring understanding)

### Hot Loop vs Cool Loop

**Hot Loop** (per-message, ~1ms):
- Classifies and compresses each message as it arrives
- Uses rule-based or ML-based classification
- Deterministic compression rules per content type

**Cool Loop** (periodic, ~10-50ms):
- Runs every N turns (default: 10)
- Detects stale/superseded entries
- Enforces token budget

### Thread Safety

Honey-Comb is thread-safe by default:
- All session state operations are protected by locks
- Can be disabled for performance: `thread_safe=False`
- Use thread-safe mode in production, disable for single-threaded batch processing

### Production Features

1. **Observability**: Structured logging, metrics (Prometheus format), health checks
2. **Configuration**: Environment variables, config files (YAML/JSON)
3. **Performance Tuning**: Configurable thread safety, metrics collection, compression levels
4. **Token Budget Management**: Automatic enforcement of context limits

## Development Workflow

### Making Changes

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make changes** to code in `honeycomb/` or `scripts/`

3. **Run tests** to verify changes:
   ```bash
   pytest tests/
   ```

4. **Check linting**:
   ```bash
   ruff check honeycomb/
   ruff format --check honeycomb/
   ```

5. **Commit changes**:
   ```bash
   git add .
   git commit -m "feat: description of changes"
   ```

### Adding New Compression Rules

To add compression logic for a new content type:

1. **Add detection logic** in `firewall.py::_detect_content_type()`:
   ```python
   if self._is_new_content_type(content):
       return "new_content_type"
   ```

2. **Add compression function** in `compressor.py`:
   ```python
   def compress_new_content_type(self, content: str) -> str:
       # Implement compression logic
       return compressed_content
   ```

3. **Wire it up** in `compressor.py::compress()`:
   ```python
   if content_type == "new_content_type":
       return self.compress_new_content_type(content)
   ```

4. **Add tests** in `tests/test_compressor.py`:
   ```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DJLougen/honey-comb](https://github.com/DJLougen/honey-comb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
