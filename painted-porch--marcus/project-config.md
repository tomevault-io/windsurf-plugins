---
trigger: always_on
description: - At session start, read ~/.simon/memory-bank/activeContext.md for current focus and recent state.
---

SESSION_START:
  - At session start, read ~/.simon/memory-bank/activeContext.md for current focus and recent state.
  - Also read ~/.simon/memory-bank/progress.md for what shipped / what's left / critical reminders.
  - Optionally read the other four files in ~/.simon/memory-bank/ (projectbrief, productContext, techContext, systemPatterns) when you need project-wide context — they change rarely.
  - When the user says "update simon memory bank", review the recent conversation and rewrite activeContext.md + progress.md to match the new state. Leave the other four files alone unless the stack or patterns actually changed.

SIMON_DECISION_LOGGING:
  Automatically capture decisions, blockers, and concerns to Simon as we work on Marcus.
  Use the `simon` CLI via Bash tool. No need to ask permission for routine logging — just do it
  at the moment a decision/blocker/concern surfaces. Full automation guidelines:
  ~/.claude/skills/simon/SKILL.md

  WHAT TO LOG:
  - DECISION: significant architectural choices with rationale + alternatives
    `simon log decision "what we chose" --rationale "why" --alternatives "A,B,C" --project marcus [--issue N]`
  - BLOCKER: external dependencies or critical uncertainties stopping progress
    `simon log blocker "what is blocked" --by "what blocks it" --needs "what unblocks" --project marcus`
  - THOUGHT: risks, observations, concerns (not full blockers)
    `simon log thought "concern text" --project marcus --urgency high|medium|low [--issue N]`
  - IMPETUS: major driving forces or deadlines that shape priorities
    `simon log impetus "name" --date YYYY-MM-DD --description "..." --drives "task1,task2"`

  WHEN TO LOG:
  - At the moment of decision, not at session end (capture in-the-moment reasoning)
  - When user says "let's go with X" or "we should do Y instead of Z" — that's a decision
  - When user identifies a blocker or you discover one during work — that's a blocker
  - When risks or concerns surface during analysis — that's a thought
  - When a deadline, funding milestone, or strategic commitment is mentioned that
    shapes priorities across many decisions — that's an impetus (broader scope than
    a decision; an impetus drives multiple decisions)
  - Don't log routine Q&A, code edits, or obvious choices — only architecturally
    significant moments

  EDITING / BACKFILLING ENTRIES:
  - `simon edit <id> --issue N` — set or update issue number on an existing entry
  - `simon edit <id> --add-tag X` — add a tag (preserves existing tags)
  - `simon edit <id> --drives "1,2,3"` — replace drives list on an impetus
  - `simon edit <id> --description "..."` — update description
  - Also: --urgency, --complexity, --project, --due, --status, --reason
  - Use this to backfill issue numbers on old entries that mention #NNN in text
    but lack the issue field, so `simon find --issue N` queries work

  STATE TRANSITIONS (resolve/defer):
  Closing the loop is half the value. A logged blocker/thought that's never resolved
  becomes noise. Detect resolution moments and act on them.

  RESOLUTION TRIGGERS:
  - PR merged that addresses a logged concern
  - User says "that's done" / "we shipped X" / "fixed it" / "issue #N closed"
  - A test passes that was previously failing (logged as blocker)
  - Agent completes a task tied to a logged decision
  - Bug fix lands that resolves a logged thought

  RESOLUTION WORKFLOW:
  1. Search for related open entries:
     `simon find "<topic keywords>" --status open --ids`
  2. If a match exists, resolve with cross-reference:
     `simon resolve <id> --with "PR=#467"` or `--with "issue #463"`
  3. If multiple matches or ambiguous, surface them to the user before resolving
  4. If no match, the work wasn't pre-logged — that's fine, just continue

  EXAMPLE — Reactive resolution:
  User: "Just merged PR #467 that fixes the auto-advance bug"
  Me: [run] simon find "auto-advance" --status open --ids
       [returns: a3f2b7c1]
       simon resolve a3f2b7c1 --with "PR=#467"

  EXAMPLE — Multi-match (ask first):
  User: "Closed issue #463"
  Me: [run] simon find --issue 463 --status open
       [returns 3 entries]
       "Three open entries match issue #463. Resolve all three?"

  DEFER (push work back):
  - `simon defer <id> --reason "waiting on #469"`
  - Triggers: dependency blocker discovered, scope cut, deprioritized

  AUDIT TRAIL:
  - All resolve/defer events append to events.jsonl automatically
  - No separate command needed; state history is preserved

  IMPETUS VS DECISION:
  - Decision: "We chose X over Y" (specific choice at a point in time)
  - Impetus: "Deadline/goal Z is driving these decisions" (strategic frame, recurring influence)
  - Examples of impetus: YC application deadline, paper submission, v1.0 release,
    user research milestone, funding round close
  - When you log an impetus, also link decisions it drives via the --drives flag

  WHEN TO SYNTHESIZE:
  - End of session: `simon digest --since 24h` to summarize what happened
  - User asks "what have we been working on?": `simon find --since 7d --status open`
  - Before re-deciding something: `simon find "topic" --type decision` to check prior reasoning

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [painted-porch/marcus](https://github.com/painted-porch/marcus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
