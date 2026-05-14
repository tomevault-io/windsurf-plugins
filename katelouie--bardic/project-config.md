---
trigger: always_on
description: This file provides guidance to Claude Code when working with the Bardic codebase.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the Bardic codebase.

## Project Overview

Bardic is a Python-first interactive fiction engine for modern web applications. It compiles `.bard` story files into JSON for runtime execution, with deployment targets including browser bundles (Pyodide), Python desktop (CLI), and web app frameworks (FastAPI + React, NiceGUI, Reflex).

**Current version:** 0.9.0 — 529 tests, modular engine architecture, browser fork eliminated.

## Python Environment

```bash
pyenv activate bardic     # Always activate before running Python commands
```

Required Python version: 3.10+

## Common Commands

```bash
# Install in development mode
pip install -e .

# Compile a .bard story to JSON
bardic compile story.bard
bardic compile story.bard -o output.json

# Play in terminal
bardic play story.json

# Lint a story (structural analysis, attribute checking)
bardic lint story.bard
bardic lint story.bard --verbose --json-output

# Generate story graph (GraphViz)
bardic graph story.json

# Bundle for browser (Pyodide + HTML)
bardic bundle story.bard
bardic bundle story.bard --theme dark --output dist/

# Initialize a new project from template
bardic init nicegui    # or: web, reflex, browser

# Start dev server (FastAPI + React)
bardic serve story.json

# Run tests
pyenv activate bardic && pytest
pytest tests/test_renderer.py -v          # specific test file
pytest -k "test_undo"                     # by name pattern
```

## Architecture

### Three-Layer Design

```
.bard file → Parser → dict → Compiler → JSON → Engine → PassageOutput
```

1. **Parser** (`bardic/compiler/parser.py` + `bardic/compiler/parsing/`)
   - Parses `.bard` source into intermediate representation
   - Modular: core.py, blocks.py, content.py, directives.py, validation.py, preprocessing.py
   - Handles: passages, choices, variables, @if/@for/@join blocks, @include, @hook, @render directives, @py: blocks, inline conditionals

2. **Compiler** (`bardic/compiler/compiler.py`)
   - Thin wrapper: `compile_file()` and `compile_string()`
   - Resolves @include directives, outputs JSON

3. **Runtime Engine** (`bardic/runtime/` — 8 modules)

### Runtime Module Structure

```
bardic/runtime/
├── engine.py       ~770 lines   Facade: goto(), choose(), current(), trigger_event()
├── renderer.py     ~600 lines   Content token rendering, loops, conditionals, choice filtering
├── executor.py     ~430 lines   Command execution, Python blocks, imports, safe builtins
├── state.py        ~400 lines   Undo/redo stacks, save/load serialization, GameSnapshot
├── directives.py   ~240 lines   @render directive processing, argument binding, React output
├── browser.py      ~130 lines   localStorage save/load adapter (BrowserStorageAdapter)
├── types.py        ~95 lines    PassageOutput, GameSnapshot dataclasses
└── hooks.py        ~75 lines    HookManager for event hook registration
```

**Key design patterns:**
- **Compose by reference** — subsystems share state dicts by reference, mutations visible everywhere
- **Callable providers** — renderer/directives get eval_context/builtins as lambdas, decoupled from engine
- **Environment parameter** — `BardEngine(story_data, environment="browser")` configures for browser vs desktop
- **Thin delegation** — engine keeps one-liner methods that forward to subsystems
- **Mutate in place** — undo/redo/load use `clear() + update()` to preserve shared references

### Engine Public API

```python
engine = BardEngine(story_data)                    # Desktop mode
engine = BardEngine(story_data, environment="browser")  # Browser mode

output = engine.goto("PassageName")   # Navigate + execute + render + cache
output = engine.current()             # Return cached output (safe, idempotent)
output = engine.choose(index)         # Select choice by index + navigate

engine.undo()                         # Restore previous state
engine.redo()                         # Restore undone state
engine.can_undo() / engine.can_redo() # Check availability

engine.save_game(filepath)            # Desktop save
engine.load_game(filepath)            # Desktop load

# Browser mode only (auto-attached):
engine.save_to_browser(slot_name)
engine.load_from_browser(slot_name)
engine.list_browser_saves()
```

**PassageOutput** fields: `content`, `choices`, `passage_id`, `render_directives`, `input_directives`

### Stdlib Modules

```
bardic/stdlib/
├── dice.py          Dice rolling (d6, 2d8+3, advantage/disadvantage)
├── inventory.py     Item management (add, remove, quantity tracking)
├── economy.py       Currency system (earn, spend, transfer)
├── relationship.py  NPC relationships (trust, affinity, thresholds)
└── quest.py         Quest journal (objectives, stages, completion)
```

## Bard Language Syntax

```bard
:: PassageName                          # Passage header
:: Shop(item, price=10)                 # Parameterized passage

Content with {variable} interpolation.
{price:.2f} for format specifiers.
{condition ? "true text" | "false text"} # Inline conditional

~ variable = value                      # Variable assignment
~ health = health - 10                  # Expression assignment

+ [Choice text] -> TargetPassage        # Basic choice (reusable)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [katelouie/bardic](https://github.com/katelouie/bardic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
