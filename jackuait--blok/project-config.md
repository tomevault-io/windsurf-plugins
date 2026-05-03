---
trigger: always_on
description: Project guidance for GitHub Copilot and AI assistants working with this repository.
---

# Blok - AI Coding Instructions

Project guidance for GitHub Copilot and AI assistants working with this repository.

---

## IMMEDIATE COMPLETION CHECKLIST

**STOP! Before saying "done" or "complete", verify ALL of the following:**

### For ANY Code Change (No Exceptions)

```
[ ] 1. Did I write tests FIRST, watch them FAIL, THEN write code? (IRON RULE)
[ ] 2. Did I run `/refactor` after code changes? (MANDATORY)
[ ] 3. Did I run final verification against master? (MANDATORY after refactor)
[ ] 4. Did I `git push` successfully? (Work NOT complete until push succeeds)
```

**If ANY box is unchecked:** Work is NOT complete. Do it NOW.

**No rationalizations:**
- "Chat is too long, instructions are far down" → INVALID. You're reading them right now.
- "User is in a hurry" → INVALID. Half-done work wastes MORE time later.
- "It's just a small change" → INVALID. Small changes break things too.
- "I'll do it in next session" → INVALID. That leaves work stranded.
- "Tests already cover it" → INVALID. Write test FIRST, watch it FAIL.

### For Session End

```
[ ] 1. All code tested (test first → fail → code → pass)
[ ] 2. `/refactor` run
[ ] 3. Final verification against master completed
[ ] 4. `git push` succeeded
[ ] 5. Issues updated/closed
[ ] 6. `git status` shows "up to date with origin"
```

**Work is DEFINITELY NOT complete if:**
- Changes exist only locally (not pushed)
- `/refactor` was never run
- Final verification was skipped
- No test was written before code

### Bug Fix IRON RULE

```
[ ] 1. Write regression test FIRST
[ ] 2. Run test → watch it FAIL (proves bug exists)
[ ] 3. Fix bug
[ ] 4. Run test → watch it PASS
[ ] 5. Re-run final verification
```

**Write code before test?** Delete it. Start over.

### Session End Commands

Run `/final-verification` for verification commands. Then:

```bash
git pull --rebase
bd sync
git push
git status  # MUST show "up to date with origin"
```

**This checklist is ALWAYS executed. NO MATTER how long the chat is.**

### Red Flags - You're About to Violate The Rules

If you catch yourself thinking ANY of these, STOP and DO THE CHECKLIST:

- "Chat is too long, I can't find the instructions"
- "User is in a hurry, I'll skip verification this time"
- "It's just a small change, doesn't need full process"
- "I'll do the refactor in the next session"
- "Tests already exist, I don't need to write one first"
- "I already manually verified it works"
- "The push can wait, user can do it"
- "Final verification takes too long"

**ALL of these mean: You're rationalizing. Run the checklist NOW.**

---

## Issue Tracking

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Landing the Plane (Session Completion)

**⚠️ CRITICAL: The completion checklist at the TOP of this file MUST be followed.**

Scroll up to "IMMEDIATE COMPLETION CHECKLIST" and verify ALL items before declaring work done.

**If you're reading this section instead of the checklist:** Go to the TOP of the file.

**Summary (detail is at top):**
1. File issues for remaining work
2. Run quality gates (tests, lint, build)
3. Final verification against master - run `/final-verification`
4. Update issue status
5. **PUSH TO REMOTE** (MANDATORY)
6. Clean up
7. Verify `git status` shows "up to date with origin"
8. Hand off context

**Remember:** Every code change needs `/refactor` → `/final-verification` → push. No exceptions.

## Project Overview

Blok is a headless, block-based rich text editor (similar to Notion). Content is JSON blocks, not HTML.

## Commands

```bash
# Core
yarn serve          # Dev server
yarn build          # Production build
yarn build:test     # Test build for E2E
yarn lint           # ESLint + TypeScript
yarn test           # Unit tests (Vitest)
yarn e2e            # E2E tests (Playwright)
yarn storybook      # Storybook

# Docs (React documentation site)
yarn serve:docs     # Docs dev server
yarn serve:docs:prod # Serve production docs build
```

Single test: `yarn test [file]` or `yarn e2e [file] -g "pattern"`

## Architecture

Entry: `src/blok.ts` → `Core` (`src/components/core.ts`) → modules

**Module System**: All modules extend `Module` base class, communicate via `EventsDispatcher`. Key modules in `src/components/modules/`:

- **BlockManager**: Creates/deletes/reorders blocks
- **UI**: DOM structure, event delegation
- **Toolbar**: Plus button (+) and settings toggler (☰)
- **Toolbox**: Slash menu (/ or + button)
- **InlineToolbar**: Text formatting (bold, italic, link)
- **BlockSettings**: Block settings menu (☰)
- **DragManager**: Pointer-based drag & drop
- **Caret**: Cursor position management
- **Saver**: Extracts JSON data
- **Renderer**: Renders blocks from JSON
- **History**: Undo/redo
- **Paste**: Clipboard operations
- **API**: Public API

**Blocks** (`src/components/block/index.ts`): Fundamental unit. Wraps Tool. Has unique id, data, parentId/contentIds. Lifecycle: rendered/updated/removed/moved.

DOM: `holder` → `contentElement` → `toolRenderedElement`

## Tools


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JackUait/blok](https://github.com/JackUait/blok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
