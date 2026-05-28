---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**FlyLLM** is a comprehensive knowledge base repository focused on **LLM (Large Language Models), Agent systems, and algorithmic problem-solving**. The repository serves as both a technical documentation hub and interview preparation resource.

### Core Content Areas

- **LLM Research & Technical Documentation** - In-depth technical articles on transformers, attention mechanisms, fine-tuning techniques, RLHF, and inference optimization
- **Agent System Design** - Research on agent architectures, tool calling mechanisms, function calling, skill systems, and memory mechanisms
- **Algorithm Practice** - LeetCode problem solutions with detailed explanations and complexity analysis
- **Documentation Subprojects** - Several full-featured ML/NLP projects including transformers library documentation

## Key Directories

### LLM Technical Documentation

- **`/llm/`** - Structured LLM documentation with sequential learning path
  - `001_tokenizer.md` - Tokenization fundamentals
  - `002_self-attention-mechanism.md` - Core attention mathematics
  - `003_mha-gqa-comparison.md` - Attention architecture comparisons
  - `004_rope-alibi-position-encoding.md` - Position encoding methods
  - And more numbered documents covering inference, fine-tuning, and optimization
  - Each file is a self-contained technical deep-dive on a specific topic
  - Covers: Tokenizers, Transformer architecture, Attention mechanisms, Training methods, Inference optimization, RAG systems, Agent frameworks, RLHF, Memory systems

### Algorithm Practice

- **`/leetcode/`** - LeetCode problem solutions and explanations
  - `hot100.md` - Complete categorized list of LeetCode Hot 100 problems
  - Individual problem files using the format: `{problem-number}.{problem-name}.md`
  - Each solution includes problem description, approach, complexity analysis, and implementation
  - Organized by difficulty and topic for systematic preparation

### Documentation Subprojects

- **`/docs/`** - Full-featured documentation and implementation projects
  - **`transformers/`** - Complete HuggingFace transformers library documentation (full project)
  - **`nanochat/`** - Lightweight LLM chatbot implementation
  - **`autoresearch/`** - Automated research and documentation generation tools
  - **`Engram/`** - Memory-related research and implementations

### Supporting Directories

- **`/image/`** - Images, diagrams, and visual assets used across documentation
- **`/interview/`** - Interview preparation materials (currently being organized)

## Common Commands

### Quick Navigation & Search

```bash
# Find files by topic (example: searching for attention-related content)
find . -name "*.md" -type f | xargs grep -l "attention" | head -10

# Search within llm directory
grep -r "LoRA\|lora" --include="*.md" llm/

# List recently modified files
ls -lt $(find . -name "*.md" -type f) | head -10

# Find LeetCode problems by topic
grep -r "dynamic programming" --include="*.md" leetcode/
```

### LLM Documentation Management

```bash
# Find all tokenizer-related documents
find llm -name "*token*" -type f

# Find RLHF training documents
find llm -name "*RLHF*" -o -name "*DPO*" -o -name "*SFT*"

# List numbered documents in sequential order
ls -lh llm/0*.md | awk '{print $9, "-", $5}'
```

### Subprojects

#### Transformers Documentation (`/docs/transformers/`)

Full HuggingFace transformers library documentation project:

```bash
cd docs/transformers

# Run tests
make test

# Check code quality
make quality

# Fix style issues
make style
```

#### NanoChat (`/docs/nanochat/`)

Lightweight LLM chatbot implementation:

```bash
cd docs/nanochat

# Check project-specific documentation in the directory
# for installation and usage instructions
```

#### AutoResearch (`/docs/autoresearch/`)

Automated research and documentation generation tools:

```bash
cd docs/autoresearch

# Check README for tool usage and setup instructions
```

#### Engram (`/docs/Engram/`)

Memory-related research and implementations:

```bash
cd docs/Engram

# Check project documentation for memory system details
```

### LeetCode

Solutions and explanations in `/leetcode/`:

```bash
# View LeetCode Hot 100 categorized list
cat leetcode/hot100.md

# Search for problems by topic (e.g., DP, tree, graph)
grep -r "dynamic programming" --include="*.md" leetcode/ | head -10

# View solution for a specific problem
cat leetcode/1.两数之和.md

# List all problem files
ls leetcode/*.md
```

Each problem file includes:
- Problem description
- Solution approach and intuition
- Time/space complexity analysis
- Code implementation with explanations
- Common pitfalls and edge cases

## Architecture and Design Patterns

### LLM Documentation Structure

The `/llm/` directory covers:

- **Tokenization** - BPE, SentencePiece, WordPiece, and subword algorithms
- **Transformer Architecture** - Attention mechanisms (MHA, GQA, MQA), positional encodings, and optimization
- **Training** - Pretraining, SFT (Supervised Fine-Tuning), LoRA/QLoRA, RLHF, DPO
- **Inference** - KV cache optimization, quantization, continuous batching, vLLM/SGLang

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Decalogue/flyllm](https://github.com/Decalogue/flyllm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
