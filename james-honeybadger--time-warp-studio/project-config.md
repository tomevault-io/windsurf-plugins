---
trigger: always_on
description: **Project:** Time Warp Studio - Educational multi-language programming environment
---

# Time Warp Studio - AI Coding Agent Instructions

**Project:** Time Warp Studio - Educational multi-language programming environment  
**Maintainer:** James Temple <james@honey-badger.org>  
**Last Updated:** April 2026

---

## Project Overview

Time Warp Studio is an educational desktop programming environment built with Python and PySide6 (Qt6) that provides a unified IDE for learning **12 programming languages** with integrated turtle graphics.

**Supported Languages:** BASIC, PILOT, Logo, C, Pascal, Prolog, Forth, Brainfuck, JavaScript, Lua, HyperTalk, Erlang.

**Current State:** Native desktop application (Python/PySide6) - single actively maintained version.

## Architecture: The Big Picture

### Implementation

- **Desktop Application (Python/PySide6)** — primary and only maintained version
    - Entry point: `Platforms/Python/time_warp_ide.py`
    - Core: `Platforms/Python/time_warp/core/interpreter.py`
    - Languages: 12 executors in `Platforms/Python/time_warp/languages/`
    - UI: PySide6 (Qt6) with modern desktop interface (30+ UI modules)
    - All UI state (editor, canvas, themes) managed by main application

**Critical Design Decision:** Language executors are stateless command processors returning text output. All UI state (turtle canvas, output display, themes) lives in the main application, not the interpreter.

## Language Executor Pattern

Each language executor is a **function** (not a class) conforming to the Protocol in `languages/base.py`:

```python
def execute_my_lang(interpreter: Interpreter, source: str, turtle: TurtleState) -> str:
    """Execute source code, return output text with emoji prefixes."""
    ...
```

**Two execution modes:**
- **Whole-program executors** (5 languages): Receive the entire source as a string. Registered in `_WHOLE_PROGRAM_EXECUTORS` dict in `core/interpreter.py`. These are: Lua, Brainfuck, JavaScript, HyperTalk, Erlang.
- **Line-by-line executors** (7 languages: BASIC, PILOT, Logo, C, Pascal, Prolog, Forth): The interpreter iterates lines and calls the executor per statement.

When adding a new whole-program language, only one dict needs updating: `_WHOLE_PROGRAM_EXECUTORS` in `core/interpreter.py`.

## Critical Workflows

### Running the Desktop IDE

```bash
# Primary method
python Platforms/Python/time_warp_ide.py

# Or use the smart launcher (handles venv + deps):
python run.py

# System Requirements
# - Python 3.10+
# - PySide6 (auto-installed if needed)
# - CPU with SSSE3/SSE4.1/SSE4.2/POPCNT support
# Note: Older VMs/QEMU may lack required CPU features
```

### Testing Strategy

```bash
# Comprehensive suite with coverage
python Platforms/Python/test_runner.py --comprehensive

# Quick smoke tests
python Platforms/Python/test_runner.py --basic

# Component-specific
pytest Platforms/Python/time_warp/tests/test_basic_executor.py -v
pytest Platforms/Python/time_warp/tests/test_logo_graphics.py -v

# All demo programs (standalone, 15s timeout per file)
python tests/test_all_demos.py
```

**Test Organization:**
- `test_*.py` = unit tests for components (41 test files in `time_warp/tests/`)
- `Platforms/Python/test_runner.py` = orchestrator with HTML reports -> `test_reports/`
- `tests/test_all_demos.py` = standalone demo verifier (subprocess per file)
- `conftest_lang.py` = shared `run()`, `ok()`, `has()`, `no_errors()` test helpers

### Adding a New Language

1. Create `time_warp/languages/my_lang.py`:
```python
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from ..core.interpreter import Interpreter
    from ..graphics.turtle_state import TurtleState

def execute_my_lang(interpreter: Interpreter, source: str, turtle: TurtleState) -> str:
    """Execute MyLang source code."""
    output_lines = []
    # Parse and execute source
    output_lines.append("✅ MyLang output")
    return "\n".join(output_lines) + "\n"
```

2. Add to `core/interpreter.py`:
   - Import: `from ..languages.my_lang import execute_my_lang`
   - Add `Language.MY_LANG` enum member
   - Add entry in `_init_whole_program_executors()` (for whole-program langs)
   - Add `Language.MY_LANG` in `Language.from_extension()` mapping

3. Add syntax highlighting and file extensions in `ui/editor.py`

## Project-Specific Conventions

### Emoji Prefixes (Universal Pattern)
- `❌` = Errors/exceptions
- `✅` = Success confirmations
- `ℹ️` = Informational messages
- `🎨` = Theme/UI changes
- `🚀` = Execution/run events
- `🐢` = Turtle graphics actions
- `📝` = Input prompts

### Safe Expression Evaluation
**Never use `eval()` directly.** Use the `ExpressionEvaluator` class for math expressions:
```python
from time_warp.utils.expression_evaluator import ExpressionEvaluator
evaluator = ExpressionEvaluator(variables={"X": 5})
result = evaluator.evaluate("2 + 3 * X")  # Returns 17
```

### Hardware/IoT Integration
Simulation-first design in `core/hardware_simulator.py` (not yet wired into main interpreter). Future feature.

## Integration Points to Watch

### Turtle Graphics State
- Turtle state managed via `graphics/turtle_state.py` (593 lines)
- Graphics rendered using Qt painter with zoom/pan support in `ui/canvas.py`
- Canvas clearing happens in main UI, not executor

### Theme System

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [James-HoneyBadger/Time_Warp_Studio](https://github.com/James-HoneyBadger/Time_Warp_Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
