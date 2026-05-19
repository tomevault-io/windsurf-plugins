---
trigger: always_on
description: Research-backed AI text improvement framework developer
---


# AGENTS.md - AI Agent Guide

**Purpose**: Quick reference for working on Sifaka
**Last Updated**: 2025-01-21

---

## Quick Start (First Session Commands)

**New to this repo? Run these 5 commands first:**

```bash
# 1. Verify you're on a feature branch (NEVER work on main)
git status && git branch

# 2. Run all quality checks
pytest --cov=sifaka --cov-report=term-missing
mypy sifaka/
ruff check .
black .

# 3. Run specific critic test to verify environment
pytest tests/critics/test_reflexion.py -v

# 4. Check for any TODOs or placeholders (should be NONE)
grep -r "TODO\|FIXME\|NotImplementedError" sifaka/ || echo "✅ No placeholders found"

# 5. Verify coverage is >80%
pytest --cov=sifaka | tail -1
```

---

## Quick Orientation

**Sifaka**: AI text improvement through research-backed critique with complete observability (v0.2.0-alpha)
**Stack**: Python 3.10+, PydanticAI 1.14+, provider-agnostic (OpenAI/Anthropic/Google/Groq)
**Coverage**: 85%+ test coverage, strict mypy, comprehensive examples

### Directory Structure

```text
sifaka/
├── sifaka/
│   ├── core/
│   │   ├── config/          # Configuration management
│   │   └── engine/          # Improvement engine
│   ├── critics/
│   │   └── core/            # Critique implementations (Reflexion, Constitutional AI, etc.)
│   ├── storage/             # Storage backends (file, redis)
│   ├── tools/               # Utility tools
│   └── validators/          # Validation logic
├── examples/                # Usage examples
├── tests/                   # Unit + integration tests
└── pyproject.toml           # Dependencies and config
```

---

## Critical Rules

### 1. Research-Backed Critique Pattern
All critics implement research-backed techniques (Reflexion, Constitutional AI, Self-Refine).

```python
from sifaka.critics.core import BaseCritic

class MyCritic(BaseCritic):
    """Implement a specific research-backed critique technique."""

    async def critique(self, text: str, context: dict) -> CritiqueResult:
        """Apply critique to text.

        Args:
            text: Text to critique
            context: Additional context for critique

        Returns:
            CritiqueResult with feedback and improvement suggestions
        """
        # Implementation following research methodology
        pass
```

### 2. Provider-Agnostic Design
Must work with ANY LLM provider (OpenAI, Anthropic, Google, Groq).

```python
# ✅ GOOD
from sifaka import improve_sync
result = improve_sync("Text to improve", provider="anthropic", model="claude-3-5-sonnet")

# ❌ BAD
from openai import OpenAI
client = OpenAI()  # Hardcoded to OpenAI
```

### 3. Complete Observability
All improvement operations must provide full audit trails.

```python
result = improve_sync("Text to improve")
# Access complete trace
for iteration in result.trace:
    print(f"Iteration {iteration.number}: {iteration.improvement}")
```

### 4. Type Safety (Strict Mypy)
All functions require type hints, no `Any` without justification.

### 5. No Placeholders/TODOs
Production-grade code only. Complete implementations or nothing.

### 6. Complete Features Only
If you start, you finish:
- ✅ Implementation complete
- ✅ Tests (>80% coverage)
- ✅ Docstrings
- ✅ Example code
- ✅ Exported in `__init__.py`

### 7. PydanticAI for Structured Outputs
All critics and validators use PydanticAI for type-safe LLM responses.

---

## Boundaries

### ✅ Always Do (No Permission Needed)
- Run tests: `pytest tests/`, `pytest --cov=sifaka`, `pytest -v`
- Format code: `black .`
- Lint code: `ruff check .`
- Type check: `mypy sifaka/` (strict mode required)
- Add unit tests for new critics in `tests/critics/`
- Add integration tests in `tests/integration/`
- Update docstrings when changing function signatures
- Export new critics in `__init__.py` files
- Add examples to `examples/` for new user-facing features
- Update audit trail documentation for new features

### ⚠️ Ask First

**Core Architecture** (Why: Affects all critique operations):
- Add new critics to `sifaka/critics/core/` - Must follow research-backed patterns
- Modify improvement engine in `sifaka/core/engine/` - All critics depend on this
- Change research-backed critique methodologies (Reflexion, Constitutional AI, Self-Refine) - Research validity at stake
- Update public API in `sifaka/__init__.py` (improve, improve_sync functions) - Breaking changes for users

**Observability & Storage** (Why: Audit trail integrity):
- Change observability/audit trail implementation - Complete traceability required
- Modify storage backends in `sifaka/storage/` - Data persistence implications
- Change validation logic in `sifaka/validators/` - Quality control affected

**Dependencies & Config** (Why: Security and maintenance burden):
- Add/update dependencies in `pyproject.toml` - Increases attack surface
- Modify configuration management in `sifaka/core/config/` - System-wide effects
- Update `README.md` examples or API documentation - User-facing changes

### 🚫 Never Touch

**CRITICAL SECURITY VIOLATION** ⚠️:
- **NEVER EVER COMMIT CREDENTIALS TO GITHUB**
- No API keys, tokens, passwords, secrets in ANY file
- No credentials in code, documentation, examples, tests, or configuration files
- Use environment variables (.env files in .gitignore) ONLY

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sifaka-ai/sifaka](https://github.com/sifaka-ai/sifaka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
