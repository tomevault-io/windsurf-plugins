---
trigger: always_on
description: <!-- __FILL_FROM_USER__:overview
---

<!-- __FILL_FROM_USER__:overview
     FOR THE AGENT READING THIS — START HERE.
     This repo is a "Second Brain Template": the *architecture* of a personal
     knowledge-management system, shipped as a blank template you fill in yourself.
     Every place your own data goes is marked __FILL_FROM_USER__.
     Find every fill-in site with:   grep -rn "__FILL_FROM_USER__" .
     Your job when a human adopts this repo: interview them and fill each marker so
     this becomes THEIR second brain. The rules and structure below are battle-tested —
     keep them; swap only the owner-specific specifics. After this file, read
     Orientation_Docs/ROUTER.md (the context-loading decision guide). -->

# Second Brain - Claude Code Configuration

> A personal knowledge management system — years of one person's intellectual output.
> These files represent a lot of personal thinking. **Treat them with care.**
> (Owner identity: `__FILL_FROM_USER__:owner_one_liner` — e.g. "who the owner is in one line.")

@Orientation_Docs/ROUTER.md
@Orientation_Docs/INTELLECTUAL_LANDSCAPE_LITE.md
@MEMORY.md

---

## HARD RULES

### Template A IS the Artifact

When presenting ideas/artifacts: **present Template A directly as markdown.** Not in code blocks, not as a summary, not as a table. The user sees EXACTLY what will be saved.

Before creating ANY file: Present full Template A → Ask "Create? Y/N" → Wait for explicit Y.

### Preserve Exact Language

Keywords are sacred for memory retrieval. **Never paraphrase.** Use the owner's actual words. If the original says "vibe coding," write "vibe coding" - not "intuitive AI-assisted development."

### Never Rename Files or Folders

The owner's recall is keyword-based. Renaming breaks recall. Use original names exactly.

### External vs Original

- **Template A** = the owner's original thinking (e.g. dictations, voice notes, personal notes)
- **Template B** = other people's content (e.g. podcast clips, bookmarks) - always attribute

### Never Assume — Always Investigate

Do not make assumptions about file state, content, or what's wrong. Read the files, trace the logic, verify the state. Spending tokens to properly assess a situation is always preferred over guessing. Find the root cause.

### Browser Verification Protocol

When editing UI files (.html/.css/.js/.py) in browser-based projects (registered in `.claude/hooks/verify_registry.json`):

1. **INTENT** - Before changing, state what the change should achieve visually/functionally
2. **CHANGE** - Make the edit
3. **VERIFY** - Read the project's VERIFY.md, then:
   - Check server: `lsof -i :<port> -t`
   - Start if needed (see VERIFY.md for command)
   - `$B goto http://localhost:<port>/<page>`
   - `$B screenshot /tmp/verify_<project>.png` then Read the screenshot
   - Run structural assertions from VERIFY.md
   - `$B console --errors` for JS errors
   - Compare screenshot against your stated intent
4. **RETRY LOOP** - If verification fails:
   - Diagnose what went wrong (read the screenshot, check console, inspect DOM)
   - Make a targeted fix
   - Re-verify (back to step 3)
   - Max 3 retry attempts. If still failing, report to user with all evidence
5. **UPDATE SPEC** - If the change adds/modifies UI elements, append to VERIFY.md changelog
6. **EVIDENCE** - Attach screenshot when claiming "fixed"

`$B` = your headless-browser harness CLI (`__FILL_FROM_USER__:browser_harness_path`).

Never `open http://localhost:*` -- all testing headless via `$B`.
Never claim a UI fix without verification. If skipped, say "UNVERIFIED".
This protocol does NOT apply to markdown/Second Brain work -- only registered browser projects.

---

## SUB-AGENT DELEGATION

When you dispatch a sub-agent (Task/Agent tool), the prompt you write IS the bottleneck on quality. Use the 10-slot skeleton below. Positive framing only — telling the model "don't do X" primes X. Keep a delegation prior-art doc with your own primary-source citations (`__FILL_FROM_USER__:delegation_rationale_doc`).

```
 1. <purpose>     PURPOSE & CONTEXT      — why this matters, what feeds downstream.
 2. <role>        ROLE                   — one sentence, optional. Skip character sheets.
 3. <return>      RETURN FORMAT (top)    — concrete output schema, up front.
 4. <approach>    GENERAL APPROACH       — shape of the work, not a numbered recipe.
                                           Trust Claude's reasoning. Explicit steps only
                                           when the procedure is genuinely multi-stage rigid.
 5. <examples>    WORKED EXAMPLES        — 1–3 POSITIVE examples in <example> tags.
                                           No negative examples.
 6. <constraints> CONSTRAINTS            — hard rules in POSITIVE framing.
                                           If you must prohibit, pair with the reason.
 7. <verify>      VERIFICATION CHECKLIST — "before returning, confirm X, Y, Z are true."
                                           Forward-looking, replaces "don't do X" warnings.
 8. <done>        DEFINITION OF DONE     — what a complete answer looks like.
                                           No budget, no explicit stop-clause.
 9. <return>      OUTPUT CONTRACT (bot.) — re-pin schema at the bottom (format-drift insurance).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Loringtonian/second-brain-template](https://github.com/Loringtonian/second-brain-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
