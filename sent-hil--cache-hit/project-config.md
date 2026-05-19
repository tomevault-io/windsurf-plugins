---
trigger: always_on
description: This document contains project-specific guidelines and best practices for Claude Code when working on this codebase.
---

# CacheHit - Claude Code Guidelines

This document contains project-specific guidelines and best practices for Claude Code when working on this codebase.

## Rules Summary

### 1. Meta - Maintaining This Document

See all rules in section **## META - MAINTAINING THIS DOCUMENT**

1. When adding new rules to detailed sections below, always update this summary section with a corresponding one-sentence summary
2. Each rule in this summary must reference its corresponding detailed section
3. Follow the writing guidelines when adding new rules

### 2. Code Organization

See all rules in section **## CODE ORGANIZATION**

1. Always place imports at the top of the file, never inside functions
2. [Add your rules here]

### 3. Testing

See all rules in section **## TESTING**

1. Always use `patch.object` rather than `patch` for mocking
2. Never use magic numbers or string literals in tests - extract constants
3. Always check for existing test helpers before creating new ones
4. [Add your rules here]

---

## META - MAINTAINING THIS DOCUMENT

### Keeping the Summary Section Up to Date

**Rule**: Whenever you add, modify, or remove rules in the detailed sections below, you MUST update the "Rules Summary" section at the top of this document.

**Process**:

1. Add the new rule to the appropriate detailed section below
2. Add a corresponding one-sentence summary to the Rules Summary section
3. Ensure the summary references the detailed section using the format: "See all rules in section **## SECTION NAME**"
4. If creating a new topic, add both a new numbered topic in the summary AND a new detailed section below

**Example**:
If you add a new rule about async patterns in the detailed "ASYNC PATTERNS" section, you must add:

- A new topic in Rules Summary: "4. Async Patterns - See all rules in section **## ASYNC PATTERNS**"
- A one-sentence summary under that topic

### Writing Effective Guidelines

When adding new rules to this document, follow these principles:

**Core Principles (Always Apply):**

1. **Use absolute directives** - Start with "NEVER" or "ALWAYS" for non-negotiable rules
2. **Lead with why** - Explain the problem/rationale before showing the solution (1-3 bullets max)
3. **Be concrete** - Include actual commands/code for project-specific patterns
4. **Minimize examples** - One clear point per code block
5. **Bullets over paragraphs** - Keep explanations concise
6. **Action before theory** - Put immediate takeaways first

**Optional Enhancements (Use Strategically):**

- **❌/✅ examples**: Only when the antipattern is subtle or common
- **"Why" or "Rationale" section**: Keep to 1-3 bullets explaining the underlying reason
- **"Warning Signs" section**: Only for gradual/easy-to-miss violations
- **"General Principle"**: Only when the abstraction is non-obvious
- **Decision trees**: Only for 3+ factor decisions with multiple considerations

**Anti-Bloat Rules:**

- ❌ Don't add "Warning Signs" to obvious rules (e.g., "imports at top")
- ❌ Don't show bad examples for trivial mistakes
- ❌ Don't create decision trees for simple binary choices
- ❌ Don't add "General Principle" when the section title already generalizes
- ❌ Don't write paragraphs explaining what bullets can convey
- ❌ Don't write long "Why" explanations - 1-3 bullets maximum

---

## TYPE CHECKING WITH ty

We use ty for gradual type checking adoption.

### General Principles

1. **Minimize Logic Changes**: Type checking should NOT change runtime behavior. When adding type hints:

   - Keep original logic intact
   - Use `# type: ignore` comments when necessary
   - Only fix actual type errors, not "improvements"

2. **Test After Type Changes**: Always run tests after adding type hints:
   ```bash
   uv run ty check  # Type checking
   uv run pytest tests/  # Full test suite
   ```

## CODE ORGANIZATION

### Import Placement

**Always place imports at the top of the file, never inside functions.**

```python
# ✅ CORRECT
from datetime import datetime
import pytest

def test_something():
    # test code here
    pass

# ❌ INCORRECT
def test_something():
    from datetime import datetime  # Never inside functions
    pass
```

**Rationale**: Top-level imports make dependencies clear, allow static analysis tools to work correctly, and avoid repeated import overhead.

### Understand lru_cache as Singleton Pattern

**`@lru_cache` on zero-argument functions creates singletons that are populated at startup.**

**Pattern**:

```python
# In another file (e.g., deck_router.py)
@lru_cache
def get_deck_cache() -> Dict[str, Deck]:
    return {}  # Returns same dict instance every call

# In main.py startup
deck_cache = deck_router.get_deck_cache()  # Get the singleton
deck_cache[deck.id] = deck  # Populate it

# In endpoint (first file)
@router.get("/decks")
async def list_decks(cache: Dict[str, Deck] = Depends(get_deck_cache)):
    # Gets the SAME dict that was populated at startup
    return {"decks": list(cache.values())}
```

**What this is NOT**:

- ❌ NOT a cache in the "memoization" sense
- ❌ NOT creating a new instance per request
- ✅ IS a way to create a shared singleton that lives for the app lifetime


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sent-hil/cache-hit](https://github.com/sent-hil/cache-hit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
