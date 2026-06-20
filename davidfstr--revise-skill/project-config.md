---
trigger: always_on
description: Perform common code quality revisions on AI-drafted code. Use when reviewing recently drafted code for readability and maintainability improvements. Use when asked to learn new revise patterns from a commit or diff.
---


# Revise

Perform common code quality improvement revisions on code that was recently drafted by AI.

## Why this skill exists

AI-written code is usually drafted in a way that is easy for an AI to *write* at the time but not necessarily easy to *read* later (by human or AI). For example:

- AIs frequently use local imports in Python code -- contrary to standard Python convention, but easy to write in isolation without considering the rest of the file.
- AIs tend to write low-level functions before high-level functions -- easy to write a high-level function when you already know the names of the low-level functions it will call, but less efficient for readers who always want to start reading at high-level entrypoints.

The goal of revising code is to optimize its organization for *readers* of the code.

## Input

`$ARGUMENTS` specifies what to review. Supported forms:

- **`uncommitted`** -- Review uncommitted changes, as reported by `git diff` and `git diff --staged`.
- **`last-commit`** -- Review the most recent git commit (`git diff HEAD~1`).
- **A list of files** -- Review the specified files in their entirety (for recently created or heavily modified files).
- **A list of functions or classes** -- Review the specified symbols in context.

If no argument is given, default to reviewing uncommitted changes.
If there are no uncommitted changes, default to reviewing the last commit.

## Procedure

1. **Identify files to review.** Collect all files from the diff or argument list that need review.

2. **Order files for review.** Review higher-level files first, before lower-level implementation details:
   1. **Documentation files** (README, RELEASE_NOTES, doc/**) -- Often explain new features/systems at the highest level, for users or developers.
   2. **Test files** (test/**) -- Define acceptance criteria for a feature and exercise new API surfaces.
   3. **Product code files** -- Among these, try to identify which are higher-level entrypoints and review those first.

3. **Pass 1 -- Immediate fixes.** Review each file in order. For each code smell found:
   - **Apply immediately** if the fix is small and localized (rename, inline, add comment, extract helper, etc.).
   - **Defer** if the fix would cause large diff traffic or is more like a new feature than a small revision. Record deferred fixes per file. Common reasons to defer:
     - **High diff traffic:** Reordering top-level sections/functions, changing indentation style. These touch many lines, easily conflict with smaller changes happening in parallel.
     - **Non-trivial new work:** Changing build systems, configuring new tooling (linters, typecheckers), adding assets. These are features, not revisions.

4. **Review point 1.** Present the immediate fixes for review before proceeding. Also present the list of unapplied deferred fixes for confirmation.

5. **Pass 2 -- Deferred fixes.** Apply the recorded deferred fixes, one file at a time.

6. **Review point 2.** Present the deferred fixes for review.

## Code smells

### Organization (file-level)

- **[Local imports in non-entrypoint modules](patterns/local_imports.md)** -- Project imports placed inside functions instead of at the top of the file.
  ```python
  def _open_window(title, diff_bytes, api):
      from gvc.renderer import render  # should be at top of file
  ```

- **[Functions ordered bottom-up](patterns/functions_ordered_bottom_up.md)** -- Callees defined above callers; reader hits details before the big picture. **Always check this whenever new functions have been added to an existing file** — the cursor-position drop-in is almost never the right place in the call tree.
  ```python
  def _build_title(args): ...    # helper defined first
  def main(): ...                # entry point buried below
  ```

- **[Unnecessarily public functions/methods](patterns/privatize_by_default.md)** -- Default functions/methods to `_`-prefixed private; make public only when externally needed. Small public APIs focus attention; private methods are easier to reason about and refactor.
  ```python
  class TestClient:
      def call(self, method: str): ...          # only used by ping/list_windows below
      def ping(self): return self.call("ping")  # -> _call
  ```

- **[Many functions with no grouping section](patterns/missing_sections.md)** -- A file has many top-level definitions (>7-10) with no visual grouping.

- **[Sections grouped by kind instead of feature/concern](patterns/sections_by_kind.md)** -- Headings like "Constants", "Data Model", "Public API" lump items by what they are, not what feature they serve.
  ```python
  # --- Constants ---       # serves 3 different features
  # --- Public API ---      # public vs. private is rarely useful
  ```

- **[Symmetric operations split across modules](patterns/symmetric_operations_colocated.md)** -- Send/receive, read/write, encode/decode pairs live in different modules instead of adjacent in the same one.
  ```python
  # _ipc.py has try_send(), but _gui.py has the receive logic inline
  ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidfstr/revise-skill](https://github.com/davidfstr/revise-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
