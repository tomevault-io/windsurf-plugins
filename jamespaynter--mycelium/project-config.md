---
trigger: always_on
description: Repository guidelines and operating instructions for humans and automation agents.
---

# agents.md

Repository guidelines and operating instructions for humans and automation agents.
Goal: enable safe changes with minimal back-and-forth by documenting system topology,
local dev workflow, configuration, and verification steps.


## Coding style & conventions

Follow **Easy Scan** principles throughout. Code should be readable at a glance.

### Code style: Easy Scan

Code should be scannable at a glance. Optimize for quick comprehension, not brevity.

#### Naming

- Descriptive names, no abbreviations (`user_resume_text` not `txt`)
- Functions say what they do (`load_job_index_from_disk` not `load`)
- Booleans read as questions (`is_ready`, `has_loaded`, `should_retry`)

#### Structure

- Code flows top-to-bottom, no jumping around
- One responsibility per function
- Related functions grouped together
- Early returns to reduce nesting

#### Section headers

Use visual breaks between logical sections in files with multiple concerns:

```python
# =============================================================================
# DATA LOADING
# =============================================================================

def load_job_index_from_disk(path: Path) -> list[dict]:
    ...


def load_job_vectors_from_disk(path: Path) -> np.ndarray:
    ...


# =============================================================================
# MATCHING LOGIC
# =============================================================================

def compute_cosine_similarity(query_vector: np.ndarray, corpus: np.ndarray) -> np.ndarray:
    ...
```

#### Comments

- Explain WHY, not WHAT
- Add context that isn't obvious from the code
- No commented-out code - delete it or use git history

#### Whitespace

- Blank lines between logical blocks
- Two blank lines before section headers
- Don't cram related code together

Python:

- 4-space indentation
- snake_case functions/variables
- PascalCase classes

TypeScript/React:

- 2-space indentation
- PascalCase components
- camelCase props

Shared code:

- Keep `shared/` import-only; do not introduce app startup side effects in `shared/`.


## Repo navigation tools

Prefer `mycelium cg` Control Graph navigation queries before grepping when you need
ownership, dependencies, blast radius, or symbol info.

Command cheat sheet:

```bash
mycelium cg components list
mycelium cg owner <path>
mycelium cg blast <path>
mycelium cg symbols find <query>
mycelium cg symbols def <symbol>
mycelium cg symbols refs <symbol>
```


## PR and commit requirements

Commits:

- use bracketed type prefixes. Recommended set (pick one per commit):
  - `[FEAT]` new functionality
  - `[FIX]` bug fix
  - `[DOCS]` documentation-only change
  - `[STYLE]` formatting/whitespace; no logic
  - `[REFACTOR]` internal code change without behavior change
  - `[PERF]` performance improvement
  - `[TEST]` add/adjust tests
  - `[BUILD]` build system or dependencies
  - `[CI]` continuous integration config/scripts
  - `[CHORE]` maintenance/housekeeping
  - `[REVERT]` revert a prior commit

PRs must include:

- services affected 
- what changed and why
- verification steps run (paste commands)
- screenshots for UI changes
- linked issue (if any)

## Project Tracking

- `ROADMAP.md`: long-term goals and future work; move items into `TODO.md` when ready to start.
- `TODO.md`: current active tasks; focus on one effort at a time; after completion, summarize and move to `CHANGELOG.md`.
- `CHANGELOG.md`: completed work with dates and brief summaries.
- `docs/history/`: dated snapshots of TODO lists; keep detailed task lists archived even after summarizing into `CHANGELOG.md`.
- Workflow: `ROADMAP → TODO → CHANGELOG`.
- When closing out a TODO batch, both: (1) summarize into `CHANGELOG.md`, and (2) save the full TODO list to `docs/history/todo-YYYY-MM-DD.md` to retain detailed context.

## Task Management System

Each TODO item links to a detailed task folder in `docs/tasks/`.

### Directory Structure

```
docs/tasks/
├── _template/              # Copy to start new task
│   ├── spec.md
│   ├── scratchpad.md
│   └── lessons-learned.md
├── active/                 # Tasks in progress
│   └── NNN-task-name/
│       ├── spec.md         # Full specification
│       ├── scratchpad.md   # Working notes
│       └── lessons-learned.md
└── archive/                # Completed tasks
```

### Task Files

**spec.md** - Full task specification:
- Status checkboxes (Scoped / In Progress / Implemented / Verified)
- Summary (one-liner)
- Model & Effort (Effort + Tier + model/reasoning selection)
- Files Changing (table: file, change type, description)
- Blast Radius (scope, risk level, rollback strategy)
- Implementation Checklist
- Verification steps
- Dependencies (blocks / blocked by)

### Effort & Model Selection

Use the Effort/Tier system for estimating and choosing a model:

- **Effort**: XS / S / M / L / XL
- **Tier**: mini / standard / pro

Model mapping:
- **Tier mini** → `gpt-5.1-codex-mini` (default reasoning)
- **Tier standard** → `gpt-5.1-codex` (default reasoning)
- **Tier pro** → `gpt-5.1-codex-max` (Reasoning: Extra high / supermax)

**scratchpad.md** - Working notes during implementation:
- Session-dated entries
- Commands run and their output
- Discoveries and findings
- Open questions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JamesPaynter/mycelium](https://github.com/JamesPaynter/mycelium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
