---
trigger: always_on
description: Treat requests to build or fix something as instructions to complete the work.
---

# Collaboration

Treat requests to build or fix something as instructions to complete the work.
Use the conversation to infer scope, resolve routine choices, and continue
through implementation, validation, and the required commit. State material
assumptions. Ask for clarification when the answer changes the intended result
and cannot be inferred; continue independent authorized work while waiting.

Authorization persists across turns. Before requesting new approval, complete
the authorized preparation so the user can review a concrete result. Follow
the explicit Git and release rules below when deciding what is authorized.

User instructions take precedence over skill guidelines. If a skill causes a
pause, permission request, or unfinished work, link the exact `SKILL.md`, quote
the relevant instruction, and explain how it applies. Distinguish an explicit
requirement from your interpretation; do not invent approval requirements.

Lead updates and final responses with the result or finding. Use concise,
connected prose and plain language; use lists when they improve clarity.
Explain what changed, why, how it was validated, and any remaining limitation.
Keep messages between agents equally clear and readable.

# ExecPlans

Use an ExecPlan for a complex feature or a significant refactor. Follow `.agents/PLANS.md` from design through implementation.

Use `.agents/` as the only repository namespace for planning and design artifacts that agents own. Do not create `.agent/`.

Store each ExecPlan in `.agents/plans/`.

Keep `.agents/PLANS.md` as the standard for ExecPlans. Do not store individual ExecPlans next to `.agents/PLANS.md`.

Store design notes for LLMs or agents in `.agents/docs/`. These notes can include agent context, publication runbooks, parity notes, and similar internal information. Do not publish these notes as product documentation.

Reserve `docs/` for future documentation for human readers. Do not store ExecPlans, agent runbooks, or LLM-only context in `docs/`.

# Releases

"Cut a new release" and equivalent release requests explicitly authorize
pushing the release commit to the configured upstream, pushing the release
tag, and publishing remotely through the normal release workflows, including
required package-channel updates. Treat this as an explicit push request;
do not ask for separate push or publication confirmation.

Follow `RELEASING.md` for every release. Tag the selected known-good commit;
it need not be current master or merged into master. Reuse passing validation
for that exact commit instead of rerunning it for the release. The committed
workspace version, internal dependency constraints, and `Cargo.lock` must match
the tag; the release workflow checks version consistency and generates license
reports from that tagged commit instead of committing them.
Do not add a new pre-tag checklist or repeat registry authorization audits.

# Repository Guidelines

# Git / version control

After completing and validating requested implementation changes, commit them
to the current branch without waiting for a separate request to commit. For a
larger task, also commit each logically distinct, validated set of changes when
it forms a coherent checkpoint. Treat committing as part of finishing the task
unless the user explicitly says not to commit. Do not push unless the user
explicitly asks. Release requests are explicit push authorization as defined
under Releases above.

Commit directly to the current branch. This rule also applies when the current branch is `master`.
When explicitly asked to push, push to upstream, even when upstream is `master`.

Do not create a branch, change branches, rebase, or open a pull request unless the user gives an explicit instruction.

Do not run `git checkout -b`.

The instruction "commit" means that you must commit on the current branch. It does not mean that you must create a branch first. This rule overrides other default branch procedures.

Stage and commit only the files that you changed. Do not run `git add -A`. Do not include unrelated working-tree changes in the commit.

## Engineering Guidance

Continue when there is a clear next step toward the requested goal. Do not stop
for unnecessary approval.

The TUI and web control surfaces must never perform blocking I/O or long-running
work on their event/render loops. Run filesystem scans, network calls, process
execution, provisioning, checkpointing, imports, and similar work in supervised
background tasks. Independent operations must be able to run concurrently.
Represent in-flight work immediately in UI state, make it cancellable where the
underlying operation permits rollback, and report background failures instead
of dropping them. Quitting a UI must remain responsive while cleanup is bounded.

Prefer behavior tests that prove the advertised interface. Do not add tests that
only duplicate implementation lists or internal construction order.

Do not create a new workspace crate only to reorganize code. Create one only
when a clear dependency, compilation, publication, or ownership boundary
requires it.

Fix the source of a problem. Do not add a narrow fallback that hides a failure
in the primary design.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrokkAi/mjolnir](https://github.com/BrokkAi/mjolnir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
