---
trigger: always_on
description: This project uses the **CodeDNA** in-source communication protocol. Follow these rules on every file operation.
---

# CodeDNA v0.9 — Protocol for AI Coding Agents (Codex, OpenCode, Aider, GitHub Copilot CLI, etc.)

This project uses the **CodeDNA** in-source communication protocol. Follow these rules on every file operation.

---

## Reading files

1. Read the **module docstring** at the top of every Python file before reading any code.
2. Parse `exports:` — these are symbols you **must never rename or remove** without explicit instruction.
3. Parse `used_by:` — callers that depend on this file. **Do not follow all of them blindly.** Ask: "does this caller's domain intersect with my current task?" Only explore callers relevant to the specific change you're making.
4. Parse `related:` — files sharing the same logic without importing each other. Same filter: is it relevant to this task?
5. Parse `rules:` — hard constraints for every edit in this file; read **before writing any logic**.
6. Parse `agent:` — session history written by previous agents; read to understand *why* the current state exists.
7. For any function with a `Rules:` docstring, read and respect those before writing logic.

## Writing new files

Every new Python source file **must begin** with a CodeDNA module docstring:

```python
"""filename.py — <what it does, ≤15 words>.

exports: public_function(arg) -> return_type
used_by: consumer_file.py → consumer_function
related: other_file.py — shares same pattern/logic (no import link)
wiki:    docs/wiki/filename.md
rules:   <hard constraint agents must never violate>
agent:   <your-model-id> | <provider> | <YYYY-MM-DD> | <session_id> | <what you implemented and what you noticed>
         message: "<open hypothesis or observation for the next agent>"
"""
```

Field guide:

| Field | Required | Rule |
|---|---|---|
| First line | ✅ | `filename.py — <purpose ≤15 words>` |
| `exports:` | ✅ | Public API with return type |
| `used_by:` | ✅ | Who calls this file's exports (structural link via import) |
| `related:` | ⬜ | Files that share the same logic/pattern without importing each other (semantic link) |
| `wiki:` | ⬜ | Opt-in pointer to a deeper markdown doc under `docs/wiki/` (experimental v0.9 — see below) |
| `rules:` | ✅ | Architectural truth — specific, actionable constraints (see examples below) |
| `agent:` | ✅ | Session narrative — rolling window of last 5 entries; drop the oldest when adding a 6th |
| `message:` | ⬜ | Inter-agent channel — open hypotheses, unverified observations (v0.8) |

## Writing good `rules:`

`rules:` must be **specific and actionable** — an agent reading it should know exactly what to do or not do. Never write vague rules.

```python
# ✅ Good rules: — specific, actionable, explains WHY
rules:   get_invoices() returns ALL tenants — caller MUST filter is_suspended() before aggregating
rules:   amount is in cents not euros — divide by 100 before display
rules:   this endpoint requires admin role — auth middleware is in routes.py, not here
rules:   soft-delete via deleted_at — NEVER use DELETE, always SET deleted_at = NOW()
rules:   SQLite has no ALTER COLUMN — schema changes must recreate the table

# ❌ Bad rules: — vague, not actionable
rules:   handle errors gracefully
rules:   follow best practices
rules:   be careful with this code
rules:   none  ← when there ARE constraints but nobody wrote them
```

**When to update rules:** every time you discover a constraint, fix a bug, or notice a non-obvious behavior — add it to `rules:` immediately. This is how you communicate with the next agent.

**`rules: none` is acceptable** only when a file truly has no domain constraints (e.g., a simple utility with no side effects). If you see `rules: none` on a file with business logic, add the missing constraints.

## Writing critical functions

Every public function **must** have a `Rules:` docstring:

```python
def my_function(arg: type) -> return_type:
    """Short description.

    Rules:   What the agent MUST or MUST NOT do here.
    message: model-id | YYYY-MM-DD | observation for next agent
    """
```

## Inline annotations on complex logic

When writing or editing code blocks with non-obvious logic (complex conditions, algorithms, loops with business rules, edge cases), add a `# Rules:` or `# message:` comment **above** the block:

```python
# Rules: skip cancelled orders — status=4 means cancelled in legacy DB
active = [o for o in orders if o.status != 4]

for order in active:
    # message: exchange rate conversion uses daily rate, not real-time
    amount = order.amount * get_exchange_rate(order.currency)

    if amount > config.threshold:
        # Rules: threshold check MUST happen before tax — tax is on gross amount
        apply_tax(order)
```

**When to add inline annotations:**
- Conditional logic that encodes a business rule (`if status != 4`)
- Loops with non-obvious filtering or transformation
- Algorithm steps where the order matters
- Edge cases or workarounds with context that would be lost

**When NOT to add them:**
- Simple getters/setters
- Obvious control flow (`if not user: return None`)
- Standard library usage

## Editing files

1. **First step**: re-read `rules:`, the `agent:` history, and the `Rules:` of the function you are editing.
2. Apply all file-level constraints before writing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Larens94/codedna](https://github.com/Larens94/codedna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
