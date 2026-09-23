---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Documentation

**Comments describe the destination, not the journey.** A reader needs what the
code does and why it is that way, not how you got there.

**The trap exception is about content, not form.** A trap someone would
otherwise fall into earns its comment - but state it as a present-tense rule.
These two carry the identical warning:

```c
/* Trap: writing along rows instead is ~20% slower - the per-column state
 * moves from registers into arrays. */
```
```c
/* Turning this inside out to write along rows was tried and is ~20% slower. */
```

The second is the same fact told as a war story. Only the first is allowed.
This is the rule that gets broken most, because almost any rejected alternative
can be called a trap - so the test is the wording, not the justification. If you
write *was tried*, *used to*, *previously*, *at first*, *the old code*,
*turned out* or *we tried*, you are narrating. Rewrite the sentence as a rule.

**Be brief - and check it, don't feel it:**
- A blank line inside a comment block usually means it is too long. One
  paragraph is the target; a second needs a reason.
- A comment longer than the code it explains is suspect. File headers are
  exempt; they carry the file's parts map.

The journey belongs in the commit message, if anywhere.

**A document has a reader. Write to them, not to the next implementer.**
A guide, a reference or a README is read by someone trying to *use* the
thing - a theme author, a player owner, whoever runs the script. They are not
deciding whether the design was right, and telling them it was is noise. This
is the same fault as narrating in a comment, one level up: it is writing to
the wrong reader.

Three things to cut:

- **Design defences.** *and there should not be*, *that is deliberate*, *on
  purpose*, *considered and rejected*, *would be worse*, *is the right call*.
  The reader cannot act on any of it. A recommendation is not a defence -
  "use `auto` unless you need to pin the colours" tells them what to do, and
  stays.
- **Mechanism they cannot see.** Internals earn a line only when they predict
  something the reader will hit. "one cached bitmap serves every row" earns
  its place, because it says why no filter is offered; "the row config is
  remembered once the tag renders and is never cleared" does not.
- **Source paths and line numbers**, in any document whose reader does not
  have the tree open. `settings_list.c` means nothing to someone writing a
  theme. An internals document is the exception and should say so at the top.

The test, sentence by sentence: **what does the reader do differently for
having read this?** If the answer is nothing, it belongs in the commit message
or in `.specifications/`, not in the document. Apply it hardest to the
sentence you were proudest of.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthonyfletcher/podbox](https://github.com/anthonyfletcher/podbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
