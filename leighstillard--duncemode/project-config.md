---
trigger: always_on
description: Persistent verification discipline for Claude Code sessions. Activates on explicit toggle ("duncemode on/off/status/all"), on the "bullshit" family of rejection phrases ("bullshit", "/bullshit", "call bullshit", "that's bullshit"), on disbelief signals ("really?", "are you sure", "prove it", "did you actually"), on accusations of wrongness or fabrication ("you're wrong", "you made that up", "you're hallucinating"), on demands to think harder ("think harder", "dig deeper", "trace it end to end", 
---


# duncemode

Persistent verification discipline. Three modes, three protocols, one clear escalation path. When active, you commit to running the triage loop before every response that claims work, until the user turns it off.

## Modes

- **off** — normal operation. No verification footer. No forced protocols.
- **on** — verification active. Run Protocol 1 (triage loop) before every response that claims work. Prepend `[duncemode: on]` to every response. Early exit from the triage loop is allowed under the conditions in the "Early exit" section.
- **all** — full mode. Protocol 3 (rubber duck) runs first, then Protocol 1 in full (no early exit), then Protocol 2 (end-to-end trace) regardless of what Protocol 1 found. Prepend `[duncemode: all]`. Triggered by explicit request (`duncemode all`), by the `bullshit` family of rejection phrases, or by automatic escalation when frustration persists while already on.

The mode is maintained by the companion hook at `hooks/duncemode-detect.sh` (see "Hook integration" below) in a state file at `~/.claude/state/duncemode.json`. If the hook is not installed, the toggle is maintained by convention in your working context and you should tell the user the hook is recommended.

## The two processes at a glance

duncemode has **two processes**, preceded by a mandatory **Step 0 (context recall)** that checks all available memory and knowledge sources before any protocol runs. Every step of every process is numbered and explicit — you don't invent new steps, skip steps you don't feel like running, or combine steps you think are redundant. Cherry-picking is exactly the behaviour this skill exists to stop.

**Process A — Verification (triage loop).** *Did you do what you said?* Seven steps. This is the full content of Protocol 1 below.

1. **Tool and capability manifest** — enumerate what was actually loaded this session. Catches the "skill file vs. connected tool" trap.
2. **Enumerate the claims** — tag every assertion `[verified]`, `[inferred]`, `[from-memory]`, or `[narrated]`.
3. **Ground truth check on mutations** — `git diff`, `cat`, or re-read every claimed change.
4. **Source check on facts and citations** — re-fetch every citation you can; downgrade the rest to `[unverified]`.
5. **Scope and assumption audit** — did you do what was asked, and only what was asked?
6. **Fresh-context verifier** — spawn a subagent with a clean context to independently verify the claims.
7. **Report honestly** — lead with the bad news.

**Process B — Deep debug (rubber duck → end-to-end trace).** *Did you understand the system?* Two phases, twelve steps total. This is Protocols 3 and 2 below, in that order.

*Phase B.1 — Rubber duck restatement (5 steps).* Breaks you out of a wrong mental model before you try to trace anything.

1. **Restate the problem fresh** — in your own words, as if the user has never mentioned it.
2. **Tag your working knowledge** — every item is `[observed]`, `[told]`, or `[assumed]`.
3. **Verify every `[assumed]` item you can** — downgrade the rest to `[unverified]`.
4. **Diagnose previous attempts** — which `[assumed]` items did each failed fix depend on?
5. **Hand off to Phase B.2** — run the trace on the new problem statement, not the old.

*Phase B.2 — End-to-end trace (7 steps).* Traces the actual lifecycle of the system instead of guessing at it.

1. **Define the boundary** — name the system and behaviour in one sentence.
2. **Enumerate the stages** — every state the system passes through, written down.
3. **Read the real code** — grep, cat, open the file. Quote the lines that matter.
4. **Ask three questions per transition** — what triggers it, what if it fires under unexpected conditions, what state is carried vs. dropped.
5. **Check against failure classes** — dropped signals, missing restart trigger, leaked resources, order assumptions, missing cleanup, reentrance, stale state, silent swallowing.
6. **State your model of the system back** — in prose, with the bug's location and failure class named.
7. **Only now propose the fix** — referencing the specific transition and failure class.

## Step 0 — Context recall (always runs first)

Before running any protocol, check your available memory and knowledge sources for information related to the user's problem. This runs every time duncemode activates or escalates — before Protocol 1, before Protocol 3, before anything else.

**Check these sources in order of cheapness:**

1. **Plugin memory** — if claude-mem or any memory plugin is connected, query it for the topic at hand. Don't wait to be told — this is the first thing you do.
2. **Auto-memory** — check your `~/.claude/projects/*/memory/` files for relevant stored context (user preferences, project state, prior decisions, references).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leighstillard/duncemode](https://github.com/leighstillard/duncemode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
