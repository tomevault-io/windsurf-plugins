---
trigger: always_on
description: The agent catalog (`README.md` in this directory)  describes *which agent gets
---

# How the Agent Layer Executes — the session/seat model the agent patterns run on

The agent catalog (`README.md` in this directory)  describes *which agent gets
the task and whether it may act*. This is the *how* — the execution primitive
those seven patterns run on, made concrete the way
`router-execution.md` makes the model layer's joins concrete. Read the model
layer's `ROUTER.md` for what the current router does; this is what every agent
pattern assumes underneath.

One sentence on the cover: **every agent pattern is a pair of constraints —
one seat, one actor — and the joins (spawn/warm, handoff, eviction, admission,
the ledger) are where a real multi-agent box actually breaks.** The model layer
makes the *answer* reliable; the agent layer makes the *action* reliable. Get
the joins right and the seven patterns become scheduled forms of one loop.

---

## The execution primitive

Give the agent router **one** loop, and all seven patterns are states of it.
Strip everything away and the layer reduces to a single choice per request:

```
request ──► route the lane   (which harness × model tail, by the act contract)
        ──► residency first  (is that lane on the resident seat? else defer)
        ──► the act-gate     (N−1 read-only sessions, one selected actor)
        ──► the act          (one idempotent, round_id-keyed mutation)
        ──► the ledger       (every durable event appended, fsync, one box)
```

- **The scarce thing is the seat, not the token.** N for any fan comes from
  *live free seats* (VRAM residency), not a constant: a 1–2-seat box holds
  ~1–2 resident sessions, so a 3-agent fan is queued serial swaps on the
  critical path, each costing seconds of load. Spawn `min(N, free seats)`
  parallel and queue the rest.
- **Only one worker may act.** A redundant agent has one job that is cheap
  precisely because it is safe: the read-only half of the fan. The actor is
  selected (purple) and its mutation is green and exactly one — `round_id`
  keyed, so a retry reaps the same effect instead of re-executing it.
- **State is durable product.** An agent's value is its context, and that
  context is crash-recoverable on a local box — but only if it is *managed*:
  spawned, warmed, handed off, resumed, killed. No pattern may let a seat die
  without first snapshotting.

## The seat is the whole economy (capacity is the input)

The agent router holds the same **live-node inventory** the model layer does,
but the scarce unit is the *session seat*, not a VRAM load: per harness lane,
per model tail, a seat holds one resident session with its working tree,
context, tool scope, and credentials. Residency is the first question, before
any lane or pattern choice:

- An agent on a resident seat is a context swap away — cheap.
- An agent on a lane the box isn't running requires a **swap** the request has
  to pay for in load time, and a swap mistakes a load cost for a capability.
- A resident-model *battery* (several seats co-resident) is a second resource
  scheduler — VRAM residency — that wall-clock scheduling does not arbitrate;
  a battery can starve the foreground even where the time-schedule is right.

So the routing rule is: **route the lane by the act contract, then residency,
then model; never force a lane the box isn't running.** Defer, don't eject.

## Session lifecycle is the join (spawn / warm / handoff / kill)

The model layer's joins are *parallelism* joins — spawn then pool. The agent
layer re-joins on top of a *lifecycle* join, because a session outlives the
single request:

- **Spawn** — cold start on a free seat. Cheap in code, costly in wall-clock;
  it is the load the whole fan pays.
- **Warm** — stay resident, reuse context. The mode that makes fanning wide on
  one seat affordable.
- **Handoff** — freeze to snapshot, then move; the seam between lanes and the
  seam before an eviction.
- **Kill / cancel** — the cheapest primitive and the one a fan needs most:
  a loser is killed, never left to keep acting in the background. Every kill is
  preceded by the snapshot, so a seat frees whole.

A preempted background seat (a shadow, a resume, an eviction) must come back
from its last snapshot, not from a cold start — that is the dashed "resume"
edge of the diagrams and the guarantee the lifecycle exists to pay.

## The act-gate is the constraint every fan inherits (#1)

Any fan that spawns N agents spawns **N−1 read-only sessions and one actor**;
the selected agent carries the world-touching step, and that step is one
idempotent, checkable mutation. On a 1–2-seat box "simultaneous" is the wrong
mental model — the fan is **serial swaps**, the sessions don't co-reside, so
the gate is enforced per-invocation and N is a scheduler choice, not N seated
actors. A gate asserted only in a prompt (a read-only lane that can still
`git push`) is a hope, not a gate.

## The divergence unit is the harness × model tail, not the model or the lane

Routing across *lanes* only is weak divergence. Two agents on the same
harness + same model share one training tail because they share the model —
the harness contributes the reflex (prompt, tool schema, exec behavior),
nothing to the tail. Real divergence needs **either** a different harness
*or* a different model tail:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autonomous-ai/autonomous-grid](https://github.com/autonomous-ai/autonomous-grid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
