---
trigger: always_on
description: You are building **CodeMap**, a CLI tool that generates structural indexes of codebases to reduce LLM token consumption. The tool creates a `.codemap/` directory that mirrors the project structure, enabling targeted line-range reads instead of full file reads.
---

# CLAUDE.md - Agent Instructions for CodeMap

## Project Context

You are building **CodeMap**, a CLI tool that generates structural indexes of codebases to reduce LLM token consumption. The tool creates a `.codemap/` directory that mirrors the project structure, enabling targeted line-range reads instead of full file reads.

## Versioning (MANDATORY)

**Every commit MUST include a version bump.** Update all three locations:
1. `pyproject.toml` → `version = "X.Y.Z"`
2. `codemap/__init__.py` → `__version__ = "X.Y.Z"`
3. `codemap/tests/test_cli.py` → version assertion string

Follow [Semantic Versioning](https://semver.org/):

| Change Type | Bump | Example | When to use |
|---|---|---|---|
| **MAJOR** (X) | `1.0.0` → `2.0.0` | Breaking API/CLI changes, removed commands, changed output format | Backward-incompatible changes |
| **MINOR** (Y) | `1.0.0` → `1.1.0` | New parser, new CLI command, new symbol type support | New features, backward-compatible |
| **PATCH** (Z) | `1.1.0` → `1.1.1` | Bug fix, accuracy improvement, parser fix, typo fix | Fixes, no new features |

**Examples:**
- Adding a new language parser → **MINOR** bump
- Fixing symbol misclassification → **PATCH** bump
- New CLI command (e.g. `codemap diff`) → **MINOR** bump
- Changing JSON output schema → **MAJOR** bump
- Improving extraction accuracy → **PATCH** bump
- Adding new symbol types to existing parser → **MINOR** bump

## Quick Start Commands

```bash
# Setup
cd codemap
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"

# Run tests
pytest

# Run CLI
codemap --help
codemap init .
codemap find "ClassName"
```

## Using CodeMap to Navigate This Codebase

This project has a `.codemap/` index. **Use CodeMap before scanning files.**

### Start Watch Mode First (EVERY new session — do this immediately)
```bash
pgrep -f "codemap watch" > /dev/null || codemap watch . -q &
```
Auto-starts watch mode with a guard to prevent duplicates.

### Commands
```bash
codemap find "SymbolName"           # Find class/function/method/type by name
codemap find "name" --type method   # Filter by type (class|function|method|interface|type)
codemap show path/to/file.py        # Show file structure with line ranges
codemap validate                    # Check if index is fresh
codemap stats                       # View index statistics
codemap watch . &                   # Start watch mode (auto-updates index)
```

### Workflow
1. **Start watch mode**: `codemap watch . &` (run once per session)
2. **Find symbol**: `codemap find "MapStore"` → `codemap/core/map_store.py:115-507 [class]`
3. **Read targeted lines**: Read only lines 115-507 instead of the full file
4. **Explore structure**: `codemap show codemap/core/map_store.py` to see all methods with line ranges

### When to Use
- **USE CodeMap**: Finding symbol definitions, understanding file structure, locating code by name
- **READ full file**: Understanding implementation details, making edits, unindexed files

### Direct JSON Access
Symbol data is in `.codemap/<path>/.codemap.json` files - read directly for programmatic access.

## Architecture Overview

```
codemap/
├── cli.py                    # Click CLI - entry point
├── core/
│   ├── indexer.py            # Orchestrates indexing
│   ├── hasher.py             # SHA256 file hashing
│   └── map_store.py          # JSON map CRUD operations
├── parsers/
│   ├── base.py               # Abstract Parser class
│   ├── treesitter_base.py    # Config-driven tree-sitter base
│   ├── python_parser.py      # AST-based (stdlib only)
│   ├── typescript_parser.py  # tree-sitter based
│   ├── javascript_parser.py  # tree-sitter based
│   ├── kotlin_parser.py      # tree-sitter based
│   ├── swift_parser.py       # tree-sitter based
│   ├── go_parser.py          # tree-sitter based
│   ├── java_parser.py        # tree-sitter based
│   ├── csharp_parser.py      # tree-sitter based
│   ├── rust_parser.py        # tree-sitter based
│   ├── c_parser.py           # tree-sitter based
│   ├── cpp_parser.py         # tree-sitter based
│   ├── html_parser.py        # tree-sitter based
│   ├── css_parser.py         # tree-sitter based
│   ├── markdown_parser.py    # Regex-based H2/H3/H4 headers
│   └── yaml_parser.py        # Recursive key hierarchy
├── hooks/
│   ├── pre-commit            # Bash script
│   └── installer.py          # Copies hook to .git/hooks/
└── tests/
```

## Implementation Order

Build in this sequence:

### Phase 1: Core Foundation
1. `core/hasher.py` - Simple, no dependencies
2. `parsers/base.py` - Abstract interface
3. `parsers/python_parser.py` - Use stdlib `ast` module only
4. `core/map_store.py` - JSON read/write
5. `core/indexer.py` - Ties everything together

### Phase 2: CLI
6. `cli.py` - Implement commands: init, update, find, show, validate

### Phase 3: Additional Parsers
7. `parsers/typescript_parser.py` - tree-sitter
8. `parsers/javascript_parser.py` - tree-sitter


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AZidan/codemap](https://github.com/AZidan/codemap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
