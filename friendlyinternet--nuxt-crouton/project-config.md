---
trigger: always_on
description: The **stack-neutral constitution**: how we work, independent of what we build with. This is the
---

# AGENTS.md — the portable method

The **stack-neutral constitution**: how we work, independent of what we build with. This is the
**Method** layer (epic #952). The **stack adapter** — framework/DB/host/UI specifics and the tools
that implement these gates — is `CLAUDE.md`. A team on another stack keeps this file, swaps that one.

> Layers: **Method** (this file) · **Stage-profile** (`harness.config.mjs`) · **Stack-adapter**
> (`CLAUDE.md`). Skills/agents carry a `layer:` tag — `node scripts/harness-layers.mjs`.

## Working style

Clarity over ceremony. Start simple; add complexity only when proven necessary (KISS). Reuse before
building — check the ecosystem first. Wrap async work in error handling, return `{ data, error }`.
Write general-purpose solutions, not ones fitted to the example. Match the surrounding code's idiom.

## The loop

`issue-first → decompose → stage-gated work → sign-off gates → commit → observe → retro`

1. **Issue-first (HARD GATE)** — open the tracking issue (epic + sub-issues if multi-step) *before*
   writing code. A missing issue is a failing build. The issue is the unit of work.
2. **Decompose** — an initiative → an epic + a tree of single-coherent-change sub-issues.
3. **Stage-gated work** — which gates fire depends on the work's **stage** (below).
4. **Sign-off gates** — get a human to sign off on the *right thing* before anything expensive.
5. **Commit** — small, atomic, conventional.
6. **Land via a PR** (`Closes #NN`), never a direct push to trunk.
7. **Observe + retro** — measure the harness; postmortem at epic close.

## Stages

A **declared** concept, not a hardcoded folder name. `harness.config.mjs` maps each stage to its
paths, the gates that fire, its deploy target, and whether edits are guarded. Resolve with
`node scripts/harness-stages.mjs <path>` (or `stageForPath`/`gateMode`) — don't match folders by hand.

Default profile (rename/repoint in that one file — e.g. `poc`→`spike` for scrum; gates travel with
the stage, not the name):

- **incubator** (`poc`/`spike`) — safe-to-fail; no required gates; preview deploys.
- **launched** (`app`) — real apps; test-first opt-in; staging deploys.
- **shared** (`package`) — many consumers inherit it; test-first required; edits guarded.

Edit-guarded stages need explicit, scoped approval before a change (granted once per initiative,
never committed).

## Sign-off gates

A probabilistic runtime needs gates where a human should decide. A gate proposes the right thing and
**holds** before the expensive step. Pick by *what the change is*:

- **Schema** — agree the data model before generating from it.
- **UI** — agree the look (preview or mockup) before building.
- **Test** — agree a committed *failing* test as the contract before writing the logic. Done = it passes.
- **Code review** — the diff, for correctness + simplification.

One shared loop: hold on a `blocked` status; the **only** resume signal is a reply comment containing
`lgtm`/`approve` — not a reaction, not a label. Anything else is a change request. When unsure a diff
is in scope, don't gate.

**Done is signed off, not asserted.** A unit of work is "done" only when the thing it promised is
*checked and concretely signed off* — never on a proxy. A green build, a passing typecheck, a deploy
URL, the agent's own confidence: every one of those can be true while the work is wrong (each lied
during a real graduation, #988). So status is **derived from a recorded sign-off**, not self-asserted:
no `lgtm`, not done. Where the work has an enumerable contract (a behaviour spec, an acceptance list),
"done" is *per entry* — each one checked and signed, not the set waved through at once. And the verdict
comes from **comparison against the expected result**, not from re-reading the list: a list can't
reveal its own gaps; running the real thing next to the contract can.

## Issues — the unit of work

- **Search before creating** — sessions are ephemeral; continue an existing epic, don't duplicate.
- **Map every issue to a real component**, never "root". One `type:*`; an epic carries `epic`. Add a
  new component's label when you add the component.
- **Status the moment you start** (`in-progress`), `blocked` when waiting, drop on close.
- **Recurring chores are standalone**, never sub-issues of a deliverable epic (a never-closing child
  pins the epic open forever).

**Write as a hypothesis** (default; trivial chores opt out) — open the body with `## Hypothesis`:
*We think that* if X then Y · *We'll do that by* … · *We'll be right if* … · *We'll know by* …

**Record what you didn't do** — when alternatives were real, a `Considered & rejected` note
(`option → ❌ why not`) stops future-us re-litigating it.

**Two audiences**, in this order: **👤 humans** lead — plain language, what changed + why it matters
(a diagram only if it clarifies); **🤖 agents** — scope, exact paths/symbols, behaviour, acceptance,
links. (Same split in PRs and commit bodies.)

**`## 🧪 How to test`** on every closeable issue/PR — for someone who knows the concept, not the
code: what changed, the concrete surface, numbered steps with before/after, test data. It's the
acceptance check.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FriendlyInternet/nuxt-crouton](https://github.com/FriendlyInternet/nuxt-crouton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
