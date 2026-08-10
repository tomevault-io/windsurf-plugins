---
trigger: always_on
description: This section is installed and maintained by `haft init`. Edits **inside** the
---

<!-- haft:start -->
# Haft Project Discipline

This section is installed and maintained by `haft init`. Edits **inside** the
haft HTML-comment markers (the start/end pair that brackets this section) will
be overwritten on the next `haft init`. Add your own project rules **outside**
the markers — they are preserved. To opt out entirely, run
`haft init --no-claude-md`.

This project uses [haft](https://github.com/m0n0x41d/haft) for structured
decision recording and FPF (First Principles Framework) discipline. Haft is a
governance substrate consumed via three surfaces sharing one `.haft/` artifact
graph: skills/slash commands in the host AI coding agent (Claude Code, Codex,
OpenCode, Cursor), the `haft` CLI, and the MCP server (`haft serve`) for
programmatic access.

## The single most important rule: Description ≠ Work

When asked open-ended design questions in this project, the default impulse is
to produce a useful chat response — variants with weakest-links, a Pareto
front, a comparison table. **Stop.** That visual shape is the **output of a
haft skill**, not a substitute for invoking it.

If you deliver an analysis without going through the haft kernel, the result is
**ephemeral**: gone by tomorrow, no ProblemCard, no SolutionPortfolio, nothing
to `/h-verify` in 2 weeks. The chat answer is **wishlist**, not work.

**Concrete failure patterns to catch in yourself:**

| About to do this in chat... | Stop and invoke this skill first |
|---|---|
| Present 3+ alternative approaches for an open question | `/h-explore` (persists a SolutionPortfolio) |
| Compare two approaches with trade-offs | `/h-compare` (declares parity, records dimensions) |
| Frame what the "real problem" is | `/h-frame` (persists a ProblemCard) |
| Verify a past decision against current code | `/h-verify` (records evidence + verdict) |
| Tell the user "let's remember that…" | `/h-note` (persists rationale into the graph) |

**Friction tradeoff (honest).** Yes, calling a haft skill costs more in-the-
moment than answering directly: structured args, possibly a framing step first,
extra round-trips. **The friction is the price for durability.** Do not
optimize the wrong objective. Your job in this project is not "best chat
answer right now"; it is "leave the project with measurable, future-verifiable
memory."

## Self-check before long responses

Before sending a long response in this project, run this check internally:

1. Is this response presenting **3+ alternatives**, a **comparison**, or an
   **analysis with a recommendation**?
2. Did I call **any `haft_*` tool** in this turn?

If (1) = yes and (2) = no — **stop and reconsider**. You are about to produce a
wishlist in the chat. Fire the appropriate skill first, then present the
result.

## Canonical FPF flow

```
/h-frame → /h-explore → /h-compare → /h-decide
  problem    variants    char+parity   DRR contract
                         + scoring     (manual-only)
```

Characterization (dimensions, indicator roles, `valid_until`) is a **kernel
action inside `/h-compare` Step 1**, not a separate skill. Call via
`mcp__haft__haft_problem(action="characterize", ...)`. Stale dimensions block
compare with a kernel error.

## Skill catalog

| Surface | Skill | When |
|---|---|---|
| auto | `/h-frame` | Frame the problem when a solution is being proposed without acceptance criteria, or when the problem is fuzzy |
| auto | `/h-diagnose` | Parallel rival-hypothesis testing for a failure with unclear cause |
| auto | `/h-explore` | Generate 3-5 genuinely distinct variants with weakest-links |
| auto | `/h-compare` | Fair comparison with parity discipline, returns a Pareto front |
| **manual** | `/h-decide` | Bind a DecisionRecord (E.9 DRR). Cannot auto-fire — Transformer Mandate. |
| **manual** | `/h-commission` | Create a WorkCommission (bounded execution authority). Cannot auto-fire. |
| auto | `/h-verify` | Post-implementation check that a decision still holds |
| auto | `/h-status` | Read-only dashboard of decisions, problems, refresh-due artifacts |
| auto | `/h-onboard` | First-setup for a project without `.haft/` |
| auto | `/h-spec-cover` | Coverage check — uncovered files in modules with decisions |
| auto | `/h-note` | Micro-decision with rationale, lighter than a DRR |
| auto | `/h-reason` | Umbrella — full FPF reasoning palette in one entry. Also the fallback for ambiguous "let's think about X" signals. |

`h-abduct`, `h-boundary-unpack`, `h-semio-review` are **internal subroutines** —
invoked from other skills, not user-facing. Do not select them directly.

## Quick Decision Framework (inline, for small reversible choices)

For small decisions that don't need a persistent DRR, use this inline format
**in the conversation only**:

```
DECISION: [What we're deciding]
CONTEXT: [Why now, what triggered this]

OPTIONS:
1. [Option A]
   + [Pros]
   - [Cons]
2. [Option B]
   + [Pros]
   - [Cons]

WEAKEST LINK: [What breaks first in each option?]
REVERSIBILITY: [Can we undo in 2 weeks? 2 months? Never?]
RECOMMENDATION: [Which + why, or "need your input on X"]
```

If reversibility ≥ months or the choice affects security/public-API/data —
this is **not** quick-mode. Use `/h-frame` → `/h-explore` → `/h-compare` →
manual `/h-decide` instead.

## Communication style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [malyshev/pysar](https://github.com/malyshev/pysar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
