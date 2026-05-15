---
trigger: always_on
description: - IMPORTANT: YOU MUST ALWAYS DO THIS: When you need to send me a notification because you need input or when you have finished a task, please use terminal-notifier tool like this: terminal-notifier -title "🔔 Claude Code: request" -message "Claude needs your permission to use ...", or terminal-notifier -title "✅ Claude Code: done" -message "The task has been completed"
---

# CLAUDE.md - GeoVibes Project Guide

## Notification

- IMPORTANT: YOU MUST ALWAYS DO THIS: When you need to send me a notification because you need input or when you have finished a task, please use terminal-notifier tool like this: terminal-notifier -title "🔔 Claude Code: request" -message "Claude needs your permission to use ...", or terminal-notifier -title "✅ Claude Code: done" -message "The task has been completed"
- Always customise the message using a short summary of the input needed or the task just completed


After making changes, commit them with a useful commit message.
Use `uv run` to run with the correct env strategy
Do not leave comments unless explicitly asked to
Do not implement classes unless explicitly necessary, where possible use functions and keep things simple.
Be parsimonious with code.

## Developer Documentation (`docs/`)

**IMPORTANT**: Before implementing features, check the relevant documentation in `docs/`:

| If you're working on... | Read this doc |
|-------------------------|---------------|
| Search vs Detection mode behavior | `docs/modes-and-state.md` |
| AppState fields, mode transitions | `docs/modes-and-state.md` |
| How user clicks trigger method chains | `docs/event-flow.md` |
| Adding new event handlers | `docs/event-flow.md` |
| GeoJSON input/output formats | `docs/data-formats.md` |
| DuckDB schema, FAISS index types | `docs/data-formats.md` |
| ipyvuetify patterns, BtnToggle events | `docs/ui-widgets.md` |
| Widget hierarchy, CSS injection | `docs/ui-widgets.md` |
| MDI icons (NOT FontAwesome in side panel) | `docs/ui-widgets.md` |
| CLI tools (faiss_db.py, download_embeddings.py) | `docs/scripts.md` |
| Building new databases | `docs/scripts.md` |

### Quick Reference

- **Mode-specific behavior**: Always check `state.detection_mode` before implementing click handlers
- **BtnToggle returns index**: `v_model=0` means first option, not string value
- **Icon systems differ**: ipyvuetify uses `mdi-*`, ipywidgets uses `fa-*`
- **Query vector formula**: `2 * mean(positives) - mean(negatives)`

## Coding Preferences

- **Fail Fast**: Do not use try-except statements. Let errors surface immediately for faster debugging.
- **Time All Steps**: When implementing workflows/pipelines, add timing to each step to understand performance characteristics.
- **Memory Constraints**: Target M1 Mac with 32GB RAM. Plan batch sizes and memory usage accordingly.
- **Physical Units**: Use proper physical units (e.g., meters for distances, not degrees).
- **CLI Scripts**: Use argparse for command-line interfaces.
- **Config Files**: Use YAML/JSON config files for complex parameter sets rather than many CLI arguments.
- **Docstrings**: Docstrings are acceptable when they add value; prefer them over inline comments.
- **Integration Tests**: Prefer integration tests where possible over mocking everything.

## Ultrathink Mode

When I say **"ultrathink"**, use extended thinking/deep analysis mode. This means:
- Thoroughly analyze the problem before proposing solutions
- Consider multiple approaches and their trade-offs
- Profile or benchmark when optimizing
- Run experiments in parallel using subagents when comparing alternatives

## Brainstorm Mode

When I say **"brainstorm"** or **"don't generate any code"**, just discuss ideas without writing implementation code.

## Autonomous Debugging

When I say **"run this yourself and debug issues"** or **"fix issues until it works"**:
- Execute the code
- Identify errors
- Fix them iteratively
- Continue until the workflow completes successfully

## Clarifying Questions

**Ask clarifying questions when unsure** before proceeding with implementation. Better to clarify upfront than to implement the wrong thing.

## Bug Analysis Guidelines

When analyzing code for bugs (especially when using subagents), apply rigorous verification to avoid false positives:

### Verification Checklist (MANDATORY before reporting a bug)

1. **Trace actual execution paths** - Don't flag code based on pattern matching alone. Follow the control flow:
   - What conditions must be true to reach this code?
   - What state exists when this code executes?
   - Example: `if key in dict: dict[key]` is NOT a KeyError bug - the condition guarantees the key exists

2. **Read docstrings and comments** - Check if behavior is intentional:
   - A function named `toggle_label` that removes then re-adds is a feature, not a bug
   - Docstrings like "Toggle label assignment" indicate intentional toggle behavior

3. **Understand library-specific behaviors**:
   - **DuckDB**: `execute(sql, [a, b, c])` correctly binds list elements to `?` placeholders
   - **Regex**: Non-greedy quantifiers (`\d+?`) still produce correct results due to backtracking
   - **ipyleaflet**: Layer removal patterns with `if layer in map.layers` are idiomatic

4. **Recognize intentional design patterns**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cr458/geovibes](https://github.com/cr458/geovibes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
