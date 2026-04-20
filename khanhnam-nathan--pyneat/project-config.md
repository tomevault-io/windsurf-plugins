---
trigger: always_on
description: You are working with the PyNeat codebase - an AI-Generated Code Cleaner that scans and fixes code patterns produced by AI assistants.
---

# PyNeat Project - Cursor AI Assistant Guidelines

You are working with the PyNeat codebase - an AI-Generated Code Cleaner that scans and fixes code patterns produced by AI assistants.

## Project Overview

- **Language**: Python (core) + Rust (pyneat-rs backend)
- **Purpose**: Detect and fix AI-generated code issues
- **Supported Languages**: Python, JavaScript, TypeScript, Go, Java, Rust, C#, PHP, Ruby
- **Architecture**: 7-layer protection system with AST Guard, Semantic Guard, Type Shield, etc.

## Code Style Guidelines

### Python

- Follow PEP 8 style guidelines
- Use type hints for function signatures
- Add docstrings for public functions and classes
- Keep lines under 100 characters
- Use `black` for formatting
- Use `ruff` for linting

### Rust (pyneat-rs)

- Use `rustfmt` for formatting
- Write unit tests for new functionality
- Use `clippy` for additional linting
- Follow Rust idioms and best practices

### File Organization

```
pyneat/
├── core/           # Core data structures (AgentMarker, Manifest)
├── rules/          # Rule implementations
│   ├── safe/       # Safe package rules
│   ├── conservative/ # Conservative package rules
│   ├── destructive/ # Destructive package rules
│   └── security/   # Security rules
├── scanner/        # Language-specific scanners
├── cli.py         # Command-line interface
└── benchmark.py   # Performance benchmarks

pyneat-rs/
├── src/
│   ├── scanner/    # Language parsers (tree-sitter based)
│   ├── rules/      # Security and quality rules
│   ├── fixer/      # Auto-fix implementations
│   └── integrations/ # CI/CD integrations
└── benches/       # Criterion benchmarks
```

## Rule Conventions

### Rule ID Format

- Security rules: `SEC-XXX` (e.g., `SEC-001`, `SEC-042`)
- Quality rules: `QUAL-XXX`
- AI bug rules: `AIBUG-XXX`
- Custom rules: `CUSTOM-XXX`

### Creating New Rules

1. Inherit from `AIBugRule` or `SecurityRule` base class
2. Define `RULE_ID`, `SEVERITY`, `LANGUAGES`
3. Implement `detect()` method
4. Optionally implement `fix()` method
5. Register in `ALL_RULES` list
6. Add tests in `tests/` directory

### Rule Severity Levels

- `critical`: Security vulnerabilities, potential RCE
- `high`: Serious issues, data leaks
- `medium`: Code quality problems
- `low`: Minor issues, style suggestions
- `info`: Informational, best practices

## Common AI-Generated Code Patterns to Detect

### Phantom Imports
```python
# Detected patterns
import utils
import helpers
import ai
from ai import something
```

### Fake Parameters
```python
# Detected patterns
def func(param1=None, fake=True, dummy_arg=None)
```

### Identity Comparison
```python
# Should be == not is
if x is 200
if status is "success"
```

### Resource Leaks
```python
# Should use 'with' statement
file = open("data.txt")
conn = connect(url)
```

### Security Issues
```python
# Detected
pickle.loads(user_input)
os.system(user_input)
hashlib.md5(password)
eval(user_input)
yaml.load(config)  # without Loader
```

## Testing Guidelines

### Test Structure
- Unit tests in `tests/test_<module>/`
- Use `pytest` framework
- Include both positive and negative test cases
- Mock external dependencies

### Running Tests
```bash
# All tests
pytest tests/

# Specific test file
pytest tests/test_agent_marker/test_marker_data.py

# With coverage
pytest --cov=pyneat --cov-report=html
```

## CI/CD Integration

### Pre-commit Hooks
- Run `pyneat check` for security scanning
- Run `pyneat clean --dry-run` for code cleanup
- Use `--check` mode for CI

### GitHub Actions
- SARIF export for GitHub Security
- JUnit XML for CI dashboards
- Scheduled security scans

## Contributing

When making changes:
1. Create a feature branch
2. Follow code style guidelines
3. Add tests for new functionality
4. Update documentation
5. Run `pre-commit` checks
6. Submit pull request

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

---
> Source: [khanhnam-nathan/Pyneat](https://github.com/khanhnam-nathan/Pyneat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
