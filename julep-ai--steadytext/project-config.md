---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## General Guidelines

| # | AI may do | AI must NOT do |
|---|-----------|----------------|
| G-0 | Ask for clarification when unsure about project-specific features or decisions | Write changes or use tools when uncertain about project context |
| G-1 | Generate code in relevant source directories (e.g., `steadytext/`, `pg_steadytext/`) | Touch test files or specs without explicit permission |
| G-2 | Add/update `AIDEV-ANCHOR:`, `AIDEV-REF:`, and `AIDEV-NOTE:` comments; keep anchors current | Remove anchor comments without updating references accordingly |
| G-3 | Follow project's lint/style configs (`pyproject.toml`, `.ruff.toml`) | Re-format code to any other style |
| G-4 | Ask for confirmation before changes >300 LOC or >3 files | Refactor large modules without human guidance |
| G-5 | Stay within current task context; suggest starting fresh if needed | Continue work from prior unrelated context |

## Anchor Comments

Structured, greppable comments for navigation and documentation.

**Tags:** `AIDEV-ANCHOR:` (headings), `AIDEV-REF:` (cross-refs), `AIDEV-TODO:` (tasks), `AIDEV-NOTE:` (context), `AIDEV-QUESTION:`, `FIXME:`

**Guidelines:**
- Ultra-short phrases (≤60 chars), no filler words
- Place anchors above major blocks (classes, functions, routes)
- Keep ~1 anchor per 40-60 lines
- Format refs: `AIDEV-REF: path/file.py -> anchor text`
- For documentation rewrite, follow the addendum at `specs/003-documentation-rewrite/anchor-guidelines.md`

> For detailed anchor management, use the **anchor-comment-manager** agent.

## Daemon Architecture

Persistent model serving via ZeroMQ to avoid repeated loading overhead.

### Usage

**CLI Commands:**
```bash
# Start daemon
st daemon start [--host HOST] [--port PORT] [--foreground]

# Check status
st daemon status [--json]

# Stop daemon
st daemon stop [--force]
```

**SDK Usage:**
```python
with use_daemon():
    text = generate("Hello world")
    embedding = embed("Some text")
```

- Remote models skip daemon when unsafe_mode=True

## Models & Configuration

### Standard Models
- **Generation:** Qwen3-4B-Instruct (default), Qwen3-30B-A3B-Instruct (large)
- **Embedding:** Jina v4 (2048-dim → 1024 truncated, requires query/passage prefixes)
- **Reranking:** Qwen3-Reranker-4B

### Mini Models (CI/Testing)
~10x faster: Gemma-3-270M, BGE-large, BGE-base
Enable: `STEADYTEXT_USE_MINI_MODELS=true` or `--size mini`

### Remote Models (Unsafe Mode)
OpenAI, Cerebras, VoyageAI, Jina, OpenRouter - use `model="openai:gpt-4o-mini" unsafe_mode=True`
OpenRouter provides unified access to multiple providers: `model="openrouter:anthropic/claude-3.5-sonnet"`

> **Embedding overrides:** Setting `EMBEDDING_OPENAI_BASE_URL` and `EMBEDDING_OPENAI_API_KEY` auto-routes embeddings (library, CLI, pg extension). Use `EMBEDDING_OPENAI_MODEL` to change the default; explicit `model=` arguments still take precedence.



## Cache Management

**Backends:** SQLite (default), D1 (Cloudflare edge), Memory (testing)

- Factory pattern in `cache/factory.py`, all implement CacheBackend interface

## AI Assistant Workflow

1. **Grep for Anchors** - Understand codebase structure first
2. **Consult CLAUDE.md** - Check project guidelines
3. **Clarify if needed** - Ask targeted questions
4. **Plan approach** - Break down complex tasks
5. **Execute** - Start immediately for trivial tasks, get approval for complex ones
6. **Track progress** - Use AIDEV-TODO comments and TodoWrite tool
7. **Update documentation** - Keep anchors and docs current
8. **Review** - Get user feedback and iterate

## Structured Generation

Generate structured output using llama.cpp grammars - JSON schemas, Pydantic models, regex patterns, choices.

### Examples

```python
from steadytext import generate, generate_json, generate_regex, generate_choice
from pydantic import BaseModel

# JSON with Pydantic model
class Person(BaseModel):
    name: str
    age: int

result = generate("Create a person", schema=Person)
# Returns: "Let me create a person...<json-output>{"name": "Alice", "age": 30}</json-output>"

# Regex pattern matching
phone = generate("My number is", regex=r"\d{3}-\d{3}-\d{4}")
# Returns: "555-123-4567"

# Choice constraints
answer = generate("Is Python good?", choices=["yes", "no", "maybe"])
# Returns: "yes"

# JSON schema
schema = {"type": "object", "properties": {"color": {"type": "string"}}}
result = generate_json("Pick a color", schema)

# Remote models with structured generation (v2.6.2+)
result = generate_json(
    "Create a person", 
    Person,
    model="openai:gpt-4o-mini",
    unsafe_mode=True
)
```

## Versioning Policy

**Date-based versioning:** `yyyy.mm.dd` format (e.g., `2025.8.27`)

Applies to both Python package and pg_steadytext extension. See CHANGELOG.md for version history.

> For release management, use the **github-pr-manager** agent.

## Prompt Registry

Jinja2-based template management for PostgreSQL with immutable versioning and automatic variable extraction.

**Key Functions:**
```sql
-- Management functions
SELECT st_prompt_create(slug, template, description, metadata);
SELECT st_prompt_update(slug, template, metadata);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [julep-ai/steadytext](https://github.com/julep-ai/steadytext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
