---
trigger: always_on
description: **Analysis Date:** 2026-03-15
---

# Coding Conventions

**Analysis Date:** 2026-03-15

## Naming Patterns

**Files:**
- Python: snake_case (`paper_store.py`, `llm_service.py`, `schema_builder.py`)
- TypeScript/React: camelCase for components and utilities (`SentimentChart.tsx`, `auth.ts`)
- Test files: `test_*.py` or `*.test.ts` (match the module name: `test_paper_judge.py` for `paper_judge`)
- Directories: lowercase snake_case (`application`, `infrastructure`, `domain`, `agents`)

**Functions/Methods:**
- Python: snake_case
  - Actions: `create_`, `update_`, `delete_`, `fetch_`, `parse_`
  - Checks: `is_`, `has_`, `can_`
  - Internal: `_private_method()` for module-private; single underscore only (not double)
- TypeScript: camelCase
  - Event handlers: `handle*` (e.g., `handleClick`)
  - Getters: plain name or `get*` (e.g., `getSession()`)
  - Async functions: same casing, clearly typed as `Promise<T>`

**Variables:**
- Python: snake_case for all (module-level, local, instance)
- TypeScript: camelCase for locals and module-level; use `const` by default
- Boolean prefixes in Python: `is_`, `has_`, `can_` (e.g., `has_code`, `is_active`)
- Abbreviations: Avoid; spell out full names (use `service` not `svc`)

**Types:**
- Python Dataclasses: PascalCase (e.g., `PaperMeta`, `Scholar`, `Track`)
- Python Protocol/Interface types: Suffix with `Port` (e.g., `RegistryPort`, `EventLogPort`)
- TypeScript Interfaces: PascalCase (e.g., `SentimentChartProps`)
- Enum members (Python): SCREAMING_SNAKE_CASE (e.g., `MUST_READ`, `WORTH_READING`)

## Code Style

**Formatting:**
- **Python**: Black (line-length 100, target py310)
  - Run: `python -m black .`
- **TypeScript**: ESLint with Next.js config
  - Run: `npm run lint` (in `web/` dir)
  - Uses latest eslint v9 with flat config format
- **Indentation**: 4 spaces (Python), 2 spaces (TypeScript/JavaScript)

**Linting:**
- **Python**: pyright (basic mode, Python 3.10)
  - Config in `pyproject.toml` with `extraPaths = ["src"]`
  - Run: `pyright src/`
- **TypeScript**: ESLint with `eslint-config-next/core-web-vitals` and `eslint-config-next/typescript`
  - Config: `web/eslint.config.mjs` (flat config)
  - Ignores: `.next/`, `out/`, `build/`, `next-env.d.ts`

**isort (Python):**
- Profile: Black
- Line length: 100
- src_paths: `["src"]`

## Import Organization

**Python Order:**
1. Future imports (`from __future__ import ...`)
2. Standard library (`import os`, `from typing import`)
3. Third-party (`from pydantic import`, `from sqlalchemy import`)
4. Local application (`from paperbot.domain import`, `from paperbot.infrastructure import`)
5. Relative imports (rare; use absolute paths to `src.paperbot`)

**Example:**
```python
from __future__ import annotations

import json
import logging
from typing import Any, Dict, Optional

from pydantic import BaseModel
from sqlalchemy import Column, String

from paperbot.domain.paper import PaperMeta
from paperbot.infrastructure.stores.models import PaperModel
```

**TypeScript Order:**
1. Built-in React/Next imports
2. External libraries (UI, utilities)
3. Local application imports
4. Relative imports (rare; use `@/*` alias)

**Path Aliases:**
- Python: None (use absolute `from src.paperbot...` or `from paperbot...` after setting PYTHONPATH)
- TypeScript: `@/*` maps to `web/src/*` (declared in `web/tsconfig.json`)

## Error Handling

**Patterns:**
- **Exceptions**: Use built-in exceptions when appropriate; create domain-specific exceptions in `domain/` if needed
- **Try/Except**: Catch specific exceptions; use `except Exception as exc:` with logging for fallbacks, not silently ignoring
- **Logging on Error**: Always log at minimum `warning` level before falling back
  ```python
  except Exception as exc:
      logger.warning("Operation failed: %s", exc)
      return fallback_value
  ```
- **Async Errors**: Use `except Exception as exc:` (async functions throw like sync); no special async exception handling needed
- **API Errors**: FastAPI routes catch exceptions and return appropriate HTTP status codes; use `HTTPException(status_code=..., detail=...)` for client errors

## Logging

**Framework:** Python built-in `logging` module

**Pattern:**
```python
import logging

logger = logging.getLogger(__name__)
```

**Usage:**
- Always use `logger.info()`, `logger.warning()`, `logger.debug()`, `logger.error()` — never `print()`
- Use %-formatting: `logger.warning("key=%s value=%d", key, value)` not f-strings (avoids unnecessary work if log level filters it)
- Log at appropriate levels:
  - `debug`: Detailed internal state (rarely needed)
  - `info`: Significant events (task start/end, key decisions)
  - `warning`: Recoverable errors, fallbacks, missing optional data
  - `error`: Unrecoverable errors (rarely used in this codebase; most raise exceptions instead)

**Do NOT log:**
- Secrets (API keys, tokens, passwords)
- PII (full email addresses, user IDs in sensitive contexts)
- Full payloads (log summary only, e.g., `status=200 latency_ms=45`)

## Comments

**When to Comment:**
- Non-obvious logic or algorithms
- Business rules that aren't self-evident from code
- References to external docs (links to issue, architecture doc, paper)
- Workarounds or temporary code: use `# TODO:` or `# FIXME:` with issue number
- Do NOT comment obvious code: `x = 5  # Set x to 5` is noise

**Docstrings (Python):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jerry609/PaperBot](https://github.com/jerry609/PaperBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
