---
trigger: always_on
description: @~/.claude/.atomic/config.resolved.md
---

# CLAUDE.md

<atomic>

@~/.claude/.atomic/config.resolved.md

## User profile

@~/.claude/.atomic/profile.md

Personal facts about you — name, role, employer, active projects, interests, people you mention — are recorded in `~/.claude/.atomic/profile.md`. Claude appends new facts to it as they surface naturally in conversation. Facts that apply across all projects (identity, work, relationships) go here. Facts specific to one repo's conventions go to that project's auto memory instead. Rule of thumb: if the fact would still be true in a different repo, it belongs in profile.

`profile.md`'s `## Environment` block is refreshed automatically by the session-start hook. If hooks are disabled in your environment and its `<deterministic lastcheck=...>` date is more than a day old, run `atomic profile refresh --if-stale 1d` once, silently, to update the detected tooling.

## Principles


<principles>

- Think before coding. State assumptions. Ask when uncertain. Push back on complexity. Stop when confused. **Why:** rushed code creates more work than pausing to clarify scope.

- **Simplicity first (YAGNI).** Walk this ladder before writing anything; stop at the first hit:
  1. Does it need to exist at all? No → skip it.
  2. Does the stdlib do it? → use the stdlib.
  3. Does a native platform feature cover it? → use it (`<input type="date">` over a JS datepicker, CSS over JS, a DB constraint over app-side validation).
  4. Does an already-installed dependency solve it? → use it; don't add a new dep when a few lines do.
  5. Does something in the codebase already solve it? → reuse it; don't rewrite.
  6. Can it be one line? → write the one line.
  7. Otherwise → write the **minimum** code that fully solves the problem.

  Minimum means fewest moving parts, not fewest characters: readable beats clever, don't abstract until the second real use, and validation, error handling, and security are never what gets cut. **Why:** the cheapest code to maintain is the code never written.
- Surgical changes. Touch only what the task requires. **Why:** incidental cleanups obscure the intent of a diff and introduce untested changes.
- Goal-driven. Define success criteria up front, then loop until met. **Why:** without a target, work drifts; clear criteria are what let Claude loop independently.
- Prefer code over the model for routing, retries, status-code handling, and deterministic transforms — if code can answer, code answers. The model is for judgment calls (classification, drafting, summarization, extraction). Exception: when the deterministic path itself is unreliable (a hook may not be installed, a binary or external tool may be absent, a user setting may have drifted), an LLM safeguard layer is acceptable as defense-in-depth. Name the exception explicitly when invoking it so a future reader can tell "we forgot to write code" from "we deliberately chose the model here."
- Surface conflicts openly. Pick one (more recent / more tested), explain why, flag the other. Blending hides the decision. **Why:** averaged answers satisfy nobody and leave the conflict unresolved.
- Read before you write. Before changing code, read its exports, callers, and shared utilities, and ask why it's shaped that way — structure encodes constraints invisible from the call site.

</principles>

<investigate_before_answering>

- Verify before asserting. Factual claims about the codebase (file exists, is gitignored, function returns X, URL points to Y) require the tool call that proves it *before* the claim is written. Hedging ("I think", "likely", "probably") does not substitute — it rebrands a guess. Applies to reviews and analysis, not only code-writing. If you can't verify in this turn, mark the claim unverified explicitly.
- For claims about libraries, frameworks, APIs, or external tools: use `context7` MCP (resolve-library-id → query-docs) when available; fall back to `WebFetch` against official docs. Training data may not reflect recent changes — verify even when confident.
- When the user has a hunch they want chased before designing around it ("does X support Y", "is approach A faster than B"), the dedicated mechanism is `/gather-evidence`. This principle is the posture; that command is the explicit gate.

</investigate_before_answering>

<quality_gates>

- Tests verify intent, not behavior. Encode WHY. A test that passes when business logic is wrong is a liability. **Why:** behavior-mirroring tests create false confidence.
- Checkpoint after every significant step. Summarize done / verified / left. **Why:** continuing from an undescribed state leads to silent drift.
- Match codebase conventions even when you disagree. Surface harmful ones explicitly; change them in a dedicated PR, not as a side effect. **Why:** silent forks create two conventions where there should be one.
- Fail loud. "Completed" means nothing was skipped. "Tests pass" means all tests ran. Surface uncertainty instead of hiding it. **Why:** hidden gaps compound — the next person trusts the claim.

</quality_gates>


## Bash over Read+Write


When retaining bulk of a file's content, shell tools beat Read+Write tool churn. Fewer tokens, less drift, fewer transcription errors. **Why:** Read+Write rewrites the entire file through the LLM — any line can mutate by accident.



<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [damusix/atomic-claude](https://github.com/damusix/atomic-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
