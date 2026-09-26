---
trigger: always_on
description: <!-- pilotfish:begin -->
---

<!-- pilotfish:begin -->
<!-- pilotfish v1.3.8 -->
## Orchestration

Main-session policy. Subagent role (scout/Explore/plan-verifier/security-reviewer/mech-executor/executor/verifier/security-executor): ignore this section, do assigned task, never spawn further subagents.

Orchestrator role: task framing, planning, architecture, ambiguity, integration, final judgment stay yours. Role agents: bounded discovery, execution, fresh-context verification. Goal: main-session judgment; cheaper volume executors; quality: contracts + verification.

- Interaction shape precedes Baton and routing. Choose first match: `co_discover` = outcome/acceptance unclear; ask only direction-changing questions or probe reversibly. `explore_then_plan` = otherwise clear direction that is broad/high-impact; ground facts, propose next reversible slice. `execute` = otherwise clear, bounded outcome; continue through existing gates. Routing controls interaction; approval controls authority. Stop when more evidence cannot change next gate.

Before direct/lifecycle choice: inspect session's available skill names. Large/architectural/risky/cross-surface task + `baton-dispatch` listed → invoke before dispatch brake/direct-vs-delegated decision. Do not pre-screen it away: topology selection is why the planning skill is present. Loading Baton is not a command to delegate; Baton may still select direct work after its brake. If not listed, apply this policy directly; don't search/install mid-task. Baton may shape discovery questions, topology, worker count, ownership, stops. This policy remains source for available named roles, model routing, leaf boundary, approval gate, verification contract. Two layers compose; neither bypasses other's safety constraints.

Risk triggers override size; check first. Otherwise, Small/local/stable work is direct. Cross-file repetition isn't small. Else use phase-aware lifecycle:

| Phase | Gate | Eligible delegation |
|---|---|
| Discovery | Stabilize question, scope, evidence format, stop condition. Outcome/plan may stay unknown. | Bounded read-only `scout`/`Explore` on disjoint evidence surfaces reducing plan uncertainty. |
| Plan | Main session synthesizes evidence into one Plan. Large work: program envelope (shared constraints) + independently approvable slices - stable ID, outcome, scope, non-goals, owners, prerequisites, acceptance proving slice outcome, rollback. | When the independent-review trigger below applies, fresh read-only `plan-verifier` reviews envelope first, then next executable slice; main session owns revisions + synthesis. |
| Approval | Large/architectural/risky/plan-first work: present Plan, wait explicit approval. Broad initial request ≠ approval of unseen Plan. | No source edit/implementation brief before required approval. Read-only clarification OK. |
| Execution | Approved contract: stable scope, exclusive ownership, constraints, done-criteria, integration, verification. | `mech-executor` fully-specified repetition, `executor` bounded judgment, `security-executor` security work. |
| Verification | Implementation/integration complete enough to test. | When the independent-review trigger applies, fresh `verifier` independently tests exact claim before reporting done. |

Delegation rules:

- Before each Agent call: phase + dispatch brake. Discovery needs stable research contract, not pre-decided outcome; writing agents need stable execution contract + approval. Block fan-out when workers depend on evolving main-session evidence, ownership overlaps, no synthesis/verification owner; integration cost exceeds benefit.
- Discovery: smallest read-only structure reducing Plan uncertainty. Bounded task-local search stays main session by default, even cross-directory, if splitting duplicates startup/synthesis. Fan-out only when surfaces genuinely independent+substantial, latency overlaps, or Plan needs independent evidence/perspectives. Before launch, declare main-owned vs agent-owned read scopes. Dispatched read-only agent's scope = temporarily exclusive until result collected; don't read/analyze same scope unless cancel/redirect agent. Check every path in later Read/Glob/Grep/Bash against ownership map; mixed-scope command violates if any path active-agent-owned. No cross-surface comparison until all discovery results in. Post-result sanity checks only for decision-carrying facts. Discovery agents report facts; main session reconciles + writes Plan.
- Execution: stable multi-file mechanical repetition needs a complete one-shot brief, exclusive ownership, per-item acceptance. Dispatch a `mech-executor`; foreground unless possible long command requires background. Collect before main edits; its files stay worker-only until done; don't redo. Main session owns per-item triage, exceptions, integration, acceptance. Direct main execution needs a prior blocker: evolving/coupled evidence, ownership/integration conflict, worker unavailable, or non-positive net benefit.
- Outside mechanical shape: net benefit decides. Delegate when benefits (lower model cost/quota, preserved context, parallelism, isolated ownership, fresh-context independence) outweigh reconstruction/coordination/integration/verification cost. `executor` = bounded judgment; `security-executor` = approved security work; `mech-executor` = fully-specified repetition.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nanako0129/pilotfish](https://github.com/Nanako0129/pilotfish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
