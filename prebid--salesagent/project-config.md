---
trigger: always_on
description: This guide helps you work effectively with the Prebid Sales Agent codebase maintained under Prebid.org. Key principles:
---

# Prebid Sales Agent - Development Guide

## 🤖 For Claude (AI Assistant)

This guide helps you work effectively with the Prebid Sales Agent codebase maintained under Prebid.org. Key principles:

### Working with This Codebase
1. **Always read before writing** - Use Read/Glob to understand existing patterns
2. **Test your changes** - Run `make quality` before committing
3. **Follow the patterns** - 7 critical patterns below are non-negotiable
4. **When stuck** - Check `/docs` for detailed explanations
5. **Pre-commit hooks are your friend** - They catch most issues automatically
6. **Name your PRs correctly** - they need to pass .github/workflows/pr-title-check.yml

### Common Task Patterns
- **Adding a new AdCP tool**: Extend library schema → Add `_impl()` function → Add MCP wrapper → Add A2A raw function → Add tests
- **Fixing a route issue**: Check for conflicts with `grep -r "@.*route.*your/path"` → Use `url_for()` in Python, `scriptRoot` in JavaScript
- **Modifying schemas**: Verify against AdCP spec → Update Pydantic model → Run `pytest tests/unit/test_adcp_contract.py`
- **Database changes**: Use SQLAlchemy 2.0 `select()` → Use `JSONType` for JSON → Create migration with `alembic revision`

### Key Files to Know
- `src/core/main.py` - MCP tools and `_impl()` functions
- `src/core/tools.py` - A2A raw functions
- `src/core/schemas.py` - Pydantic models (AdCP-compliant)
- `src/adapters/base.py` - Adapter interface
- `src/adapters/gam/` - GAM implementation
- `tests/unit/test_adcp_contract.py` - Schema compliance tests

### DRY (Don't Repeat Yourself) — Non-Negotiable Invariant

**DRY is not "premature optimization." It is not "refactoring beyond what was asked." It is a correctness requirement, equivalent to type safety or test integrity.**

- If you write a block of logic that is structurally similar to an existing block (same pattern, different variables), you **MUST** extract a shared helper function
- If you are asked to refactor duplicated code, that is a **bug fix**, not an "improvement"
- **NEVER** cite "avoid over-engineering" or "keep it simple" to justify leaving duplicated logic in place
- Duplicated code is a defect. It means the next person who fixes a bug in one copy will miss the other copy. This is not theoretical — it has caused real bugs in this codebase
- **Enforced by:** `check_code_duplication.py` pre-commit hook (pylint R0801, ratcheting baseline in `.duplication-baseline`)

**How to apply DRY correctly:**
```python
# WRONG: copy-paste with variable substitution
formats = [f for f in formats if {fid.id for fid in f.output_format_ids} & requested_output_ids]
formats = [f for f in formats if {fid.id for fid in f.input_format_ids} & requested_input_ids]

# CORRECT: extract the shared pattern
def filter_by_format_ids(formats, requested, attr):
    if not requested:
        return formats
    ids = {fmt.id for fmt in requested}
    return [f for f in formats if {fid.id for fid in getattr(f, attr)} & ids]

formats = filter_by_format_ids(formats, req.output_format_ids, "output_format_ids")
formats = filter_by_format_ids(formats, req.input_format_ids, "input_format_ids")
```

**What DRY is NOT:**
- It is not an excuse to create deep abstraction hierarchies for one-time code
- It is not about collapsing two genuinely different operations that happen to look similar today
- It applies when the same **logical operation** is repeated with only parameter substitution

### What to Avoid
- ❌ Don't use `session.query()` (use `select()` + `scalars()`)
- ❌ Don't duplicate library schemas (extend with inheritance)
- ❌ Don't hardcode URLs in JavaScript (use `scriptRoot`)
- ❌ Don't bypass pre-commit hooks without good reason
- ❌ Don't skip tests to make CI pass (fix the underlying issue)
- ❌ Don't leave duplicated logic — extract shared helpers (DRY invariant above)

### Commit Messages & PR Titles
**Use Conventional Commits format** - release-please uses this to generate changelogs.

PR titles should use one of these prefixes:
- `feat: Add new feature` - New functionality (appears in "Features" section)
- `fix: Fix bug description` - Bug fixes (appears in "Bug Fixes" section)
- `docs: Update documentation` - Documentation changes
- `refactor: Restructure code` - Code refactoring (appears in "Code Refactoring" section)
- `perf: Improve performance` - Performance improvements
- `chore: Update dependencies` - Maintenance tasks (hidden from changelog)

**Without a prefix, commits won't appear in release notes!** The code will still be released, but the change won't be documented in the changelog.

### Structural Guards (Automated Architecture Enforcement)
AST-scanning tests enforce architecture invariants on every `make quality` run. New violations fail the build immediately. See [docs/development/structural-guards.md](docs/development/structural-guards.md) for full details.

| Guard | Enforces | Test File |
|-------|----------|-----------|
| No ToolError in _impl | `_impl` raises AdCPError, never ToolError | `test_no_toolerror_in_impl.py` |
| Transport-agnostic _impl | `_impl` has zero transport imports | `test_transport_agnostic_impl.py` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prebid/salesagent](https://github.com/prebid/salesagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
