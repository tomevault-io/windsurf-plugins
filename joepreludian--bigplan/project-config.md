---
trigger: always_on
description: Use when the user asks for a "big plan", "big delivery", a full feature, or any webapp scope that clearly decomposes into 5+ sub-specs with at least 2 backend and 2 frontend slices — orders them (backend first, then frontend, optionally DEVOPS), maintains a master index, and tracks which slices have been implemented via filename status. Make sure to use this skill whenever a request would produce 5+ deliverable specs spanning both an API/database and a UI, when the user mentions decomposing a la
---


# bigspec — Orchestrating Big Webapp Deliveries

## Purpose

Big webapp features die when you try to spec them as one document. The backend, the API surface, the data model, the frontend pages, the wiring — bundled together they produce a spec that's either too vague to execute or too long to review. `bigspec` solves this by **decomposing the big idea into an ordered list of small, self-contained sub-specs**, each of which is itself authored using the regular `superpowers` flow (`brainstorming` → `writing-plans`).

You are the **orchestrator**. You don't write the small specs from scratch — `superpowers:brainstorming` does. You decide *what* the slices are, *what order* they ship in, and you keep the master index honest about which ones are done.

## When to engage

bigspec has a **hard size threshold**. Engage only when **all three** of the following hold:

1. The work decomposes to **5 or more sub-specs** total.
2. At least **2 of those slices are backend** (data model, API, jobs, auth, infra-adjacent).
3. At least **2 of those slices are frontend** (pages, components, state, API client).

If the work doesn't clear that bar, don't use bigspec — go straight to `superpowers:brainstorming`. bigspec adds index-keeping overhead that only pays off when there are enough slices to coordinate.

Other strong signals that you're in bigspec territory (subject to the threshold above):

- The user says "big plan", "big delivery", "big feature", "the whole flow", or anything that signals a larger-than-one-spec scope.
- The user lists many distinct deliverables in the same request.
- A first attempt at `superpowers:brainstorming` reveals the spec covers multiple independent subsystems — back out and use bigspec instead.

### How to check the threshold

If you're uncertain whether the request clears the threshold, do a quick **slice sketch in your head** before committing: list the deliverables you'd expect, tag each as BE/FE/DEVOPS, count. If you can't reach 5 with at least 2 BE and 2 FE, drop bigspec and use `superpowers:brainstorming` instead. Don't pad slices to clear the bar — that produces fake decomposition that wastes everyone's time.

If the count is genuinely close to the threshold, ask the user briefly: "I'm sizing this — sounds like roughly N backend pieces and M frontend pieces. Does that match how you're thinking about it?" Their answer settles it.

## Process overview

```
1. Capture the big picture            (you, with the user)
2. Decompose into slices              (you, with the user)
3. Write master index + TOPLAN files
   one TOPLAN file per slice with a starter prompt for brainstorming  (you)

   ─── SPEC PHASE (brainstorm) ───────────────────────────────
4. For each slice, in order:          (TOPLAN → BRAINSTORMED)
     brainstorm the starter prompt    (superpowers:brainstorming)
     rename TOPLAN → BRAINSTORMED
     emit progress: "X of Y specs brainstormed"
   Gate: do not proceed to plan phase until X == Y.

   ─── PLAN PHASE (writing-plans) ────────────────────────────
5. For each BRAINSTORMED slice:       (BRAINSTORMED → AVAILABLE)
     ask user: run plans in parallel? (default: yes — safe)
     write the plan                   (superpowers:writing-plans)
     rename BRAINSTORMED → AVAILABLE
     emit progress: "X of Y plans written"
   Gate: do not proceed to implementation until X == Y.

   ─── IMPLEMENTATION PHASE ──────────────────────────────────
6. For each AVAILABLE slice:          (AVAILABLE → IMPLEMENTED)
     default sequential, in slice number order
     parallel only if user asks AND parallel-safety check passes
     rename AVAILABLE → IMPLEMENTED   (you, post-merge)
```

You move through this with the user — not in one shot. After step 3 the master index becomes the source of truth and you return to it between phases.

### The two gates

bigspec has **two named gates** that separate the three phases. You will refer to them by name when narrating progress to the user — saying "we're at the spec-phase gate" is faster and clearer than "we've finished step 4 but not started step 5".

- **Spec-phase gate** — between Step 4 (brainstorming) and Step 5 (plan-writing). The gate is closed until **every** slice is `BRAINSTORMED`. While the gate is closed, you must not write any plan, even for slices that are already brainstormed. The gate opens when `X of Y specs brainstormed` reads `Y of Y`.
- **Plan-phase gate** — between Step 5 (plan-writing) and Step 6 (implementation). Closed until **every** slice is `AVAILABLE`. While the gate is closed, you must not implement any slice. Opens when `X of Y plans written` reads `Y of Y`.

Whenever you advance a slice's status, end the message with a one-line "gate state" so the user sees where you are: *"Spec-phase gate: 4 of 7 — closed."* / *"Spec-phase gate: 7 of 7 — open. Plan-phase gate: 0 of 7 — closed."*


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joepreludian/bigplan](https://github.com/joepreludian/bigplan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
