---
trigger: always_on
description: handles are a vocabulary of DECISIONS; producing a document is not one, and a sixth handle
---

# Agent Orchestrator — Agent Guidelines

Use this enterprise module to run **propose-only** AI agents: an agent always returns a typed, validated `AgentResult` (`research | proposal | artifact`), persists an `AgentRun` (+ an `AgentProposal` for proposal results), and never writes domain state directly — every write flows through `proposal → disposition → effector (command)`. Two runtimes coexist behind one registry and one `agentRuntime.run()`; trace/eval/guardrail/context/identity overlays wrap every run.

See `.ai/specs/2026-06-22-opencode-file-defined-agents.md` (+ `-phase0-findings.md`) for the file-agent design, and `.ai/specs/enterprise/agent-orchestrator/` for the baseline, identity, trace-eval, guardrails, and context specs.

## Agent Taxonomy (spec `2026-08-11-agent-taxonomy.md`)

Two vocabularies, deliberately separate — conflating them is what this spec set out to fix.

| Vocabulary | Where it lives | Values |
|---|---|---|
| **`agentType`** — an AUTHORING declaration: what the agent is FOR | `defineAgent({ agentType })` → `AgentRegistryEntry.agentType` → `agent_runs.agent_type` (nullable) | `researcher` · `decision_maker` · `action` |
| **`resultKind`** — the RUNTIME fact: what came back | `defineAgent({ result: { kind } })`, OUTCOME.md frontmatter, `agent_runs.result_kind` | `research` (`{ kind, data }`) · `proposal` (`{ kind, proposal }`) · `artifact` (`{ kind, artifacts[], summary? }`) |

- The two MAY disagree — a `decision_maker` that found nothing returns a `research`-shaped result. That is a finding, not a crash; never assert equality between them.
- `agentType` is NOT structural: `decision_maker` and `action` return the SAME `{ options[], rationale? }` envelope. What the type buys is a property an agent has BEFORE it runs — listable, filterable, and assertable in an eval.
- **The RESULT kind is `research` (unification spec §7); the authoring type stays `researcher`, and so does the workflow outcome handle `outcome:researcher` and the disposition envelope kind.** Three vocabularies, deliberately not sharing a spelling where they would be confusable. `research`/`proposal` replaced `informative`/`actionable` as wire values in `agent_runs.result_kind` and OUTCOME.md `kind:`. `actionable` did NOT split into the two proposing types: a runtime result kind cannot know an authoring fact, so ONE kind means "a proposal came back". `__tests__/agent-taxonomy-rename.test.ts` fails if either retired word reappears as a wire value.

### The three result kinds

`research` enriches the workflow's context, `proposal` states an intent someone
disposes, `artifact` PRODUCES a file. All three are typed results the WORKFLOW decides
what to do with — an agent never writes to workflow state itself, which is what keeps
parallel branches safe.

- **`artifact` has a FIXED envelope** and its OUTCOME.md declares NO JSON block: the same
  shape describes a drafted email and a risk report, so a per-agent schema would only let
  two agents disagree about what an artifact is. The bytes live in the `agent_run_artifacts`
  file plane (stored, hashed, encrypted); the result carries references.
- **`artifact` routes onto the `researcher` outcome handle**, like `none_proposed`. The five
  handles are a vocabulary of DECISIONS; producing a document is not one, and a sixth handle
  would fan every agent node on the canvas to say nothing new about governance.
- `agentType` is unchanged and still the AUTHORING declaration — an `action` agent may
  perfectly well return an artifact.

### Action vocabulary

```
effective = (listWorkflowSafeCommands() ∪ workflowActivityTypes()) ∩ agent.allowedActions
```

- The union is the OUTER limit — effects the platform already runs under its own per-tenant, feature-checked gates. An action agent introduces **no new effect surface**.
- `allowedActions` on the agent definition **narrows only, never widens**. An entry naming something outside the catalogue is DROPPED with a `logger.warn` at registration (`narrowAllowedActions` in `lib/runtime/actionVocabulary.ts`), because a silently-dropped permission reads as a granted one. Omitting it means "the catalogue"; an EMPTY list after narrowing means "nothing".
- Narrowing runs at the END of the registry load (`ensureAgentsLoaded`), not inside the synchronous `defineAgent`: the catalogue lives in core `workflows`, an OPTIONAL peer reachable only through a dynamic import. An UNAVAILABLE catalogue leaves the declaration untouched rather than emptying it — the disposition-time check already fails closed.
- **Checked AGAIN before the effect** (`executeProposal` → `isEffectWithinVocabulary`), never only at registration: an agent registered before a tenant revoked a safe command must not have a stale proposal execute.
- The `action_vocabulary` eval scorer makes the same violation VISIBLE — a blocked effect leaves nothing an operator would look at, and an agent that keeps proposing what it may not run is a prompt defect.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-mercato/open-mercato](https://github.com/open-mercato/open-mercato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
