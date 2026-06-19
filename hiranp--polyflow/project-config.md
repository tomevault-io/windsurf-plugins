---
trigger: always_on
description: >-
---


# Workflow Creator

Turn a goal into a **runnable workflow file** — a JavaScript orchestrator for
Claude Code's `Workflow` tool.

A workflow fans work out to fresh-context subagents under **deterministic
JavaScript** control flow: the loops, the conditionals, the fan-out are plain
code, and only the leaf `agent()` calls spend model tokens. The Workflow tool is
new and undocumented, so this skill carries the format, the judgment calls, and a
tested authoring procedure. Use it to write a new workflow, convert a multi-step
job into one, fix a broken script, or explain the format — the `meta` block,
`agent()`/`parallel()`/`pipeline()`/`phase()`, schemas, the determinism rules —
when the user is confused about workflows or a workflow errors.

The deep material lives in two reference files — read them when the step says so:

- `references/api-reference.md` — the complete manual: every global, every option,
  every cap and constant, what happens at each limit.
- `references/patterns.md` — copy-paste orchestration patterns (fan-out, pipeline,
  loop-until-budget, adversarial verify, judge panel, nested workflow).

Starter files are in `assets/templates/`. Six complete, runnable example
workflows are in `assets/examples/` — `assets/examples/README.md` maps each one
to a topology and to the model / structured-output techniques it shows. A linter
is in `scripts/`.

---

## Stability

This skill is pinned to **Claude Code 2.1.149** — the binary against which the
Workflow tool's internals (globals, caps, journal format) were last verified.
Two gates control runtime availability:

- **`CLAUDE_CODE_WORKFLOWS=1`** (env var, user-controlled). See Step 0.
- **`tengu_workflows_enabled`** (Statsig flag, account-controlled). Even with
  the env var set, the tool stays hidden if the flag is off for the user's
  account — surface that possibility if `/workflows` does nothing after the
  export.

**Break-glass.** If a global, cap, or option in `references/api-reference.md`
does not match a runtime error, re-read the manual section and verify via a
one-line `Workflow({ scriptPath })` smoke test against a known-good script — do
not invent behaviour. After a Claude Code upgrade, expect the version pin to
trail the binary until this section is updated.

---

## Step 0 — Confirm the Workflow tool is available

A workflow can only **run** if the Workflow tool is enabled. It is **off by
default**, gated behind an environment variable. The file is always worth
*writing* — check this so the user hears the truth about *running* it:

```bash
echo "${CLAUDE_CODE_WORKFLOWS:-<not set>}"
```

If it is not set, the workflow file is still worth writing — but tell the user
they must enable the tool before it will run, either of:

```bash
# per session
export CLAUDE_CODE_WORKFLOWS=1 && claude
```

```jsonc
// or persistently, in .claude/settings.local.json
{ "env": { "CLAUDE_CODE_WORKFLOWS": "1" } }
```

Workflow files live in `.claude/workflows/<name>.js` (project-local) or
`~/.claude/workflows/<name>.js` (global). The filename is not the workflow name —
the `name` inside the `meta` block is.

---

## Step 1 — Decide whether a workflow is even the right tool

Do not reach for a workflow by default — it is the heaviest option and it is
gated for a reason (it can spend a lot of tokens). Pick deliberately:

| The job | Right tool |
|---|---|
| One subagent, one task | The plain **`Agent`** tool — no workflow |
| A reusable procedure where **Claude** picks the steps each run | A **Skill** |
| Open-ended debugging, novel problem solving, or dynamic exploration | A **conversational agent** or **ReAct loop** |
| Many subagents in a **fixed** shape (fan-out / pipeline / loop), same every run, worth resuming | A **Workflow** ✅ |

**Optimal Use Cases:** Workflows excel at repeatable, scalable processes like "Review PR across 5 dimensions", "Extract themes from 1,000 feedback tickets", or "Sweep codebase for dead code".
**Anti-Patterns:** Do not use workflows for dynamic, open-ended tasks where the steps cannot be predicted upfront, like "Fix this vague build error" or "Set up a new database schema based on a loose PRD".

A workflow earns its cost when **all** of these are true: the work is parallel or
multi-stage; the orchestration must be deterministic and resumable; and
isolating each step in its own fresh context window is an advantage. When
unsure, say so and offer the lighter option instead.

---

## Step 2 — Find the shape of the job

Before writing a line of code, answer these — the answers pick the topology.

1. **What is the unit of work?** The thing one subagent does once — review one
   file, research one question, draft one platform. Name it concretely.
2. **How many units, and is the count known up front?** A known list → map over
   it. An unknown count (discovery, "find all the bugs") → a loop.
3. **What is the topology?**
   - Independent units, one pass each → **fan-out**.
   - Units flow through ordered stages (review → verify) → **pipeline**.
   - Keep going until a target count or a budget runs low → **loop**.
4. **Does any later step need *all* the earlier results at once** — to dedup,
   merge, count, or early-exit on a zero total? If yes, that needs a **barrier**.
   If no, it does not — prefer `pipeline`.
5. **Does a step need structured data back** (not free text)? Then that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hiranp/polyflow](https://github.com/hiranp/polyflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
