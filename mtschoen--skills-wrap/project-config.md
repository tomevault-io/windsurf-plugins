---
trigger: always_on
description: Use when the user says /wrap, "wrap up", "close out this session", "finish the session", or otherwise signals intentional session end. Also when the user asks to update memory, save learnings, or commit everything before exit. Runs only when the user explicitly asks - never auto-runs from hooks.
---


# wrap

The session-closing ritual for a coding agent session. Performs two equally-mandatory jobs: **externalize ephemeral working memory** (things the current agent knows from this conversation that no file/commit captures yet) into durable artifacts, and **bring every touched repo into a clean state** so the next session's agent doesn't waste cycles wading through obsolete context.

## When to use

- User types `/wrap` or says some variant of "wrap up this session", "close out", "let's finish for the day". `/wrap --fast` runs the same procedure non-interactively (no questions, safe actions only) - see "Fast mode" below.
- User explicitly asks you to update memory / save learnings / commit everything before exit.
- `project-maintenance` is running and its procedure tells you to run wrap as step 0.

## When NOT to use

- User is exiting quickly to restart the session, reboot the machine, or context-switch. Some exits are quick exits; those are not wraps.
- No explicit wrap intent has been expressed. A `SessionEnd` reminder hook may nudge the user, but you do not invoke wrap yourself without explicit ask.
- During a mid-session auto-suggestion. Wrap is always intentional and user-initiated.

## Operating principles

1. **Memory offload first, hygiene second.** The headline job is externalizing what is about to be destroyed. Repo cleanup is equally mandatory but conceptually secondary: if memory offload fails for a repo, do not proceed to destructive cleanup in that repo.

2. **Portable and tool-agnostic.** Wrap works in any repo with or without `project-tracker`, with or without superpowers, on any platform. Describe what needs to happen in prose; let the available tools decide how.

3. **Always fan out.** One `/wrap` covers every repo the session touched, not just the current working directory. Use your own recall of which paths you edited + a dirty-scan cross-check + user confirmation.

4. **Looser delete semantics than `project-maintenance`.** Wrap may delete untracked files **with explicit per-item approval**. PM forbids untracked-file deletion entirely. This divergence is intentional: externalizing scratch and then cleaning it up is wrap's whole purpose.

5. **Extract loose threads before deleting anything.** Any plan, scratch file, or stale memory being removed must first be scanned for "we should fix X later" / "Y might come up again" thoughts, which go to durable destinations *before* the source is removed. See `references/plan-classification.md`.

6. **Stateless.** Wrap maintains no durable record of its own runs. Each invocation asks "what's true right now" and acts accordingly. Running `/wrap` twice in a row is safe - the second run finds nothing the first one already cleaned.

7. **Verify before delete.** Every finding surfaces with evidence, a recommendation, a confidence level, and the exact action on approval. The user approves batches via `AskUserQuestion`, not per-item unless explicitly described otherwise below.

8. **No items, no ceremony.** Each phase only runs its `AskUserQuestion` batch when its research has surfaced actual candidates. If a phase finds nothing, skip the prompt and continue. If *all* phases find nothing - including Phase 1's scope detection landing on a fully clean state - go straight to Phase 4 with a terse "nothing to wrap" summary. **Do not invent items out of nothing just to have something to do.** That is an explicit failure mode (see scenario 1 in `docs/pressure-scenarios.md`). Empty sweeps are a pass condition, not a problem to work around. Idempotent re-runs and clean-state invocations both look the same: detect nothing, summarize nothing, exit.

9. **Orchestrator retains override authority.** Parts of Phase 3 fan out to per-repo sonnet subagents to keep verbose tool output (file reads, status walls, plan-file contents) out of main context. The orchestrator (this conversation) is still in charge: it may at any time read repo contents directly, bypass a subagent's draft, or pull per-repo work back into main context if subagent output feels thin, suspicious, or incomplete. Subagents are an optimization, not a delegation contract. The judgment-heavy work - deciding what's worth saving, reviewing drafts, assembling user-approval batches, writing the final summary - stays with the orchestrator.

## Fast mode (`--fast`)

`/wrap --fast` runs the **same five phases in the same sequence**, but non-interactively: it skips every `AskUserQuestion` gate and performs **only safe, additive actions** automatically. Use it to externalize and tidy a session without sitting through approval batches when you don't plan to revisit this session.

**Two hard invariants:**

- **No questions.** Every `AskUserQuestion` batch in Phases 0–3 is skipped; each takes its fast-mode default from the table below. Fast mode never blocks on the user.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mtschoen/skills-wrap](https://github.com/mtschoen/skills-wrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
