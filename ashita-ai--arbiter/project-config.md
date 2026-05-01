---
trigger: always_on
description: Production-grade LLM evaluation framework developer
---


# Arbiter Agent Guide

**Purpose**: Quick reference for working on Arbiter

**Arbiter**: Production-grade LLM evaluation framework (v0.1.2)
**Stack**: Python 3.11+, PydanticAI, provider-agnostic (OpenAI/Anthropic/Google/Groq)
**Coverage**: 96% test coverage, strict mypy, comprehensive examples
**Pricing**: LiteLLM bundled database (consistent with Conduit)

**Design Philosophy**: Simplicity wins, use good defaults, YAML config where needed, no hardcoded assumptions.

---

## Quick Start (First Session Commands)

**New to this repo? Run these 5 commands first:**

```bash
# 1. Verify you're on a feature branch (NEVER work on main)
git status && git branch

# 2. Run all quality checks
make all

# 3. Run specific evaluator test to verify environment
pytest tests/unit/test_semantic.py -v

# 4. Check for any TODOs or placeholders (should be NONE)
grep -r "TODO\|FIXME\|NotImplementedError" arbiter/ || echo "✅ No placeholders found"

# 5. Verify coverage is >80%
make test-cov | tail -1
```

---

## Boundaries

### Always Do (No Permission Needed)

- Run tests: `make test`, `pytest tests/`, `pytest -v`
- Format code: `make format` (runs black)
- Lint code: `make lint` (runs ruff)
- Type check: `make type-check` (runs mypy in strict mode)
- Add unit tests for new evaluators in `tests/unit/`
- Update docstrings when changing function signatures
- Add examples to `examples/` for new user-facing features
- Export new evaluators in `__init__.py` files
- Run `make all` before committing (format + lint + type-check + test)

### Ask First

**Core Architecture** (Why: Breaks all evaluators):
- Add new evaluators to `arbiter/evaluators/` - Must follow template pattern
- Modify core API in `arbiter/api.py` (evaluate, compare functions) - Breaking change for all users
- Change template method pattern in `BasePydanticEvaluator` - All evaluators inherit from this
- Modify middleware pipeline in `arbiter/core/middleware.py` - Affects all evaluations
- Change LLM client abstraction in `arbiter/core/llm_client.py` - Provider-agnostic guarantee at risk

**Dependencies & Config** (Why: Security and maintenance burden):
- Add/update dependencies in `pyproject.toml` - Increases attack surface
- Change public API examples in `README.md` - User-facing documentation
- Add new storage backends in `arbiter/storage/` - Data persistence implications

**Monitoring & Observability** (Why: Production debugging):
- Modify interaction tracking in `arbiter/core/monitoring.py` - Breaks observability

### Never Touch

**Security (CRITICAL)**:
- NEVER EVER COMMIT CREDENTIALS TO GITHUB
- No API keys, tokens, passwords, secrets in ANY file
- No credentials in code, documentation, examples, tests, or configuration files
- Use environment variables (.env files in .gitignore) ONLY
- This is non-negotiable with serious security consequences

**Other Prohibitions**:
- `.env` files or API keys (use environment variables)
- Production deployment configurations
- Git history manipulation (no force push, interactive rebase on shared branches)
- User's `~/.claude/` configuration files
- Any files outside the `arbiter/` repository
- Test files to make them pass (fix the code, not the tests)
- Type checking configuration to reduce strictness
- Coverage thresholds (must maintain >80%)

**Detection Commands** (Run before committing):
```bash
# Check for security violations
grep -r "API_KEY\|SECRET\|PASSWORD" arbiter/ tests/ examples/ && echo "🚨 CREDENTIALS FOUND" || echo "✅ No credentials"

# Check for code quality violations
grep -r "TODO\|FIXME" arbiter/ && echo "🚨 TODO comments found" || echo "✅ No TODOs"

# Check for incomplete features
grep -r "NotImplementedError\|pass  # TODO" arbiter/ && echo "🚨 Placeholder code found" || echo "✅ No placeholders"

# Verify on feature branch
git branch --show-current | grep -E "^(main|master)$" && echo "🚨 ON MAIN BRANCH - CREATE FEATURE BRANCH" || echo "✅ On feature branch"

# Verify coverage >80%
make test 2>&1 | grep "TOTAL" | awk '{if ($NF+0 < 80) print "🚨 COVERAGE " $NF " < 80%"; else print "✅ Coverage " $NF}'
```

---

## Communication Preferences

Don't flatter me. I know what [AI sycophancy](https://www.seangoedecke.com/ai-sycophancy/) is and I don't want your praise. Be concise and direct. Don't use emdashes ever.

---

## Session Analysis & Continuous Improvement

**When to Analyze** (Multiple Triggers):
- During active sessions: After completing major tasks or every 30-60 minutes
- When failures occur: Immediately analyze and update rules
- Session end: Review entire session for patterns before closing
- User corrections: Any time user points out a mistake

**Identify Failures**:
- Framework violations (boundaries crossed, rules ignored)
- Repeated patterns (same mistake multiple times)
- Rules that didn't prevent failures
- User corrections (what needed fixing)

**Analyze Each Failure**:
- What rule should have prevented this?
- Why didn't it work? (too vague, wrong priority, missing detection pattern)
- What would have caught this earlier?

**Update AGENTS.md** (In Real-Time):
- Add new rules or strengthen existing rules immediately
- Add detection patterns (git commands, test patterns, code patterns)
- Include examples of violations and corrections
- Update priority if rule was underweighted

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ashita-ai/arbiter](https://github.com/ashita-ai/arbiter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
