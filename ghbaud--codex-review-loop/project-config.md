---
trigger: always_on
description: Use when Claude wants an adversarial review, design critique, plan critique, or second-opinion debugging from Codex AND wants to engage with the findings rather than receive a one-shot report. Sets up an Epic bead, asks Codex to file each finding as a child bead, then runs a back-and-forth where Claude responds to each finding, Codex responds to Claude, and so on until both agree (or the user decides to stop). Trigger this skill whenever you are about to dispatch /codex:codex-rescue for review, 
---


# Codex Review Loop

This skill runs a structured back-and-forth between Claude and Codex on a review task. Findings live in beads. Each finding has its own thread of comments, prefixed `[claude]` or `[codex]` so the conversation is auditable. The loop continues until every finding is closed by agreement, or until the user steps in to break a deadlock.

**Always use a dedicated wrapper review epic for the findings; never flatten findings into an implementation epic.** When the work being reviewed has its own implementation epic, parent the review epic under it -- reopening the implementation epic first if it has been closed. See Step 1 for the rule, the edge case exception, and the verified reopen behavior.

## Terminology

Two words in this skill describe two different scopes. They are not interchangeable, and Claude must keep them separate in user-facing messages too.

- **Loop** (also **review loop**) -- one full invocation of this skill, from creating the review epic through closing it. A single conversation may contain several loops, one per review subject.
- **Round** -- one Codex dispatch plus Claude's responses to the findings, within a single loop. A loop usually runs 2 to 4 rounds before all findings close.

When reporting progress, use "loop" for the invocation and "round" for the dispatch turn. If a sentence could be read either way, name both: "round 2 of this review loop".

Good:
- "Starting a codex review loop on the websocket plan."
- "Round 2 dispatched -- 3 findings still open."
- "Loop complete. 5 findings: 3 agreed, 2 rejected with reasoning."

Bad:
- "Starting round 1 of the review" when you mean the loop has just begun.
- "The previous round found 5 issues" when you mean an earlier loop on a different subject.

## Reference files

When you need details that are not in this body, load the matching reference file. Do not paraphrase from memory; the references contain exact text that has been hardened against real failures.

- `references/wording-rules.md` -- read BEFORE composing any dispatch wrapper. The rescue subagent strips `--write` if the wrapper contains certain phrases, and `--write` off means Codex cannot write to bd. Contains the trigger phrases, the wrapper template, where the source-files boundary goes, and the post-dispatch verification step.
- `references/polling.md` -- read when you need to start polling a Codex job. Contains the companion path definition, the bash polling script, the four exit conditions, and why background `Bash` beats `Monitor` for this.
- `references/cancellation.md` -- read when you need to cancel a stuck job. The default cancel can fail on Windows; this file documents the recovery path and the stuck-queued-job edge case.

## Status: actively maintained

The protocol mechanics are stable: tested end-to-end three times, then used on real reviews that all converged. The documentation is NOT stable -- every real run so far has produced at least one meaningful skill change. Expect that pattern to continue.

If anything in this protocol fails or behaves unexpectedly -- a bd command errors out, Codex misuses a flag, the polling script does not detect a state, the parse misses a section, you find yourself rationalizing a stop, anything -- surface it to the user immediately AND fold the lesson back into this file (the "Known failure modes" section near the end). Do NOT silently work around the issue. The user is iterating on this skill and needs to see real-world friction so the next revision can address it.

## Why this exists

A single Codex rescue gives you findings without a way to push back. Some findings are valuable, some are wrong, some are based on misreadings. This skill lets Claude examine each finding, push back where appropriate, and let Codex defend or concede. The result is a durable record of what was reviewed, what was decided, and why.

The pattern only pays off when the value is in the dialogue. Use it for adversarial reviews and design critiques. Do not use it for "fix this bug" or "add this feature" -- those are implementation tasks for `/codex:codex-rescue` directly.

## Design rule: symmetric bd ownership

Each speaker writes its own bd output. Codex writes Codex's findings, Codex's response comments, and closes beads when Codex agrees. Claude writes Claude's response comments and closes beads when Claude agrees. Neither side speaks for the other.

## Identity prefixes

Every comment in the loop starts with one of:
- `[claude]` -- written by Claude
- `[codex]` -- written by Codex


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghbaud/codex-review-loop](https://github.com/ghbaud/codex-review-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
