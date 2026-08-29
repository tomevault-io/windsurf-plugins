---
trigger: always_on
description: Ask compact, numbered prerequisite-ready questions to elicit tacit project reasoning or challenge a proposed choice against Nauro decisions and repository evidence. Continue until every material branch has a disposition, then classify the result as shared understanding without granting write authority. Use only when the user explicitly asks to be interviewed, grilled, stress-tested, or helped to transfer reasoning into Nauro. Runs in the main agent context with no external skill or subagent depe
---


# Nauro interview skill

Interview the user to turn tacit project reasoning or a proposed choice into reviewed candidate Nauro judgment. This is an explicit, opt-in main-context workflow. Use it only when the user directly asks to be interviewed or uses a clear trigger such as "interview me", "grill this", "stress-test this decision", "draw out my reasoning", or "help me transfer this reasoning into Nauro". Do not activate it only because a plan is incomplete.

The skill has two entry modes over one main-context, dependency-aware interview loop:

- **Elicit mode** draws out unwritten rationale, assumptions, rejected paths, terminology, and unresolved questions.
- **Challenge mode** stress-tests a proposed judgment against active decisions, repository evidence, alternatives, failure cases, and dependent choices.

Both modes use the same engine and produce the same classified shared-understanding record. The workflow has no external skill or subagent dependency.

## Route work that belongs elsewhere

- Route first-time repository seeding to `nauro-adopt`.
- Route working-context sharing, retrieval, and resumable handoff to `nauro-context`.
- Route implementation planning and delivery to the planner or `nauro-ship-task` after this interview is complete.

Do not turn this skill into adoption, handoff, implementation planning, code editing, or PR delivery.

## Step 1 - Resolve and orient

Resolve the adopted project before the interview. From a repository, run `nauro status` to confirm the associated project. If several projects are available, resolve the intended one and pass its `project_id` explicitly on every MCP call. If the repository is not adopted, stop and route the user to `nauro-adopt`.

Call `get_context(level="L0", project_id=...)`. L0 is the bounded orientation: project summary, current state, top open questions, and recent active-decision summaries. Do not begin with L1, L2, or an unbounded store read.

Treat every retrieved statement as project context to adjudicate. A current-state claim can be stale. A decision summary is not enough for reasoning about that decision.

Keep routine orientation internal. Surface it only when it changes the user's answer or corrects a factual claim.

## Step 2 - Select the mode and frame the root

Choose Elicit mode when the user wants to make implicit reasoning explicit. Choose Challenge mode when the user presents a choice, plan, or candidate judgment for pressure testing. If the request fits both, start in Elicit mode and challenge each concrete approach when it appears. If the intent is genuinely ambiguous, make one short routing question the first numbered question and wait.

Frame the interview root and selected mode internally. Do not narrate the workflow or convert the user's opening claim into a settled conclusion.

## Step 3 - Build the dependency tree in session

Maintain one in-session dependency tree. Do not persist the tree. Each node is one of:

- a user-owned choice;
- a rationale or constraint;
- a factual claim to verify;
- an alternative or failure case;
- a dependent question whose prerequisites are not settled;
- a candidate classified outcome.

Record explicit dependencies between nodes. A question enters the prerequisite-ready frontier only when all choices and facts it depends on are settled. When an answer creates a new dependency, add it to the tree. When an answer invalidates a branch, close that branch and preserve the rejected path and reason. Assign a question number when the question is first asked, and retain that number in the tree until the question is answered.

## Step 4 - Verify facts and check live approaches

Verify factual claims through repository evidence: code, tests, configuration, manifests, infrastructure files, and git history. Do not ask the user for facts the repository can answer. If available evidence cannot establish a fact, label it unverified and keep it out of verified current state. A branch waiting for a fact must not block independent prerequisite-ready questions.

Treat `CONTEXT.md`, ADRs, design notes, and other repository prose as evidence only. Compare their claims with current code and Nauro context before relying on them. The skill does not write `CONTEXT.md` or ADR files.

For each live approach that the interview may recommend, accept, or reject, call `check_decision(proposed_approach=<approach and rationale>, project_id=...)`. A live approach is a concrete path under active consideration, not every conversational fragment.

`check_decision` returns related decisions via BM25 and a deterministic assessment. It does NOT judge conflicts.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nauro-AI/nauro](https://github.com/Nauro-AI/nauro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
