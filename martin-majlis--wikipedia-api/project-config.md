---
trigger: always_on
description: validates that `ATTRIBUTES_MAPPING` in `_base_wikipedia_page.py` is
---

# AGENTS.md

Guide for AI agents (and developers) on how to install, build, and test the Wikipedia-API project.

## Before You Start

**📖 CRITICAL: Always read the design and API documentation first**

Before making any changes, read these two documents to understand the
architecture, class hierarchy, and public API:

- **`DESIGN.rst`** — internal architecture, class hierarchy, request
  lifecycle, dispatch helpers, and a step-by-step guide for adding new
  API calls.
- **`API.rst`** — public API reference: every method, property, and
  attribute available on `Wikipedia`, `AsyncWikipedia`, `WikipediaPage`,
  `AsyncWikipediaPage`, `WikipediaPageSection`, and the CLI.

Skipping this step risks duplicating existing logic, violating
established conventions, or breaking the sync/async symmetry.

## Sync / Async Symmetry

**🚨 CRITICAL: The sync and async APIs MUST stay in perfect symmetry.**

`WikipediaPage` and `AsyncWikipediaPage` are parallel classes.
Every public attribute or method on one **must** have the same kind of
interface on the other:

| If `WikipediaPage` has …     | then `AsyncWikipediaPage` MUST have …                                            |
| ---------------------------- | -------------------------------------------------------------------------------- |
| `@property foo`              | awaitable property `await page.foo` (explicit `@property` returning a coroutine) |
| plain method `foo()`         | coroutine method `await page.foo()`                                              |
| plain `@property` (no fetch) | plain `@property` (no fetch)                                                     |

**Never** convert a property to a method (or vice versa) in one class
without making the matching change in the other. Violations break
the documented API contract and confuse callers who switch between
the two clients.

Examples of correct symmetry currently in place:

- `page.summary`, `page.text`, `page.langlinks`, `page.links`,
  `page.backlinks`, `page.categories`, `page.categorymembers` —
  `@property` in sync; explicit `@property` returning a coroutine in async.
- `page.pageid`, `page.fullurl`, `page.displaytitle`, and all other
  info attributes — same pattern: `@property` in sync, awaitable
  `@property` in async.
- `page.exists()` — plain method in sync; coroutine method
  `await page.exists()` in async (both use call syntax `()`).
- `page.sections`, `page.title`, `page.ns`, `page.language`,
  `page.variant` — plain `@property` in both (no awaiting needed).

## Typing Standards

**🧠 Prefer explicit type annotations and minimize `Any`.**

When writing or updating Python code in this repository:

- Use inline type annotations directly on variables, attributes, parameters,
  and return values (e.g. `value: dict[str, int] = {}`), instead of legacy
  `# type:` comments.
- Avoid `Any` whenever a more specific type can be expressed.
- Use `Any` only when it is absolutely necessary (for example, dynamic external
  payloads or framework boundaries where precise typing is not practical).
- If `Any` is required, keep its scope as small as possible and prefer typed
  wrappers/conversions at the boundary.
- Validate typing-related changes by running `make run-pre-commit` before
  submitting.

## Docstring Standards

**📝 Write descriptive docstrings with consistent structure.**

All functions, methods, classes, and modules must have descriptive docstrings that follow this structure:

### Required Docstring Format

```python
def example_function(param1: str, param2: int) -> bool:
    """One-line summary of the function's purpose.

    Detailed description of the function's behavior, including important
    implementation details, usage patterns, or context.

    Args:
        param1: Description of the first parameter, including expected format
            and constraints.
        param2: Description of the second parameter, including valid ranges
            or special values.

    Returns:
        Description of the return value, including its type and meaning.
        Include possible return values and their significance.

    Raises:
        ExceptionType: Description of when this exception is raised,
            including the conditions that trigger it.
        AnotherException: Description of when this exception occurs.

    Invariants:
        - Any conditions that remain true before and after execution
        - State guarantees that the function maintains
        - Thread safety considerations if applicable
    """
```

### Docstring Requirements

1. **One-line summary**: Must be a complete sentence ending with a period
   that concisely describes what the function does.

2. **Detailed description**: Expand on the summary with implementation details,
   usage examples, or important context.

3. **Parameters section (`Args`)**: Document all parameters with:
   - Parameter name (matching the function signature)
   - Description including expected format, constraints, and valid values
   - Use proper indentation and formatting

4. **Return values section (`Returns`)**: Document the return value with:
   - Type information (if not obvious from type hints)
   - Meaning and significance of different return values
   - Special cases or conditions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martin-majlis/Wikipedia-API](https://github.com/martin-majlis/Wikipedia-API) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
