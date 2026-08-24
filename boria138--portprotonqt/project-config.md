---
trigger: always_on
description: **Project:** PortProtonQt — GUI for PortProton, Steam, Epic Games Store
---

# PortProtonQt — AI Agent Guidelines

**Project:** PortProtonQt — GUI for PortProton, Steam, Epic Games Store
**Language:** Python 3.10+
**Platform:** Linux (POSIX)
**License:** GPL-3.0
**Build:** Meson + uv

---

**Summary:** AI agents MUST behave as conservative patching assistants, prioritizing minimal changes and existing patterns over architectural refactoring or code cleanup. AI MUST NOT perform architectural improvements unless explicitly requested.

**Scope:** These guidelines apply exclusively to AI-generated code. Documentation updates and human maintainers are exempt from line limits and these constraints.

## License and Attribution

- All code MUST be compatible with GPL-3.0.
- AI agents MUST NOT add `Signed-off-by` tags.

When AI contributes to the project, add an `Assisted-by` tag:

`Assisted-by: AGENT_NAME:MODEL_VERSION`

Where:
- `AGENT_NAME` is the AI agent or framework name.
- `MODEL_VERSION` is the specific model version.

Example:

`Assisted-by: Claude:claude-3-opus`

---

## Core Principles

| Principle | Required | Forbidden |
|-----------|----------|-----------|
| KISS | New/rewritten functions ≤50 lines | Deep nesting |
| YAGNI | Concrete code | Future abstractions |
| DRY | Extract methods (if directly related to task) | Copy-paste |
| SRP | 1 task per method | God functions |
| Linux | POSIX paths, shebangs | Windows-specific code |
| Minimal changes | Modify relevant section only | Rewrite entire files |

**Priority order (highest to lowest):**

1. Minimal changes (overrides DRY, SRP)
2. Security (no shell=True, no hardcoded credentials)
3. Linux compatibility
4. KISS / YAGNI
5. DRY / SRP

**When principles conflict:**

- Prefer minimal diff over extracting methods
- Prefer existing patterns over "correct" refactoring
- Prefer small targeted fix over comprehensive cleanup
- Never break security for code quality
- Remove duplicates only if directly related to current task
- Do not split untouched legacy functions only to satisfy line limits
- Do not rewrite untouched legacy files or blocks only to satisfy metrics

---

## Code Metrics

| Check | Limit |
|-------|-------|
| Functions | ≤50 lines, ≤4 params (required for new/fully rewritten functions) |
| Files | ≤800 lines (required for new/fully rewritten files) |
| Nesting | ≤4 levels (required in new/modified code) |
| Comments | English, 1-2 lines max |
| Indentation | 4 spaces (no tabs) |
| Whitespace | No trailing spaces |
| Blank lines | No excessive blank lines |
| EOF | Exactly one newline |
| Commits | English, ≤72 chars |

---

## Forbidden Patterns

```python
# NEVER: 6+ parameters
def process_game(user, ctx, log, val, map, cache): ...

# NEVER: Deep nesting
if c1:
    if c2:
        if c3:
            if c4: ...

# NEVER: print statements
print(f"Game {name} started")

# NEVER: shell=True
subprocess.run(cmd, shell=True)

# NEVER: Hardcoded credentials
API_KEY = "sk-abc123"

# NEVER: Path traversal
file_path = f"/data/{user_filename}"

# NEVER: Hardcoded styles or constants (colors, sizes, etc.)
shadow.setBlurRadius(20)
shadow.setColor(QColor(0, 0, 0, 150))
label.setStyleSheet("color: #bbbbbb;")
QColor(128, 128, 128)

# NEVER: Unexplained magic numbers in business logic
# Status codes, flags, timeouts, limits, protocol values — use named constants or enums
if status == 3: ...                    # BAD
if key_value >= 0x01000000: ...        # BAD

# NEVER: Visual constants in application code (game_card.py, detail_pages/, etc.)
# ALL visual/layout constants MUST live in themes/standart/styles/constants.py
# Application code reads them via self.theme.CONSTANT_NAME
COMPACT_CARD_WIDTH_THRESHOLD = 150  # BAD: in game_card.py
RIBBON_SIZE_RATIO = 0.28           # BAD: in game_card.py
BADGE_ICON_SIZE = 16               # BAD: in detail_pages/widgets.py
SOURCE_CORNER_RATIO = 0.28         # BAD: in detail_pages/widgets.py
```

```python
# ALWAYS: ≤4 parameters
def process_game(game_id: str, config: dict) -> Game:
    """Process game data."""
    ...

# ALWAYS: Early returns
if not condition1:
    return
if not condition2:
    return

# ALWAYS: Logging
from portprotonqt.logger import get_logger
logger = get_logger(__name__)
logger.info("Game %s started", name)

# ALWAYS: Explicit subprocess arguments
subprocess.run(["cmd", "arg1", "arg2"])

# ALWAYS: Environment variables
API_KEY = os.getenv("API_KEY")

# ALWAYS: Sanitize paths
file_path = os.path.join(BASE_DIR, os.path.basename(user_filename))

# ALWAYS: Use theme constants for styles
shadow.setBlurRadius(self.theme.shadow_blur_radius)
shadow.setColor(QColor(self.theme.color_shadow_card))
label.setStyleSheet(self.theme.CONTENT_STYLE)
QColor(self.theme.color_disabled_text)

# ALWAYS: Add new constants to theme files
# New colors → portprotonqt/themes/standart/styles/constants.py
# New QSS styles → portprotonqt/themes/standart/styles/base.py or submodule

# ALWAYS: Use descriptive named constants, enums, or built-in Qt values instead of magic numbers
STATUS_COMPLETED = 3

if status == STATUS_COMPLETED:
    ...

# ALWAYS: Use named Qt.Key constants with comments for keyboard system key checks
# Skip system/modifier keys (Shift, Enter, Arrow keys, etc.)
if key_value >= Qt.Key.Key_Escape.value: ...
```

---

## Code Modification Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Boria138/PortProtonQt](https://github.com/Boria138/PortProtonQt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
