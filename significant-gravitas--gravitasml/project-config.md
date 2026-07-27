---
trigger: always_on
description: GravitasML is a lightweight Python library for parsing custom markup languages, built specifically for LLM-generated outputs. It uses a two-stage parsing approach (tokenization → parsing) with Pydantic integration for validation.
---

# GravitasML
GravitasML is a lightweight Python library for parsing custom markup languages, built specifically for LLM-generated outputs. It uses a two-stage parsing approach (tokenization → parsing) with Pydantic integration for validation.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap, Build, and Test the Repository:
- Install dependencies: `poetry install` -- NEVER CANCEL. Set timeout to 60+ seconds for safety
- Build: `poetry build` -- NEVER CANCEL. Set timeout to 60+ seconds for safety
- Lint: `poetry run black . --check` -- quick execution
- Test with pytest: `poetry run pytest -v -s ./tests` -- NEVER CANCEL. Set timeout to 60+ seconds for safety  
- Test with unittest: `poetry run python -m unittest discover -v` -- Alternative to pytest

### Requirements:
- Python 3.10+ (tested in CI with 3.10, 3.11, 3.12)
- Poetry for dependency management (install with `pip install poetry` if not available)
- All dependencies managed through `pyproject.toml`

### Key Dependencies:
- Pydantic 2.x (for model validation features)
- Black (for code formatting)
- Pytest (for testing, though unittest also works)

## Validation

### Always manually validate any new code with these scenarios:
1. **Basic parsing**: Test simple markup like `<name>John</name><age>30</age>`
2. **Nested structures**: Test complex nested markup like `<person><name>John</name><address><street>123 Main</street></address></person>`
3. **Repeated tags**: Test `<collection><item>A</item><item>B</item></collection>` (creates list structure)
4. **Multiple roots**: Test `<item>A</item><item>B</item>` (creates list of dictionaries)
5. **Error handling**: Test malformed markup (unclosed tags, mismatched tags) to ensure proper SyntaxError exceptions
6. **Pydantic integration**: Test converting parsed data to Pydantic models

### Manual Validation Example:
```python
poetry run python -c "
from gravitasml.token import tokenize
from gravitasml.parser import Parser
from pydantic import BaseModel

# Test basic functionality
markup = '<name>John</name><age>30</age>'
tokens = tokenize(markup)
parser = Parser(tokens)
result = parser.parse()
print('Result:', result)

# Test Pydantic integration
class Person(BaseModel):
    name: str
    age: str

tokens = tokenize(markup)
parser = Parser(tokens)
person = parser.parse_to_pydantic(Person)
print('Pydantic result:', person)

# Test repeated tags (creates list structure)
markup = '<collection><item>A</item><item>B</item></collection>'
tokens = tokenize(markup)
parser = Parser(tokens)
result = parser.parse()
print('Repeated tags result:', result)
# Result: {'collection': [{'item': 'A'}, {'item': 'B'}]}
"
```

### ALWAYS run these validation steps before committing:
- `poetry run black . --check` -- CI will fail if code is not Black-formatted
- `poetry run pytest -v -s ./tests` -- to ensure all tests pass (most should pass, some expected failures are normal)

## Architecture

### Core Components:
- **gravitasml.token** - Tokenization module that converts raw markup into token streams
- **gravitasml.parser** - Parsing module that builds tree structures and converts to Python objects

### Two-Stage Parsing Process:
1. **Tokenization**: Raw markup → stream of tokens (TAG_OPEN, TEXT, TAG_CLOSE)
2. **Parsing**: Token stream → Python dictionaries/lists or Pydantic models

### Key Features:
- Simple API for parsing markup to dictionaries
- Pydantic integration for validation  
- Nested structure support
- Tag normalization (automatic whitespace handling, case conversion)
- Error detection for unmatched/improperly nested tags
- Security by design (no DTD, no external entities, no namespaces)

### Parsing Behavior:
- **Single root with unique children**: `<a>1</a><b>2</b>` → `{'a': '1', 'b': '2'}`
- **Multiple root elements**: `<item>A</item><item>B</item>` → `[{'item': 'A'}, {'item': 'B'}]`
- **Repeated tags in single root**: `<root><item>A</item><item>B</item></root>` → `{'root': [{'item': 'A'}, {'item': 'B'}]}`
- **Nested structures**: `<person><name>John</name></person>` → `{'person': {'name': 'John'}}`
- **Mixed content**: `<root><a>1</a><b>2</b><a>3</a></root>` → `{'root': [{'a': '1'}, {'b': '2'}, {'a': '3'}]}`

## Common Tasks

### Repository Structure:
```
.
├── README.md
├── pyproject.toml
├── poetry.lock
├── LICENSE
├── SECURITY.md
├── .github/
│   └── workflows/
│       └── cicd.yml
├── gravitasml/
│   ├── __init__.py
│   ├── token.py      # Tokenizer implementation
│   └── parser.py     # Parser implementation
└── tests/
    ├── __init__.py
    ├── test_tokenizer.py
    └── test_parser.py
```

### Development Workflow:
1. Make changes to code
2. Run `poetry run black .` to format code (or `poetry run black . --check` to verify)
3. Run `poetry run pytest -v -s ./tests` to run tests
4. Manually test changes with validation scenarios above
5. Commit changes

### CI/CD Pipeline (.github/workflows/cicd.yml):
- Tests on Python 3.10, 3.11, 3.12
- Build → Lint → Test → Publish workflow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Significant-Gravitas/gravitasml](https://github.com/Significant-Gravitas/gravitasml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
