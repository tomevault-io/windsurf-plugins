---
trigger: always_on
description: Sloppy detects "AI slop" patterns in Python code - over-engineering, hallucinations,
---

# Sloppy - Python AI Slop Detector

## Project Overview

Sloppy detects "AI slop" patterns in Python code - over-engineering, hallucinations, 
dead code, and anti-patterns commonly produced by LLMs. Inspired by KarpeSlop.

**Core concept**: Three axes of AI slop detection:
1. **Noise** (Information Utility) - redundant comments, debug code, placeholders
2. **Lies** (Information Quality) - hallucinated imports, wrong APIs, assumptions
3. **Soul** (Style/Taste) - overconfident comments, god functions, deep nesting

## Project Structure

```
sloppy/
├── src/sloppy/
│   ├── __init__.py          # Package init, version
│   ├── __main__.py          # Entry point: python -m sloppy
│   ├── cli.py               # Argument parsing (argparse)
│   ├── detector.py          # Main orchestration
│   ├── scoring.py           # Slop index calculation
│   ├── reporter.py          # Terminal + JSON output
│   ├── config.py            # pyproject.toml / .sloppyrc support
│   ├── patterns/
│   │   ├── __init__.py      # Pattern registry
│   │   ├── base.py          # BasePattern class
│   │   ├── noise.py         # Axis 1 patterns
│   │   ├── hallucinations.py # Axis 2 patterns
│   │   ├── style.py         # Axis 3 patterns
│   │   └── structure.py     # Structural anti-patterns
│   └── analyzers/
│       ├── __init__.py
│       ├── ast_analyzer.py  # AST-based detection
│       ├── import_validator.py # Package existence checks
│       └── duplicate_finder.py # Cross-file similarity
├── tests/
│   ├── conftest.py          # Pytest fixtures
│   ├── test_patterns/       # Pattern-specific tests
│   ├── test_analyzers/      # Analyzer tests
│   └── fixtures/            # Sample Python files with known issues
├── pyproject.toml
├── README.md
├── AGENTS.md
└── .github/workflows/ci.yml
```

## Development Environment

### Prerequisites
- Python 3.9+
- No external dependencies for core (stdlib only)
- Optional: `rich` for pretty output

### Setup
```bash
# Clone and setup
git clone https://github.com/rsionnach/sloppy.git
cd sloppy

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install in editable mode
pip install -e ".[dev]"
```

### Dependencies
Core tool uses **stdlib only**:
- `ast` - Python AST parsing
- `re` - Regex patterns
- `pathlib` - File handling
- `difflib` - Similarity detection
- `importlib.util` - Import validation
- `json` - Config and output
- `argparse` - CLI

Dev dependencies:
- `pytest` - Testing
- `pytest-cov` - Coverage
- `rich` - Pretty output (optional runtime)

## Build & Test Commands

```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ --cov=src/sloppy --cov-report=term-missing

# Run specific test file
pytest tests/test_patterns/test_hallucinations.py -v

# Type checking (if mypy added)
mypy src/sloppy

# Format check
black --check src/ tests/
isort --check-only src/ tests/

# Run the tool locally
python -m sloppy .
python -m sloppy src/ --severity high
python -m sloppy --output report.json
```

## Code Style & Conventions

### Python Style
- Follow PEP 8
- Use type hints for all function signatures
- Docstrings for public functions (Google style)
- No external dependencies in core - stdlib only
- Keep functions < 50 lines where possible

### Pattern Implementation
All detection patterns inherit from `BasePattern`:

```python
from sloppy.patterns.base import BasePattern, Issue, Severity

class MutableDefaultArg(BasePattern):
    """Detect mutable default arguments."""
    
    id = "mutable_default_arg"
    severity = Severity.CRITICAL
    axis = "quality"  # noise | quality | style | structure
    message = "Mutable default argument - use None instead"
    
    def check_node(self, node: ast.FunctionDef) -> list[Issue]:
        issues = []
        for default in node.args.defaults:
            if isinstance(default, (ast.List, ast.Dict, ast.Set)):
                issues.append(self.create_issue(node, code=ast.dump(default)))
        return issues
```

### AST Visitor Pattern
Use `ast.NodeVisitor` subclasses for traversal:

```python
class SlopVisitor(ast.NodeVisitor):
    def __init__(self, patterns: list[BasePattern]):
        self.patterns = patterns
        self.issues: list[Issue] = []
    
    def visit_FunctionDef(self, node):
        for pattern in self.patterns:
            if hasattr(pattern, 'check_function'):
                self.issues.extend(pattern.check_function(node))
        self.generic_visit(node)
```

### Regex Patterns
Comment/string patterns use compiled regex:

```python
PATTERNS = {
    'overconfident': re.compile(
        r'#\s*(obviously|clearly|simply|just|easy|trivial)\b',
        re.IGNORECASE
    ),
    'hedging': re.compile(
        r'#\s*(should work|hopefully|probably|might|try this)\b',
        re.IGNORECASE
    ),
}
```

## Testing Guidelines

### Test Organization
- One test file per pattern category
- Use fixtures for sample Python code
- Test both detection AND non-detection (no false positives)

### Fixture Format
Create sample files in `tests/fixtures/`:

```python
# tests/fixtures/mutable_defaults.py
def bad_func(items=[]):  # Should detect
    items.append(1)
    return items

def good_func(items=None):  # Should NOT detect
    if items is None:
        items = []

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rsionnach/sloppylint](https://github.com/rsionnach/sloppylint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
