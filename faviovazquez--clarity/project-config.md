---
trigger: always_on
description: >
---


# clarity

A knowledge mapping skill that makes the gap between what your agent built and
what you understand into a first-class, persistent artifact. Where `learnship`
manages the agent's memory, `clarity` manages yours — and helps you build it.

**Core principle:** "The code works" and "I understand this code" are not the same
statement. This skill makes the difference visible before it becomes a problem,
and helps fix it when it does.

Six actions:
- **map** — classify every module by how well you understand it
- **debt** — measure comprehension from a session, the full codebase, or a description
- **review** — autonomous agent scan of the codebase, no questions required
- **explain** — agent teaches you a specific module interactively
- **handoff** — generate or consume a context transfer document
- **status** — five-line project knowledge snapshot

Based on research cited in [references/cognitive-debt.md](references/cognitive-debt.md)
and [references/feynman-technique.md](references/feynman-technique.md).

---

## Actions

### `map` — Knowledge map

**Trigger:**
```
# Claude Code
/clarity map
/clarity map --quick
/clarity map --module <module>
/clarity map --explain

# Windsurf, Cursor, and others
@clarity map
@clarity map --quick
@clarity map --module <module>
@clarity map --explain
```

**What to do:**

1. **Scan the codebase.** Read the directory structure and identify top-level modules,
   components, or meaningful areas. If `AGENTS.md` exists, read it to understand
   decisions already documented. If `CLARITY_MAP.md` already exists, read it before
   updating — preserve prior scores and dates for unchanged modules.

2. **For each module or area** (skip unchanged modules if `--quick` is set):

   If `--explain` is set: before asking anything, give a 2-3 sentence summary of what
   the agent sees in this module — its apparent purpose, key patterns, and any notable
   complexity. This gives the user something to react to rather than starting from a
   blank page. Then proceed to Question A.

   Otherwise proceed directly to Question A.

   **Question A (what):** "Walk me through what `<module>` does — as if explaining it
   to someone joining the project today."

   **Question B (why):** "What was the key decision that shaped how this was built?
   Why that approach and not the obvious alternative?"

   Wait for each answer before moving to the next. Do not batch questions.

3. **Classify** each module into one of three zones based on the answers:
   - **Green (Understood):** The user explained what it does and why it is built that
     way, with no significant gaps or "I think" hedges.
   - **Yellow (Partial):** The user understood the what but was vague on the why, or
     could not articulate the key decision. Also assign Yellow when the user's
     explanation matches the code's surface behavior but misses the mechanism.
   - **Red (Risk zone):** The user could not explain what the module does, said
     "I'm not sure," said "the AI wrote it," or gave a description that contradicts
     what the code actually does.

4. **Write `CLARITY_MAP.md`** to the project root using the template at
   [templates/CLARITY_MAP.md](templates/CLARITY_MAP.md). Preserve all prior entries.
   Update only modules that were evaluated in this session.

5. **Generate or update `clarity-graph.html`** in the project root using the template
   at [templates/clarity-graph.html](templates/clarity-graph.html).
   Inject the current module data as JSON into the `CLARITY_DATA` variable. Each
   module entry must include:
   - `id` (string, module name slug)
   - `label` (string, display name)
   - `zone` ("green" | "yellow" | "red")
   - `lines` (approximate line count — use `wc -l` if available, estimate otherwise)
   - `last_evaluated` (ISO date string)
   - `what` (one sentence from the user's answer, or empty string)
   - `why` (one sentence on the key decision, or empty string)
   - `dependencies` (array of module id strings this module imports from or calls)

6. **Report** the full classification summary, highlighting any new Red zones.
   Tell the user to open `clarity-graph.html` for the visual map.

**Flags:**
- `--quick`: Re-evaluate only modules that are new, Red, or Yellow. Skip unchanged Green.
- `--module <module>`: Evaluate only the named module. Update its entry and regenerate graph.
- `--explain`: Before asking each question, give the agent's reading of the module so
  the user has a starting point. Useful when entering unfamiliar territory.

**Never** pre-classify modules without asking. The point is to surface what the user
actually knows, not what the agent infers.

---

### `debt` — Cognitive debt measurement

**Trigger:**
```
# Claude Code
/clarity debt
/clarity debt --scan
/clarity debt --session
/clarity debt --history
/clarity debt --threshold <0-100>

# Windsurf, Cursor, and others
@clarity debt
@clarity debt --scan
@clarity debt --session
@clarity debt --history
@clarity debt --threshold <0-100>
```

**How the source is chosen:**

`debt` has three modes. Choose automatically based on context unless a flag forces one:

1. **Diff mode** (default when git is available and there are recent commits):
   Run `git diff HEAD~1 HEAD`. If the diff is non-trivial (more than ~20 lines of
   meaningful logic), use it as the question source.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FavioVazquez/clarity](https://github.com/FavioVazquez/clarity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
