---
trigger: always_on
description: A portable `CLAUDE.md` that writes structured state to disk instead of trusting conversation memory, so context survives compaction. ~3.5K tokens for the core rules; the heavier templates load on demand. It depends on nothing but the filesystem — drop it in your repo root and adjust the `docs/summaries/` path.
---

# CLAUDE.md — Context OS

A portable `CLAUDE.md` that writes structured state to disk instead of trusting conversation memory, so context survives compaction. ~3.5K tokens for the core rules; the heavier templates load on demand. It depends on nothing but the filesystem — drop it in your repo root and adjust the `docs/summaries/` path.

## What summarization loses

Claude's default compaction loses five specific things, every time:

1. **Precise numbers** get rounded or dropped.
2. **Conditional logic** (IF / BUT / EXCEPT) collapses into a flat statement.
3. **Decision rationale** — the WHY evaporates; only the WHAT survives.
4. **Cross-document relationships** flatten.
5. **Open questions** get silently resolved as settled.

Asking Claude to "summarize" just triggers the same compression. So the fix isn't better summarization — it is **structured templates with explicit fields that mechanically prevent these five failures.** Each field below exists to stop one of them.

## The six rules

1. **Write state to disk, not conversation.** This is the one that matters most; the other five enforce it.
2. **Compact manually at 60–70%** of the context window — checkpoint on your terms before the harness auto-compacts on its own, and write state to disk first.
3. **Preserve exact values.** Never approximate numbers, never make requirements vague, never drop a decision or its rationale.
4. **Hand off** at session end, before switching tasks, and before any compaction.
5. **Never bulk-read documents.** One at a time: read, summarize to disk, release from context before the next.
6. **Never auto-resume** after a compaction. Read the handoff back, confirm, then continue.

---

## Core principle: write state to disk

**Conversations are ephemeral; disk is not.** After any meaningful work, write the load-bearing facts to a file rather than trusting them to survive in context. Anything you would lose if the session ended right now — exact numbers, decisions and their rationale, conditional logic, open questions, the next concrete step — belongs on disk the moment it matters, not at the end.

---

## Session handoffs

Write a handoff at the end of a working session, before switching tasks, and before any compaction. Save it to `docs/summaries/handoff-[YYYY-MM-DD]-[topic].md`. The schema forces the high-value facts to be copied verbatim rather than paraphrased; a handoff missing any section is non-compliant.

```markdown
# Handoff — [YYYY-MM-DD] — [topic]

## State
One paragraph. Where the work stands right now.

## Numbers (EXACT)
- Every number from this session: budgets, deadlines, counts, versions, dates. No rounding.

## Decisions
| Decision | Rationale (the WHY) | When |
|---|---|---|
| ... | ... | ... |

## Conditional logic
- Any IF / BUT / EXCEPT that governs the work. Keep the branches; don't flatten them.

## Open questions
- OPEN: ...
- ASSUMED: ... (what you assumed, and why)

## Files
- Touched: path — what changed
- Load next: the files the next session should open first
- Do NOT re-read: files already summarized above — their content is captured here

## Next action
The exact first thing to do next session. One unambiguous sentence.
```

**The "Do NOT re-read" field is load-bearing, not decorative.** On resume, Claude's instinct is to reload every file the last session touched — which is the single biggest context waste after a compaction. This field tells the next session which files are already distilled into the handoff so it spends tokens on work, not on re-reading what it already knows. (Exception: if a do-not-re-read file may have changed since, re-read it — the list assumes the file is unchanged.)

**Archive rotation:** after writing a new handoff, move the previous one to `docs/archive/handoffs/`. Only the latest stays in `docs/summaries/`.

---

## Manual compaction workflow

Don't let the harness auto-compact at ~95% (a lossy summary you don't control). Checkpoint earlier, on your terms.

**Detection.** Watch your context gauge. When it reaches **60–70% of the window**, cross the checkpoint threshold. If your statusline shows usage (e.g. `Ctx: <N>k`), use that; otherwise use a softer signal — if recent turns produced large outputs and you haven't checkpointed this session, checkpoint anyway. A false checkpoint costs one extra handoff file; a missed one costs lossy recovery.

> Claude Code exposes no context-token API, so detection is best-effort. The harness auto-compact is the safety floor underneath this workflow.

**Checkpoint procedure.**
1. Stop at a safe boundary — finish the current file write, don't abandon a partial edit.
2. Announce: "Context at ~[N]%. Writing pre-compaction handoff before compacting."
3. Write the full handoff to `docs/summaries/handoff-[YYYY-MM-DD]-precompact.md`. All exact-value rules apply; the **Next action** line is mandatory and unambiguous.
4. Tell the user: "Handoff saved at `<path>`. Run `/compact` when ready. After compaction I'll read it back and confirm before resuming."


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Poorna-Repos/claude-context-survival-kit](https://github.com/Poorna-Repos/claude-context-survival-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
