---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hawk is an infrastructure system for running Inspect AI evaluations and Scout scans in Kubernetes. It consists of:

- A `hawk` CLI tool for submitting evaluation and scan configurations
- A FastAPI server that orchestrates Kubernetes jobs using Helm
- Multiple Lambda functions for log processing, access control, and sample editing
- Terraform infrastructure for AWS resources
- A PostgreSQL data warehouse for evaluation results

## Quick Decision Guide

**Before starting any task, follow this checklist:**

1. ✅ **Read files first** - Never propose changes without inspecting the actual code
2. ✅ **Understand context** - Use Grep/Glob to find related code and patterns
3. ✅ **Scout mindset** - Fix what's requested + low-cost cleanup (typos, unused imports, obvious bugs)
4. ✅ **Add tests** - Run tests before declaring completion
5. ✅ **Run quality checks** - Ensure ruff, basedpyright, and tests pass

**Common scenarios:**

| If the task is...         | Then...                                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------------------------ |
| Adding an API endpoint    | Read Security Requirements → Add auth dependency → Implement logic → Add tests                         |
| Fixing a bug              | Read relevant files → Add a test to reproduce the bug → Make minimal fix → Run tests to verify the fix |
| Adding CLI command        | Check Common Code Patterns → Follow CLI pattern → Update docs                                          |
| Modifying database schema | Update model → Create Alembic migration → Test upgrade/downgrade against a local database              |
| Adding config field       | Update Pydantic model → Update examples / regenerate schemas → Document in README                      |
| Debugging stuck eval      | Check pod logs → Analyze sample buffer → Test API directly → See Debugging Stuck Evaluations section   |

**When in doubt:**

- Check existing patterns in the codebase (use Grep to find similar code)
- Refer to Common Code Patterns section below
- Review Common Mistakes to Avoid section

**Note:** Hawk only runs on Linux and macOS. There is no need for Windows compatibility workarounds.

## Coding Standards

### Import Style

Import submodules, not functions/classes:

```python
# ✓ Good
import hawk.core.types.evals as evals

# ✗ Avoid
from hawk.core.types.evals import EvalSetConfig

# Exception: Type hints in TYPE_CHECKING blocks, or imports from `typing` or `collections.abc`
if TYPE_CHECKING:
    from hawk.core.types import EvalSetConfig
```

### Documentation

Update README.md, CLAUDE.md, and `examples/` when adding features or changing schemas.

### Security Requirements

**All API endpoints MUST have authorization.** Add auth dependency first, before implementing logic:

```python
from typing import Annotated
from hawk.api.auth import auth_context
from hawk.api import state

@app.get("/my-endpoint")
async def my_endpoint(
    auth: Annotated[auth_context.AuthContext, fastapi.Depends(state.get_auth_context)]
):
    # Validate permissions: permissions.validate_permissions(auth.permissions, {...})
```

**Model Access Control:** Access to models and eval logs is controlled by `model_groups`:

- To **use a model**: User must belong to that model's model_group
- To **view eval logs**: User must have access to all model_groups used in that eval set's folder (stored in `.models.json`)
- To **launch scans**: User must have access to all model_groups in the target eval set's folder

## Development Workflow

### Before Making Changes

**Read files first.** Never propose changes without inspecting the actual code. Use Read/Grep/Glob to understand context before making changes.

### Minimum Viable Changes

Fix what's requested, but **leave the code better than you found it** when the cost is low and risk is minimal.

**✓ Encouraged cleanup (same file/function you're already editing):**

- Fix typos in comments or docstrings
- Remove unused imports
- Fix obvious bugs you notice (if trivial)
- Improve variable names that are genuinely confusing
- Add missing type hints to functions you're modifying

**✓ Encouraged cleanup (separate commit in same PR):**

- Consistent cleanup across multiple files (e.g., fixing typo in many comments)
- Removing genuinely dead code
- Explain in commit message: "cleanup: remove unused helper function"

**✗ Ask first or suggest separately:**

- Refactoring function signatures or abstractions
- Restructuring modules or files
- Adding features not requested
- Changes that affect tests in non-obvious ways

**When making cleanup changes:**

- Keep cleanup commits separate from functional changes when practical
- Mention what cleanup you're doing: "Also fixed typo in docstring while here"
- If unsure whether cleanup is appropriate, suggest it to the user

### Testing Changes

Always run tests before declaring completion:

```bash
# Changed hawk/X/? → Run:
pytest tests/X/ -n auto -vv
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [METR/inspect-action](https://github.com/METR/inspect-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
