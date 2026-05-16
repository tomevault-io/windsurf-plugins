---
trigger: always_on
description: - **No emojis** in code, display names, or documentation
---

# WanVideo Activation Editor Project

## Code and Writing Style Guidelines

- **No emojis** in code, display names, or documentation
- Keep all naming and display text professional
- Avoid "Enhanced", "Advanced", "Ultimate" type prefixes - use descriptive names instead
- Clean, simple node names that describe what they do
- Stay professional and concise in writing and in code
- Simplify and clean up code where possible
- Be concise in README.md and other documentation except when detail is important. Avoid the 'no shit' parts of READMEs and keep the tone professional and to the point and direct. No hype or hyperbole.

## Overview
This project implements block-level activation editing and vector arithmetic operations for WanVideo models through ComfyUI nodes. It allows injecting alternative text conditioning into specific transformer blocks and performing mathematical operations on embeddings to create novel effects.

## Current Status
The runtime patching system is fully functional and activation injection is confirmed working. Testing shows the injection successfully replaces context in activated blocks with strength=1.0.

**What's Working:**
1. Runtime patching of all 40 transformer blocks
2. Forward method interception confirmed during generation
3. Context parameter successfully captured (shape: [1, 512, 5120])
4. Injection successfully replaces context when strength=1.0
5. Block-level activation configuration stored correctly
6. Vector arithmetic operations with automatic shape alignment
7. DuckDB storage with compression (3-4x reduction)
8. Memory-efficient operations preventing VRAM leaks
9. Debug system with runtime log level control

**Key Insights from Testing:**
- With strength=1.0, context is completely replaced by injection embeddings
- Context norm varies significantly between prompts (18.2 vs 106.8 observed)
- Injection norm remains consistent (~23.2)
- Post-projection differences vary (1.2% to 22.6%) due to projection normalization
- The system is working correctly - injection replaces context as designed

**Recent Fixes:**
- Fixed model path discovery (`model.blocks` instead of `transformer_blocks`)
- Fixed forward method signature for WanAttentionBlock
- Added `@torch.compiler.disable` to prevent torch dynamo issues
- Simplified debug system with built-in console output
- Fixed all `verbose_only` parameter errors
- Added runtime log level control in WanVideoActivationEditor node
- Added injection mode selection (context vs hidden states vs both)
- Added WanVideoInjectionTester for debugging effectiveness

**Understanding the Projection Issue:**
The text_embedding layer normalizes embeddings differently based on content:
- Same prompts can have vastly different norms after projection
- This affects how much "difference" is measured post-projection
- But injection still works - it replaces the context regardless

**Solutions for Better Control:**
1. **Embedding Amplifier**: Ensures raw embeddings differ by target amount
2. **Projection Booster**: Amplifies differences after projection
3. **Latent Encoder/Injector**: Bypasses projection using model's internal representations
4. **Direct Injector**: Forces specific difference levels in context

**Next Steps:**
- Experiment with different prompts and strengths
- Document which blocks produce which effects
- Test latent injection for stronger effects
- Map block sensitivities empirically

## Architecture

### Core Systems
- **Block-level injection**: Stores activation configuration in transformer_options for WanVideoWrapper to process
- **Vector arithmetic**: Mathematical operations on T5 embeddings with automatic shape alignment
- **DuckDB storage**: Persistent embedding cache with zstd compression and SQL analytics
- **Memory management**: Context managers and immediate cleanup to prevent VRAM leaks

### Data Flow
1. **Text Encoding**: WanVideoTextEncode creates raw T5 embeddings [seq_len, 4096]
2. **Vector Operations**: Optional arithmetic operations on embeddings
3. **Configuration**: WanVideoActivationEditor adds injection config to model and embeds
4. **Storage**: Embeddings cached in DuckDB with compression
5. **Sampling**: WanVideoSampler passes embeddings to transformer
6. **Processing**: Transformer processes raw embeddings through text_embedding layer
7. **Injection**: Blocks can access config via transformer_options (requires WanVideoWrapper modification)

## Key Components

### Activation Editing Nodes

#### WanVideoActivationEditor
Main node that configures the model and text embeddings for activation injection. 

**Key Features:**
- Runtime log level control (off/basic/verbose/trace)
- Injection mode selection:
  - `context`: Injects into cross-attention context (default, proven working)
  - `hidden_states`: Injects into transformer hidden states (experimental)
  - `both`: Injects into both context and hidden states
- Embedding difference measurement with warnings for low differences

Stores configuration in:
- `model.model_options.transformer_options['wan_activation_editor']`
- `text_embeds['wan_activation_editor']`

#### WanVideoBlockActivationBuilder
Helper node for creating block activation patterns. Provides presets:
- `early_blocks`: First 10 blocks
- `mid_blocks`: Middle 20 blocks
- `late_blocks`: Last 10 blocks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fblissjr/ComfyUI-WanActivationEditor](https://github.com/fblissjr/ComfyUI-WanActivationEditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
