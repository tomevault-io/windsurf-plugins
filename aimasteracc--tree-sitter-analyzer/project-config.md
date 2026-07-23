---
trigger: always_on
description: - Do what has been asked; nothing more, nothing less
---

# Ruflo — Claude Code Configuration

## Rules

- Do what has been asked; nothing more, nothing less
- NEVER create files unless absolutely necessary — prefer editing existing files
- NEVER create documentation files unless explicitly requested
- NEVER save working files or tests to root — use `/src`, `/tests`, `/docs`, `/config`, `/scripts`
- ALWAYS read a file before editing it
- NEVER commit secrets, credentials, or .env files
- Keep files under 500 lines
- Validate input at system boundaries

## Test Quality Rules — LOCKED (2026-06-28)

These rules exist because the test suite grew to 1,093 files / 21,000 tests with
systematic duplication, coverage-chasing files, and weak assertions that let real
bugs through (ref: 2026-06-08 TOON incident, §11). **All 5 rules are enforced at
pre-commit and CI. Violations block merge.**

### T-1: No new test files for existing plugins (BLOCKER)

**🔒 LOCKED:** When adding tests for an existing language plugin or MCP tool,
ADD to the existing `test_{plugin}.py` file. Do NOT create new files.

Banned name patterns — pre-commit will reject any new file matching:
- `*_comprehensive*.py`
- `*_edge_cases*.py`
- `*_coverage_boost*.py`
- `*_coverage*.py` (unless it is a coverage measurement tool, not a test)
- `*_extended*.py`
- `*_optimized*.py`

**Why:** java grew to 38 test files for 498 lines of code. The root cause was
"not sure if I covered it → add another file" without a deletion culture.

**Legitimate exception:** A new file is allowed ONLY when it tests a genuinely
new subsystem (a new subpackage, a new MCP tool, a new CLI command) that has no
existing test file. The file name must match the module it tests exactly.

### T-2: No weak assertions — ratchet-enforced (BLOCKER)

See §"Exact assertions only" below. The `weak-assertion-ratchet` pre-commit hook
enforces this mechanically. Current baseline: 171 (only decreases, never increases).

### T-3: Coverage rate is a by-product, never a goal (BLOCKER)

**🔒 LOCKED:** Writing a test whose primary purpose is to increase a coverage
percentage is prohibited. Every test must assert a concrete behavioral fact.

Symptom of violation: the test has no `assert` that would fail if the function
returned the wrong value — only asserts that it "ran without error" or returned
"something non-None".

**Why:** `_coverage_boost_` files were created exactly this way (commit a4d4b57e).
They add lines to the coverage report without adding any bug-detection power.

### T-4: One behavior per test function

Each `def test_*` function must test exactly one behavior. Mixed-behavior tests
hide failures and make root-cause analysis slow.

```python
# WRONG: tests import extraction AND class extraction in one function
def test_extract():
    imports = plugin.extract_imports(tree, code)
    assert len(imports) == 3
    classes = plugin.extract_classes(tree, code)
    assert len(classes) == 2

# RIGHT: split by behavior
def test_extract_imports_count():
    imports = plugin.extract_imports(tree, code)
    assert len(imports) == 3

def test_extract_classes_count():
    classes = plugin.extract_classes(tree, code)
    assert len(classes) == 2
```

### T-5: Regression tests require an issue reference

Any test written to prevent a specific past bug from recurring MUST include a
comment citing the issue number or incident date.

```python
def test_annotation_not_duplicated_in_ifdef():
    # Issue #534: @Cacheable extracted twice when defined in both #ifdef branches
    result = extractor.extract_functions(tree, code)
    assert len([f for f in result if f.name == "SQUARE"]) == 1
```

Tests without an issue reference are assumed to be behavioral tests (T-4),
not regression tests. Do not add "regression" to the name without the reference.

---

### Exact assertions only — no `>=` / approximate test assertions

**🔒 LOCKED BY USER (2026-06-10):** 「测试拒绝大于等于这样的约等不严谨的测试」。
Count/measurement assertions in tests MUST pin the **exact** expected value
(`== 11`), never a loose bound (`>= 10`, `> 0`, `<= 100`) that lets drift pass
silently. If an upstream change (e.g. a grammar-version bump) shifts the
number, the test SHOULD go red and force a conscious re-pin with the new
measured value — an approximate green is a false green. Reviewer suggestions
to "relax to a lower bound for resilience" are REJECTED under this rule.
Legitimate exceptions are rare and only where the value is genuinely
nondeterministic (timing, memory) — and then the test should assert a
documented invariant, not a hand-waved bound on a deterministic count.

## Deliberate design decisions — do NOT "fix" these

These look like inconsistencies in a dogfood pass, but they are intentional and reflect the project's design priorities. Reverting them costs real value. **If a dogfood agent proposes any of the items below as a "finding", REJECT the finding and link the agent back to this section.**

### 1. MCP defaults to TOON; CLI defaults to JSON — LOCKED

- **Why**: TOON is 50-70% more token-efficient than JSON. MCP callers are LLM agents — token cost is real money. CLI callers are humans / shells — JSON is human-readable and pipes into `jq`.
- **Symptom that looks like a bug**: `MCP execute()` returns `{format: "toon", toon_content: "..."}` while CLI returns a parsed dict.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aimasteracc/tree-sitter-analyzer](https://github.com/aimasteracc/tree-sitter-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
