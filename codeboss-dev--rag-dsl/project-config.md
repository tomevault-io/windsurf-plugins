---
trigger: always_on
description: This project builds **PromptScript**, a lightweight typed DSL for constructing RAG prompts, and benchmarks it against equivalent plain-English prompts across 50 standardized RAG tasks. The goal is to empirically test whether structured prompt construction improves faithfulness, format compliance, and token efficiency compared to freeform English prompts.
---

# PromptScript for RAG — Implementation Plan

## Context

This project builds **PromptScript**, a lightweight typed DSL for constructing RAG prompts, and benchmarks it against equivalent plain-English prompts across 50 standardized RAG tasks. The goal is to empirically test whether structured prompt construction improves faithfulness, format compliance, and token efficiency compared to freeform English prompts.

---

## Project Structure

```
CoSamplingPlayground/
├── pyproject.toml
├── CLAUDE.md
├── .gitignore
│
├── src/promptscript/
│   ├── __init__.py
│   ├── grammar.lark              # Lark EBNF grammar
│   ├── parser.py                 # Lark-based parser
│   ├── ast_nodes.py              # Dataclass AST node definitions
│   ├── transformer.py            # Parse tree -> AST
│   ├── type_checker.py           # Static type validation
│   ├── compiler.py               # AST -> PromptSegments -> output
│   ├── token_budget.py           # tiktoken-based budget enforcement
│   ├── targets/
│   │   ├── __init__.py
│   │   ├── markdown.py           # Markdown prompt renderer
│   │   └── json_api.py           # JSON API body renderer (OpenAI/Anthropic)
│   ├── runtime/
│   │   ├── __init__.py
│   │   ├── context.py            # Variable bindings, retriever stubs
│   │   └── builtins.py           # Built-in functions (retriever.fetch, len)
│   └── cli.py                    # CLI: promptscript compile/check/tokens
│
├── benchmark/
│   ├── dataset/
│   │   ├── corpus/               # Source documents (JSON)
│   │   └── tasks.jsonl           # 50 tasks with ground truth
│   ├── prompts/
│   │   ├── plain_english/        # 50 plain-English prompts (.txt)
│   │   └── promptscript/         # 50 PromptScript files (.ps)
│   ├── retriever/
│   │   ├── __init__.py
│   │   └── bm25.py               # BM25 retriever (rank_bm25)
│   ├── runner.py                 # LLM API orchestration
│   ├── scorer.py                 # 4-metric scoring
│   ├── pipeline.py               # End-to-end evaluation
│   └── config.yaml               # Models, retriever params, paths
│
├── report/
│   ├── analysis.py               # Tables, charts, statistical tests
│   ├── figures/
│   └── report.md
│
└── tests/
    ├── test_parser.py
    ├── test_transformer.py
    ├── test_type_checker.py
    ├── test_compiler_markdown.py
    ├── test_compiler_json.py
    ├── test_token_budget.py
    ├── test_scorer.py
    └── fixtures/                 # .ps files + expected outputs
```

---

## Phase 1: DSL Grammar + Compiler

### 1a. Grammar (`grammar.lark` using Lark EBNF)

**Why Lark:** EBNF syntax, built-in Transformer pattern for tree-to-AST, LALR backend for speed, large community.

Key grammar rules:
- **Declarations**: `type_spec IDENT "=" expression` — types are `str`, `int`, `float`, `bool`, `persona`, `instruct`, `context[]`
- **Parameters**: `set_param IDENT "=" literal` — routes to API envelope, not prompt body
- **Control flow**: `for var in expr { ... }` and `if condition { ... } else { ... }` — braces over indentation for unambiguous parsing
- **Compile call**: `prompt.compile(args...)` — explicit compilation trigger
- **Triple-quoted strings**: `"""..."""` for multi-line instruct blocks
- **Dotted names**: `retriever.fetch(...)` without needing a full object system
- **Comments**: `// single line`

### 1b. AST Nodes (`ast_nodes.py`)

Dataclass-based: `Program`, `Declaration`, `Assignment`, `SetParam`, `ForLoop`, `IfBlock`, `CompileCall`, plus expression types (`StringLiteral`, `NumberLiteral`, `BoolLiteral`, `Identifier`, `FuncCall`, `ListExpr`, `Condition`).

### 1c. Transformer (`transformer.py`)

Lark `Transformer` subclass — maps parse tree nodes to AST dataclasses. ~120-150 lines.

### 1d. Type Checker (`type_checker.py`)

Single-pass AST visitor with a symbol table. Validates:
- Type consistency (e.g., `int x = "hello"` is an error)
- `context[]` must come from a function call returning a list
- `set_param` values must be numeric or boolean literals
- All `prompt.compile()` args must reference declared variables
- Loop variable typing (`for chunk in docs` where `docs: context[]` gives `chunk: context`)

### 1e. Compiler (`compiler.py`)

Two-phase design:
1. **Evaluation**: Walk AST, resolve identifiers, expand loops/conditionals, produce flat list of `PromptSegment(role, content, token_count, metadata)`
2. **Rendering**: Dispatch segments to the selected target renderer

### 1f. Token Budgeting (`token_budget.py`)

Uses `tiktoken` (cl100k_base). When total tokens exceed budget, drops `context[]` segments from lowest confidence first (whole chunks, not mid-chunk truncation). Logs dropped chunks.

### 1g. Render Targets

- **`targets/markdown.py`**: Structured markdown with `## Role`, `## Context`, `## Query`, `## Instructions` sections
- **`targets/json_api.py`**: Ollama/OpenAI-compatible messages array; `set_param` values map to top-level API params (works with Ollama's `/v1/chat/completions` endpoint)

### 1h. CLI (`cli.py` via Click)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CodeBoss-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
