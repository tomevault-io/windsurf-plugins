---
trigger: always_on
description: |
---


# CodeQuiz — Know Your Codebase

An interactive, gamified quiz skill that tests your understanding of your own codebase.
Human understanding is the force multiplier for AI effectiveness.

## Preamble

```bash
# Check for codequiz updates
_CQ_LOCAL=$(git -C ~/.claude/skills/codequiz describe --tags --always 2>/dev/null || echo "unknown")
git -C ~/.claude/skills/codequiz fetch --quiet 2>/dev/null || true
_CQ_REMOTE=$(git -C ~/.claude/skills/codequiz describe --tags --always origin/main 2>/dev/null || echo "unknown")
if [ "$_CQ_LOCAL" != "$_CQ_REMOTE" ] && [ "$_CQ_REMOTE" != "unknown" ]; then
  echo "UPDATE_AVAILABLE: $_CQ_LOCAL -> $_CQ_REMOTE"
else
  echo "CODEQUIZ_VERSION: $_CQ_LOCAL"
fi
```

If output shows `UPDATE_AVAILABLE`: use AskUserQuestion to ask the user:
"CodeQuiz update available: {old} → {new}. Update now?"
- **Update** — run `git -C ~/.claude/skills/codequiz pull --ff-only` and continue
- **Skip** — continue with current version

If the update fails (e.g. local changes), tell the user and continue with the current version.

```bash
REPO_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
if [ -z "$REPO_ROOT" ]; then
  echo "NO_REPO"
else
  REPO_SLUG=$(basename "$REPO_ROOT")
  echo "REPO_ROOT: $REPO_ROOT"
  echo "REPO_SLUG: $REPO_SLUG"
  mkdir -p ~/.codequiz/$REPO_SLUG
  MASTERY_FILE=~/.codequiz/$REPO_SLUG/mastery.json
  if [ -f "$MASTERY_FILE" ]; then
    echo "MASTERY_FILE: $MASTERY_FILE"
    echo "STATE: existing"
  else
    echo "MASTERY_FILE: $MASTERY_FILE"
    echo "STATE: new"
  fi
  # Quick repo stats for context
  TOTAL_FILES=$(find "$REPO_ROOT" -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.rb" -o -name "*.go" -o -name "*.rs" -o -name "*.java" -o -name "*.swift" -o -name "*.kt" -o -name "*.c" -o -name "*.cpp" -o -name "*.h" -o -name "*.cs" -o -name "*.php" -o -name "*.vue" -o -name "*.svelte" \) -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/vendor/*" -not -path "*/dist/*" -not -path "*/build/*" -not -path "*/__pycache__/*" -not -path "*/venv/*" -not -path "*/.venv/*" 2>/dev/null | wc -l | tr -d ' ')
  echo "SOURCE_FILES: $TOTAL_FILES"
fi
```

If output shows `NO_REPO`: tell the user "CodeQuiz needs to run inside a git repository. Navigate to a repo and try again." Then stop.

If `SOURCE_FILES` is 0: tell the user "No source files found in this repo. CodeQuiz needs code to quiz you on!" Then stop.

## State Management

### Loading State

If `STATE` is `new`, initialize the mastery state in memory:

```json
{
  "version": 1,
  "xp": 0,
  "level": 1,
  "streak": { "current": 0, "best": 0, "last_date": null },
  "badges": [],
  "total_sessions": 0,
  "total_questions": 0,
  "total_correct": 0,
  "modules": {}
}
```

If `STATE` is `existing`, read the mastery file. If it fails to parse (corrupted JSON):
1. Back up the corrupted file: `cp mastery.json mastery.json.bak.{timestamp}`
2. Tell the user: "Your mastery file was corrupted. I've backed it up and started fresh."
3. Initialize empty state as above.

### Schema Migration

After loading, check the `version` field:
- If `version` is missing or less than 1: treat as v1, add any missing fields with defaults.
- If `version` equals 1: current schema, no migration needed.
- For future versions: add migration logic here. Always preserve existing data, only add new fields with sensible defaults. Never delete user progress.

### Saving State

**Save mastery.json after EVERY question** — not just at session end. This protects against mid-session quits. Use the Write tool to write the full JSON state to the mastery file path.

### Streak Logic

When starting a session, update the streak based on `last_date`:
- If `last_date` is null (first session): set `current` to 1.
- If `last_date` is today: do not change streak (already counted).
- If `last_date` is yesterday: increment `current` by 1. Update `best` if `current > best`.
- If `last_date` is more than 1 day ago: reset `current` to 1.

Always set `last_date` to today's date (YYYY-MM-DD format).

## XP & Leveling Reference Table

Define these ONCE — reference them throughout the skill.

```
XP AWARDS:
  Correct answer (Explain This):        +10 XP
  Correct answer (What Happens When):   +20 XP
  Correct answer (Connect the Dots):    +30 XP
  Partially correct answer:             +5 XP (any type)
  Spot-check passed:                    +25 XP (bonus)
  Boss Fight question correct:          +25 XP (per question)
  Boss Fight completed (all correct):   +50 XP (completion bonus)

LEVEL THRESHOLDS:
  Level 1:  0 XP       (Beginner)
  Level 2:  50 XP      (Getting Started)
  Level 3:  150 XP     (Warming Up)
  Level 4:  300 XP     (Building Knowledge)
  Level 5:  500 XP     (Competent)
  Level 6:  800 XP     (Proficient)
  Level 7:  1200 XP    (Advanced)
  Level 8:  1800 XP    (Expert)
  Level 9:  2500 XP    (Master)
  Level 10: 3500 XP    (Codebase Wizard)

MODULE MASTERY LEVELS (based on confidence score):
  Novice:      0.0 - 0.29
  Familiar:    0.3 - 0.59
  Proficient:  0.6 - 0.84
  Expert:      0.85 - 1.0
```

After awarding XP, recalculate the user's level from the thresholds above.

## Badge Definitions

```
BADGE TRIGGERS:
  first_blood:    First question answered correctly
  edge_lord:      5 "What Happens When" questions answered correctly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nessielabs/codequiz](https://github.com/nessielabs/codequiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
