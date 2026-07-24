---
trigger: always_on
description: This project provides ROCm-optimized ComfyUI nodes for AMD GPUs (specifically gfx1151 architecture).
---

# ROCm Ninodes - Cursor AI Rules

## Project Overview
This project provides ROCm-optimized ComfyUI nodes for AMD GPUs (specifically gfx1151 architecture).
Focus on performance, memory efficiency, and maintainability.

## Naming Conventions (CRITICAL)

### ROCm Branding
- **Always use "ROCm"** (capital R, capital O, capital C, lowercase m)
- ❌ Wrong: "RocM", "ROCM", "rocm", "Rocm"
- ✅ Correct: "ROCm"
- Examples:
  - Class names: `ROCmDiffusionLoader` (when part of compound name)
  - Display names: "ROCm Checkpoint Loader"
  - Documentation: "ROCm-optimized nodes"
  - Categories: "ROCm Ninodes/Loaders"

## Environment Management (CRITICAL)

### Using `uv` Package Manager
This project uses **`uv`** (not pip, not conda) for dependency management:
- **Lock file**: `uv.lock` (committed to git)
- **Configuration**: `pyproject.toml` (project metadata and dependencies)
- **Python version**: >=3.13 (defined in uv.lock)

### Key `uv` Commands
```bash
# Sync dependencies (install from lockfile)
uv sync

# Add a new dependency
uv add <package-name>

# Add a dev dependency
uv add --dev <package-name>

# Update dependencies
uv lock --upgrade

# Run a command in the virtual environment
uv run <command>

# Run tests
uv run pytest tests/

# Activate the virtual environment (if needed)
source .venv/bin/activate  # Linux/Mac
.venv\Scripts\activate     # Windows
```

### Why `uv`?
- **Fast**: Rust-based, 10-100x faster than pip
- **Reliable**: Deterministic dependency resolution via lock file
- **Modern**: Designed for contemporary Python workflows
- **Compatible**: Works with existing pip/PyPI infrastructure

### PyTorch Installation
PyTorch with ROCm is NOT in `pyproject.toml` because it requires platform-specific installation:
```bash
# Install PyTorch with ROCm (do this manually in ComfyUI environment)
# See: https://pytorch.org/get-started/locally/
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm6.4
```

### Important Notes
- **Don't use `pip install` directly** - use `uv add` to maintain lockfile
- **Don't edit `uv.lock` manually** - let `uv` manage it
- **Commit both `pyproject.toml` and `uv.lock`** to version control
- **Platform-specific builds** (like PyTorch) installed separately in ComfyUI's environment

## Code Organization (CRITICAL)

### File Size Limits
- **Maximum file size: 500 lines** (excluding comments/docstrings)
- If a file exceeds 400 lines, consider splitting it
- Each module should have a single, clear responsibility
- AI tools struggle with files >1000 lines - keep modules focused

### Package Structure
```
rocm_nodes/
├── __init__.py          # Package exports
├── nodes.py             # Node registry (NODE_CLASS_MAPPINGS)
├── constants.py         # Project-wide constants
├── core/                # Node implementations
│   ├── __init__.py
│   ├── vae.py          # VAE nodes (decode operations)
│   ├── sampler.py      # Sampler nodes (KSampler, etc.)
│   ├── checkpoint.py   # Checkpoint loader
│   ├── lora.py         # LoRA loader
│   └── monitors.py     # Monitoring/benchmark nodes
└── utils/              # Utility functions
    ├── __init__.py
    ├── memory.py       # Memory management
    ├── diagnostics.py  # ROCm diagnostics
    ├── quantization.py # Quantization detection
    └── debug.py        # Debug utilities
```

### Module Guidelines
1. **One class per file** for complex nodes (>200 lines)
2. **Group related utilities** in utils/ modules
3. **Clear separation**: nodes vs utilities vs constants
4. **Avoid circular imports**: use forward references if needed
5. **Explicit imports**: avoid `from module import *`

## Mandatory Testing (NON-NEGOTIABLE)

### Test Requirements
Every new feature/node MUST have:
1. **Unit tests**: Test individual components in isolation
2. **Integration tests**: Test node interaction with ComfyUI
3. **Performance tests**: Verify no regression in speed/memory
4. **Correctness tests**: Validate output quality

### Test Organization
```
tests/
├── unit/              # Unit tests (test individual functions/classes)
│   ├── test_vae.py
│   ├── test_sampler.py
│   └── test_utils.py
├── integration/       # Integration tests (test node workflows)
│   ├── test_workflows.py
│   └── test_flux.py
└── benchmarks/        # Performance benchmarks
    └── test_performance.py
```

### Test Execution
- Run tests before every commit: `uv run pytest tests/`
- Check coverage: `uv run pytest --cov=rocm_nodes`
- Target: >80% code coverage
- All tests must pass before merging

### Writing Tests
```python
# Good: Focused, isolated, fast
def test_memory_cleanup():
    initial_mem = torch.cuda.memory_allocated()
    simple_memory_cleanup()
    final_mem = torch.cuda.memory_allocated()
    assert final_mem <= initial_mem

# Bad: Too broad, slow, unclear assertions
def test_everything():
    # Tests multiple unrelated things...
    pass
```

## ComfyUI Best Practices

### Node Structure
Every node MUST have:
```python
class MyNode:
    @classmethod
    def INPUT_TYPES(cls):
        return {"required": {...}, "optional": {...}}
    
    RETURN_TYPES = ("TYPE1", "TYPE2")
    RETURN_NAMES = ("output1", "output2") 
    FUNCTION = "process"
    CATEGORY = "RocM Ninodes/Category"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iGavroche/rocm-ninodes](https://github.com/iGavroche/rocm-ninodes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
