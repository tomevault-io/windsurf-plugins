---
trigger: always_on
description: This document provides comprehensive rules and guidelines for AI agents working with the LLM Trainer codebase. Follow these rules strictly to maintain code quality and consistency.
---

# Agent Rules for LLM Trainer Codebase

This document provides comprehensive rules and guidelines for AI agents working with the LLM Trainer codebase. Follow these rules strictly to maintain code quality and consistency.

## Tool Preferences

**ALWAYS use `uv` for package management and tooling:**

- ✅ `uv run <command>` - Run commands in the project environment
- ✅ `uvx <tool>` - Run tools via uvx (e.g., `uvx ty check`)
- ✅ `uv pip install` - Install packages
- ✅ `uv sync` - Sync dependencies

- ❌ NEVER use `pip` directly
- ❌ NEVER use `python -m` for tools (use `uv run` instead)
- ❌ NEVER use `npx` or other package managers

## Code Quality Standards

### Type Checking

**Always run type checking before committing:**
```bash
uvx ty check
```

**Key rules:**
- Use proper type hints: `Callable` from `typing`, not `callable`
- Handle `Optional` types properly - always check for `None` before use
- Use `Union` for multiple types, `Optional[T]` for `Union[T, None]`
- Provide default values for optional parameters when calling functions

### Linting

**Always run linting:**
```bash
uv run ruff check . --select E,F,W
uv run ruff check . --select E,F,W --fix  # Auto-fix when possible
```

**Rules:**
- Follow PEP 8 style guidelines
- Maximum line length: 88 characters (Black default)
- Use `ruff` for all linting - it's fast and comprehensive
- Fix all errors before committing

## Project Structure

```
llm-trainer/
├── src/llm_trainer/          # Main package source
│   ├── config/               # Configuration classes
│   ├── data/                 # Data loading and preprocessing
│   ├── kernels/              # Kernel optimizations
│   ├── models/              # Model architectures
│   ├── tokenizer/           # Tokenizer implementations
│   ├── training/            # Training infrastructure
│   └── utils/               # Utility functions
├── scripts/                 # CLI scripts
├── examples/                # Python examples
├── notebooks/               # Jupyter notebooks
├── configs/                 # YAML configuration files
└── docs/                    # Documentation (essential only)
```

## Code Patterns

### Tokenizer Creation

**ALWAYS use the factory function:**
```python
from llm_trainer.tokenizer import create_tokenizer

# ✅ Correct
tokenizer = create_tokenizer("bpe")
tokenizer = create_tokenizer("bpe", pretrained_path="./tokenizer")

# ❌ Wrong - don't import tokenizer classes directly
from llm_trainer.tokenizer import BPETokenizer  # Avoid
```

### Model Creation

**Use ModelConfig for configuration:**
```python
from llm_trainer.config import ModelConfig
from llm_trainer.models import TransformerLM

model_config = ModelConfig(
    vocab_size=32000,
    d_model=512,
    n_heads=8,
    n_layers=6
)
model = TransformerLM(model_config)
```

### Training Setup

**Use TrainingConfig and Trainer:**
```python
from llm_trainer.config import TrainingConfig
from llm_trainer.training import Trainer

training_config = TrainingConfig(
    batch_size=16,
    learning_rate=1e-4,
    num_epochs=3
)

trainer = Trainer(model, tokenizer, training_config)
```

### ❌ Avoided Patterns

- **Don't use monkey-patching** - Use proper inheritance or composition
- **Don't create unnecessary MD files** - Only essential documentation
- **Don't use `pip` directly** - Always use `uv`
- **Don't import from `__init__` unnecessarily** - Use direct imports when possible

### Notebooks

**All notebooks go in `notebooks/` directory:**
- Use descriptive names: `01_`, `02_`, etc. for ordering
- Keep notebooks focused and small
- Include markdown cells with explanations
- Make notebooks runnable standalone

## Type Safety Rules

### Required Type Hints

**Always provide type hints for:**
- Function parameters
- Return types
- Class attributes (when possible)
- Module-level variables

**Example:**
```python
from typing import List, Optional, Dict, Callable

def process_text(
    texts: List[str],
    tokenizer: BaseTokenizer,
    callback: Optional[Callable[[str], str]] = None
) -> List[int]:
    ...
```

### Handling Optional Types

**Always check for None:**
```python
# ✅ Correct
if dataset_name:
    dataset = load_dataset(dataset_name)

# ❌ Wrong
dataset = load_dataset(dataset_name)  # dataset_name might be None
```

### Default Values

**Provide sensible defaults:**
```python
# ✅ Correct
pad_token_id = getattr(tokenizer, "pad_token_id", None) or 0
eos_token_id = getattr(tokenizer, "eos_token_id", None) or 3

# ❌ Wrong
pad_token_id = getattr(tokenizer, "pad_token_id", None)  # Might be None
```

## Error Handling

### Required Checks

**Always validate:**
- None values before use
- Empty lists/collections before indexing
- Required attributes exist before access
- Dataloaders are not None before iteration

**Example:**
```python
if pbar is None:
    raise ValueError("train_dataloader is None. Please provide training data.")
```

## Import Organization

### Standard Import Order

1. Standard library imports
2. Third-party imports
3. Local application imports

**Example:**
```python
# Standard library
import os
from typing import List, Optional

# Third-party
import torch
from datasets import load_dataset


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HelpingAI/llm-trainer](https://github.com/HelpingAI/llm-trainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
