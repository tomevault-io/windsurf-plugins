---
trigger: always_on
description: **Readability first.** Favour clarity over DRY or efficiency. A little repetition is fine if it
---

# Claude Coding Preferences for ISPyPSA

## Code Style

### Guiding principle

**Readability first.** Favour clarity over DRY or efficiency. A little repetition is fine if it
makes the code easier to follow. Optimise for a reader who hasn't seen the codebase before.

### Orchestrator + helper pattern

High-level functions should read like a narrative — a sequence of descriptive verb-phrase calls
that tell the story of what the function does. Push all data manipulation into private (`_`)
helper functions.

```python
def template_network_transmission_paths(iasr_tables, scenario):
    paths = _extract_flow_paths(iasr_tables["flow_path_transfer_capability"])
    paths = _add_transfer_limits(paths, iasr_tables["interconnector_transfer_capability"])
    paths = _filter_to_scenario(paths, scenario)
    return paths
```

### Helper function guidelines

- **≤ 10 lines** of actual logic (excluding docstrings/blank lines) for any function that
  touches or transforms data.
- **Private by default** — prefix with `_` unless the function is part of the public API.
- **Descriptive names** — the name should make the orchestrator readable without needing to
  look at the helper's body. Prefer verb phrases: `_extract_*`, `_add_*`, `_filter_*`,
  `_map_*`, `_merge_*`.
- **Single responsibility** — each helper does one thing. If a helper needs an internal
  comment explaining a second step, it should probably be two helpers.
- **No hidden preconditions** — if a helper's correctness depends on a column
  value or invariant set by the caller, set it explicitly inside the helper
  instead. Preconditions that don't appear in the signature or name are easy
  to break during refactoring.

### Clarity over cleverness

- **Avoid positional access** like `iloc[:, 0]` — use named column access (e.g.
  `df["Flow Paths"]`) so the code states what it means.
- **Comment non-obvious regex** — add a concrete example of the input being matched and
  annotate each capture group.
- **Prefer explicit data over clever detection.** If the set of special cases is small and
  stable, declare them as data rather than building logic to infer them from surrounding
  context.

### Control flow

- **Keep it flat.** Prefer simple, linear control flow even if it means some repetition.
- **One level of nesting max** for `if` and `for` statements. If you find yourself writing
  nested loops or nested conditionals, extract the inner block into a helper function.
  The exception is when nesting genuinely is the simplest way to express the logic — but
  that should be rare.

### Non-defensive code

Write non-defensive code by default. Trust design decisions and caller contracts. Don't add
`None` checks, fallback logic, or `try/except` blocks unless explicitly needed or revealed
through testing. Let the code fail clearly when preconditions aren't met.

No backwards compatibility unless explicitly requested — update all call sites directly.

### Docstrings: I/O Example

Every non-trivial function should include an `I/O Example:` section in its docstring
showing a concrete input → output mapping. The goal is that a reviewer can understand
the function's behaviour from the docstring alone, without reading the body.

Conventions:

- Use a plain CSV-like table format for DataFrame inputs and outputs — no need to
  wrap in runnable `csv_str_to_df` calls, since this is illustrative, not a doctest.
- Abbreviate long column names when they would otherwise overflow the line; point at
  the relevant constants for the real names.
- Cover representative edge cases in the same example, with trailing `# comment`
  notes on the rows that demonstrate each case.
- For trivial utility functions, one-line input → output cases are enough.

```python
def _duplicate_for_both_directions(limits: pd.DataFrame) -> pd.DataFrame:
    """Mirrors each row into a forward and a reverse entry.

    I/O Example:
        limits:
            path_id  timeslice    capacity
            Q1-NQ    peak_demand  750

        returns:
            path_id  direction  timeslice    capacity
            Q1-NQ    forward    peak_demand  750
            Q1-NQ    reverse    peak_demand  750
    """
```

## Logging

Logging surfaces things a user or operator wants to know during a template/translation
run that aren't visible from the returned DataFrames. Errors that should halt the run
are `raise`d, not logged.

### Levels

- **INFO** — used for:
  - The top of a public template/translator orchestrator
    (`logging.info("Creating a template for X")`). Gives a progress trace for long runs.
  - Start and completion of long-running CLI operations (downloads, deletions, file
    generation).
  - Silently dropped or filtered data — rows that appear in the input but not in the
    output (e.g. unmatched options dropped by an inner merge).

- **WARNING** — for data integrity issues the run will tolerate but the caller might
  want to act on:
  - Per-row computations that fail and produce NaN in the output, including paths/REZs
    that were missing from the IASR tables and will receive a default downstream.
  - Empty templated tables that mean a class of components won't appear in the model.
  - User-supplied filter inputs that match nothing in the data.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-ISP/ISPyPSA](https://github.com/Open-ISP/ISPyPSA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
