---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 1. Project Overview

`marktripy` is a Python package for converting Markdown to AST (Abstract Syntax Tree) and back to Markdown. The project is currently in the research phase with comprehensive documentation comparing various Python Markdown processing libraries.

## 2. Project Goals

1. Convert Markdown to HTML programmatically
2. Convert Markdown to an intermediate AST format
3. Manipulate the AST (e.g., downgrade headings, add IDs)
4. Serialize the modified AST back to Markdown
5. Support custom syntax extensions (e.g., `++content++` → `<kbd>content</kbd>`)

## 3. Research Findings

The `ref/` directory contains extensive research on 8+ Python Markdown packages. Key candidates identified:

- **For AST manipulation**: `markdown-it-py`, `marko`, `mistletoe`
- **For performance**: `umarkdown` (C-based), `markdown-it-pyrs` (Rust-based)
- **For extensibility**: `markdown`, `marko`, `markdown-it-py`
- **For round-trip conversion**: `mistletoe`, `marko`, `markdown-it-py`

## 4. Development Commands

**Note**: The project is in research phase - no build/test infrastructure exists yet. When implementation begins, consider:

```bash
# Recommended setup for new Python project
curl -LsSf https://astral.sh/uv/install.sh | sh
uv venv --python 3.12
uv init
uv add fire rich
uv sync
```

## 5. Architecture Considerations

When implementing `marktripy`, consider:

1. **Parser Selection**: Based on research, `markdown-it-py` or `marko` offer the best balance of AST manipulation, extensibility, and round-trip conversion
2. **Plugin Architecture**: Design for extensibility from the start to support custom syntax
3. **AST Structure**: Ensure the AST is easily traversable and modifiable
4. **Round-trip Fidelity**: Preserve formatting and structure when converting back to Markdown

## 6. Key Research Documents

- `ref/ref1.md`: Practical Python Guide to Advanced Markdown Processing
- `ref/ref2.md`: Python Libraries for Markdown Parsing and Extension  
- `ref/ref3.md`: Comprehensive Python Markdown Packages Research Report

These documents contain detailed comparisons, code examples, and recommendations for each library evaluated.


# Software Development Rules

## 7. Pre-Work Preparation

### 7.1. Before Starting Any Work
- **ALWAYS** read `WORK.md` in the main project folder for work progress
- Read `README.md` to understand the project
- STEP BACK and THINK HEAVILY STEP BY STEP about the task
- Consider alternatives and carefully choose the best option
- Check for existing solutions in the codebase before starting

### 7.2. Project Documentation to Maintain
- `README.md` - purpose and functionality
- `CHANGELOG.md` - past change release notes (accumulative)
- `PLAN.md` - detailed future goals, clear plan that discusses specifics
- `TODO.md` - flat simplified itemized `- [ ]`-prefixed representation of `PLAN.md`
- `WORK.md` - work progress updates

## 8. General Coding Principles

### 8.1. Core Development Approach
- Iterate gradually, avoiding major changes
- Focus on minimal viable increments and ship early
- Minimize confirmations and checks
- Preserve existing code/structure unless necessary
- Check often the coherence of the code you're writing with the rest of the code
- Analyze code line-by-line

### 8.2. Code Quality Standards
- Use constants over magic numbers
- Write explanatory docstrings/comments that explain what and WHY
- Explain where and how the code is used/referred to elsewhere
- Handle failures gracefully with retries, fallbacks, user guidance
- Address edge cases, validate assumptions, catch errors early
- Let the computer do the work, minimize user decisions
- Reduce cognitive load, beautify code
- Modularize repeated logic into concise, single-purpose functions
- Favor flat over nested structures

## 9. Tool Usage (When Available)

### 9.1. Additional Tools
- If we need a new Python project, run `curl -LsSf https://astral.sh/uv/install.sh | sh; uv venv --python 3.12; uv init; uv add fire rich; uv sync`
- Use `tree` CLI app if available to verify file locations
- Check existing code with `.venv` folder to scan and consult dependency source code
- Run `DIR="."; uvx codetoprompt --compress --output "$DIR/llms.txt"  --respect-gitignore --cxml --exclude "*.svg,.specstory,*.md,*.txt,ref,testdata,*.lock,*.svg" "$DIR"` to get a condensed snapshot of the codebase into `llms.txt`

## 10. File Management

### 10.1. File Path Tracking
- **MANDATORY**: In every source file, maintain a `this_file` record showing the path relative to project root
- Place `this_file` record near the top:
- As a comment after shebangs in code files
- In YAML frontmatter for Markdown files
- Update paths when moving files
- Omit leading `./`
- Check `this_file` to confirm you're editing the right file

## 11. Python-Specific Guidelines

### 11.1. PEP Standards
- PEP 8: Use consistent formatting and naming, clear descriptive names
- PEP 20: Keep code simple and explicit, prioritize readability over cleverness
- PEP 257: Write clear, imperative docstrings
- Use type hints in their simplest form (list, dict, | for unions)

### 11.2. Modern Python Practices

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twardoch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
