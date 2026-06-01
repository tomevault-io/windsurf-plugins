---
trigger: always_on
description: - **IMPORTANT: Respect nnsight execution order**: ALL code examples must access components in forward pass order (layers_output[1] before layers_output[2], attention before layer output of same layer, etc.)
---

## Sphinx Documentation Guidelines
- **IMPORTANT: Respect nnsight execution order**: ALL code examples must access components in forward pass order (layers_output[1] before layers_output[2], attention before layer output of same layer, etc.)
- **Use demo.py tone**: Keep explanations factual and concise, avoid verbose language like "core philosophy" or "research-first design"
- **nnterp is for transformers**: Describe it as a nnsight wrapper specifically for transformer models, not general mechanistic interpretability

## Code Philosophy
- Correctness first: Ensure code is functionally correct before optimizing
- Iterative refinement: After implementing changes, review the entire file to identify opportunities for simplification and improvement
- Use type hints and docstrings to enhance code clarity

## Research Context
You assist me - a researcher - with a research oriented library, not production systems. This context allows for specific approaches:
- Make reasonable assumptions based on common research practices and my instructions. Avoid writting fallbacks in case something is missing. THIS IS VERY IMPORTANT as you shouldn't create bloated code!
- Fail fast philosophy: Design code to crash immediately when assumptions are violated rather than silently handling errors. This means that you should only use try/catch blocks if it explicitely benefits the code logic. No need to state this in comments. DON'T WRITE FALLBACKS FOR NON-COMMON INPUTS! Instead write asserts for you assumptions. This is very important!
        - Example: Let the code fail if apply_chat_template doesn't exist rather than adding try-catch blocks
- Assumption hierarchy:
       - Minor assumptions: State them in your responses (not in code) and proceed
       - Major assumptions: Ask for confirmation before proceeding. Depending on the severity state them in code using comments.
- If you are working with tensors, INCLUDE SHAPE ASSERTIONS in your code. For example, you could write "assert x.shape = (batch_size, self.dictionary_size)".
- It is crucial that you only implement what I asked for. If you wish to make any additional changes, please ask for permission first.
- It is fine if you fail to implement something. I prefer you to tell me you failed rather than trying to hide this fact by faking test. Don't reward hack, Claude :<.

## Test Philosophy
- Tests should FAIL! When writing tests, you should NEVER use try except blocks. Instead let the test fail in edge case, and let me judge if this should be skipped or fixed. NEVER EVER AGAIN REWARD HACKING WITH TRY CATCH IN TEST CLAUDE, OK???
- Never try to fix a test by considering it an edge case and skipping it. I consider that reward hacking. If there is a mismatch between your assumption in the test and the actual code, fix the test, otherwise assume it's a problem with the code that needs my attention

## Development Commands

### Package Management
- `uv install` - Install dependencies
- `uv run python -m pytest` - Run all tests
- `uv run python -m pytest tests/test_interventions.py` - Run specific test file
- `uv run python -m pytest tests/test_interventions.py::test_logit_lens` - Run specific test

### Code Quality
- `uv run black .` - Format code with Black (line length 88)
- `uv run python -m build` - Build package for distribution

### Documentation
- `cd docs && make html` - Build Sphinx documentation
- `cd docs && make clean` - Clean documentation build files

## Architecture Overview

nnterp is a mechanistic interpretability library built on top of nnsight, providing a unified interface for transformer analysis through several key components:

### Core Components

**nnsight** `nnterp` is built on top of `nnsight`. A very important thing about `nnsight` is that interventions in a trace **MUST BE WRITTEN IN ORDER**. This means e.g. you can't access the output of a layer and then access its input / its mlp output.

**StandardizedTransformer** (`standardized_transformer.py`)
- Unified interface for different transformer architectures (extends `nnsight.LanguageModel`)
- Standardizes module naming across models (layers, attention, MLP components)
- **Primary model loading method**: Use `StandardizedTransformer("model_name")` instead of deprecated `load_model()`
- **Attention probabilities**: Opt-in with `enable_attention_probs=True` (automatically sets `attn_implementation="eager"`)

**Key Accessors**:
- `layers_input[i]` / `layers_output[i]` - Layer I/O
- `attentions[i]` / `attentions_input[i]` / `attentions_output[i]` - Attention modules
- `mlps[i]` / `mlps_input[i]` / `mlps_output[i]` - MLP modules
- `token_embeddings` - Token embedding layer (read/write)
- `logits` - Final model logits
- `next_token_probs` - Softmax of last token logits
- `input_ids`, `attention_mask`, `input_size` - Input tensor accessors

**Key Methods**:
- `skip_layer(layer_idx)` / `skip_layers(layer_indices)` - Skip layer computation
- `project_on_vocab(hidden_state)` - Project to vocabulary space
- `get_topk_closest_tokens(hidden_state, k)` - Get k closest tokens

**Intervention Framework** (`interventions.py`)

**Key Functions**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ndif-team/nnterp](https://github.com/ndif-team/nnterp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
