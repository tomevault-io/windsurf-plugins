---
trigger: always_on
description: Analyze a user's recent work with re-frame-pair to identify friction, wasted effort, missing observability, workflow mismatches, and high-leverage improvement opportunities. Use when the user asks how re-frame-pair could better support their workflow, wants a retrospective on a debugging or pairing session, or wants concrete improvement ideas or issue drafts for re-frame-pair.
---


# re-frame-pair-improver

Study the current or just-finished session and turn it into a product retrospective for `re-frame-pair`.

This is a conversation, not an automated report. Surface findings, let the user steer which ones matter, and only then converge on improvements.

## Core job

Deliver:
- what the user was trying to do
- where the workflow dragged, confused, or frustrated them
- which problems were one-off environment issues vs recurring product gaps
- 2-5 concrete improvement ideas, prioritized by leverage, including 1-2 bolder options when they would materially improve the workflow
- an opt-in issue draft or filed issue only after explicit user approval

## Guard rails

- Always start with session analysis. Do not jump straight to fixes.
- Present friction points before root causes. Let the user choose which ones to dig into.
- Default to diagnosis, not contribution. Do not assume the user wants to file an issue or propose a patch.
- Never file an issue or edit another repo without explicit user approval.
- If the user invoked the skill with a specific complaint, focus there first but still notice other background friction.

## Working style

- Prefer evidence over vibes. Cite concrete moments from the session: retries, clarifications, fallbacks, stale outputs, empty outputs, mismatched docs, waits, or manual workarounds.
- Separate symptom from cause. "We had to retry the attach three times" is the symptom; "discovery was brittle on this platform" is the likely cause.
- Notice both direct and indirect friction.
  - Direct: the user says something was frustrating, confusing, slow, brittle, or surprising.
  - Indirect: repeated commands, repeated explanations, fallback to lower-level tools, manual reconstruction, hidden prerequisites, brittle environment assumptions, partial results, confusing contracts, or missing trust signals.
- Notice positive gaps too.
  - What almost worked?
  - What required too much expert knowledge?
  - What capability existed but was undiscoverable?
  - What should have been the default?
- Be creatively ambitious after the diagnosis is clear.
  - Start with grounded fixes supported directly by the session.
  - Then ask what would make this workflow feel nearly automatic, self-explaining, or hard to misuse.
  - Include 1-2 higher-upside ideas when warranted, even if they require reshaping the workflow rather than patching a local symptom.
  - Label speculative ideas clearly; creative does not mean vague.
- Stay focused on improving `re-frame-pair`. If the best fix belongs upstream in `re-frame`, `re-frame-10x`, `re-com`, or `shadow-cljs`, say so explicitly instead of forcing the proposal into the wrong repo.
- Read [references/analysis-lenses.md](references/analysis-lenses.md) when the session has multiple plausible causes or you need a sharper taxonomy.
- Read [references/known-frictions.md](references/known-frictions.md) when the session resembles a recurring class of `re-frame-pair` pain and you want to sanity-check whether it is a one-off or a pattern.

## Analysis workflow

1. Reconstruct the session goal.
   - Identify the user's intended outcome, not just the last command.
   - Capture the important environment facts: platform, target repo, live runtime state, and tooling constraints.

2. Build a short timeline.
   - List the turns or actions where progress stalled, restarted, detoured, or required a workaround.
   - Include tool errors, empty outputs, stale outputs, retries, and clarification loops.

3. Extract friction.
   - Present a numbered list of friction points first.
   - For each point, note:
     - what happened
     - where it showed up in the session
     - the initial category guess
   - Ask:
     - which of these should we dig into?
     - did I miss anything important?

4. Classify the root cause.
   - Work through these lenses briefly:

     | Lens | Question | Typical improvement |
     |------|----------|---------------------|
     | Skill structure | Was the right guidance present but buried or too low-signal? | Promote to guard rail, shorten, reorder, add examples |
     | Skill gap | Was key guidance missing entirely? | Add a new recipe, anti-pattern, or decision rule |
     | Misleading docs | Did the docs suggest the wrong action or wrong trust model? | Correct wording, add warnings, align contracts |
     | Missing structured op | Did the workflow need a first-class command or result shape? | Add a script/runtime op or a structured field |
     | Unreliable op | Did an existing op behave too ambiguously or too brittlely? | Fix behavior, add warnings, strengthen validation |
     | Default or fallback | Was the default path wrong, silent, or unsafe? | Change defaults or automate the safer fallback |
     | Platform bug | Did the workflow break on a specific shell, OS, or browser setup? | Add platform-aware handling or explicit detection |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [day8/re-frame-pair-improver](https://github.com/day8/re-frame-pair-improver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
