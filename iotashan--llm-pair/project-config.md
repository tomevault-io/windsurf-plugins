---
trigger: always_on
description: >-
---


# llm-pair — calibrated pairing with a peer LLM

Pair the main agent with a **peer LLM** (the "pair") to raise quality through
independent drafting, adversarial review, and consensus — **without** spending
the most expensive model + maximum reasoning on every change.

The pair backend is **Codex** (GPT-5.x), invoked directly via the `codex` CLI
(`codex exec`). The concept is backend-agnostic; see
[Portability](#portability--swapping-the-pair-backend).

## The two problems this solves

1. **Cost calibration.** Run at max (top model + `xhigh`) on everything and a
   one-line fix gets the same super-review as a risky migration; the usage window
   evaporates. This skill picks a **(model, reasoning-effort) tier** sized to the
   task.
2. **Reliability.** Pin the pair to **read-only**, a fixed **working directory**,
   **structured JSON output**, and **verified non-empty results** — and **never
   confabulate** a result the pair didn't actually return.

---

## CONFIG — the tier ladder (edit this block to tune)

The classifier returns a **verdict**; this table maps verdict → (model, effort).
This table is the single source of truth. Verify model IDs against `codex` /
your account; if a model is rejected, fall back one rung toward the stronger
model and note it.

| verdict     | model                  | effort   | when                                                          |
|-------------|------------------------|----------|--------------------------------------------------------------|
| `skip`      | — (do not call pair)   | —        | trivial **and** zero risk signals — not worth pairing at all  |
| `trivial`   | `gpt-5.3-codex-spark`  | `low`    | rename, comment, one-line tweak, isolated copy/config change  |
| `small`     | `gpt-5.4-mini`         | `low`    | small, well-contained change, low blast radius                |
| `normal`    | `gpt-5.5`              | `medium` | ordinary feature/fix, moderate scope                          |
| `big`       | `gpt-5.5`              | `xhigh`  | large, cross-cutting, or high breakage risk                   |

**Effort floor is `low`.** `none`/`minimal` are rejected by the current Codex tool
config (incompatible with the `web_search` tool) — never emit them.

**Fixed overrides (never run the classifier for these):**

- **Planning** (overall plan, implementation plan, implementation pre-work) →
  always `gpt-5.5 @ xhigh`.
- **Blocker / error diagnosis** → classifier runs, but the **floor is `small`**
  (never `skip` or `trivial`).

**Risk signals** (any one blocks `skip` and biases the verdict upward, often to
`big`): auth / authz / sessions, DB migrations or schema, money / billing /
payments, shared libraries or widely-imported utilities, infra / CI / deploy,
public API or contract changes, security-sensitive code, concurrency.

---

## When to pair — granularity (read this first)

**Pairing happens at the work-item boundary, not per edit.** Over-triggering
burns the window faster than the old always-max default.

- A ticket with 3–4 work items → pair ~3–4 times: **plan once** up front, then
  **review each work item** as it reaches a coherent, complete state.
- A single work item with a dozen edits → do **not** pair on each edit. Pair once,
  on the completed unit.
- **Blocker pairing is reactive** — only when an error **survives 2+ fix
  attempts** *or* **cascades into new errors**. Not the first failing test.

### Proactive triggering

You do **not** wait for the user to say "pair with codex." During substantial
**defined task work** (a ticket, a feature, "do XYZ-123"), invoke this skill at
the boundaries above (plan / work-item review / blocker). The `skip` verdict +
work-item granularity keep small work cheap or unpaired, so erring toward
invoking is safe. Don't pair on pure questions or trivial one-off edits.

---

## The four contexts

| Context                         | Collaboration pattern              | Tier               |
|---------------------------------|------------------------------------|--------------------|
| Overall plan / implementation plan / pre-work | **Parallel-draft → converge** | **Fixed: `gpt-5.5 @ xhigh`** |
| Implementation review           | **Draft → advisory review → integrate** | Classifier    |
| Blocker / error diagnosis       | **Advisory diagnosis**             | Classifier, floor `small` |

The pair is **always read-only** — it advises; the main agent writes and
integrates. Only give the pair write capability if the user explicitly asks it to
patch something this turn.

### Context A — Planning (parallel-draft → converge)

Do **not** draft a plan and hand it to the pair for critique — that anchors it on
your framing and you lose the approach you didn't consider. Instead:

1. **Dispatch the pair's independent draft in the background** (run the `codex
   exec` call below, or an Agent, asynchronously). Give it the **same source
   material you have** (ticket, requirements, relevant code) but **not your
   draft**. Use the plan schema.
2. **While it runs, draft your own plan** independently.
3. **Collect both, diff the ideas** — agreements, real disagreements, anything one
   side caught that the other missed.
4. **Iterate to consensus** (see [Consensus loop](#the-consensus-loop)) on the real
   disagreements only.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iotashan/llm-pair](https://github.com/iotashan/llm-pair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
