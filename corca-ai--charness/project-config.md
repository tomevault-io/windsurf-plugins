---
trigger: always_on
description: `charness` is a Claude Code / Codex plugin developed by [Corca](https://github.com/corca-ai).
---

# Charness - Corca Harness

`charness` is a Claude Code / Codex plugin developed by [Corca](https://github.com/corca-ai).

## Start Here

- [docs/design-north-star.md](./docs/design-north-star.md) governs: brief a
  capable judge and keep teeth only where a wrong answer escapes. At irreversible
  boundaries, confirm through a different observer and evidence channel.
- Load matching skills before improvising, and continue active repo work from [docs/handoff.md](./docs/handoff.md).
- Before changing operating contracts, prompts, skills, exports, or artifacts,
  read [recent lessons](./charness-artifacts/retro/recent-lessons.md).
- Cautilus is eval-only and ask-before-run; use the repo planner/wrapper from
  [cautilus-on-demand.md](./skills/public/quality/references/cautilus-on-demand.md).
- Keep host behavior in adapters/presets/manifests, and prefer executable
  validators plus structured state over prose rituals.

## Skill Routing

At session start, a pickup follows docs/handoff.md `## Workflow Trigger`; ordinary requests use installed skill metadata and model judgment to start the matching workflow directly. When hidden support/integration availability is unclear, run the read-only `charness catalog list --repo-root <repo> --summary` inventory. Treat its facts only as inventory; if the command returns nonzero, report the command failure. If the SessionStart hook is installed it may inject this context; it remains context-only.

External URLs or source links that should become working context for this repo route through `gather` before summarizing, implementing, or deciding from them.

Validation-shaped closeout or operator reading test requests go through `quality` validation recommendations before HITL or same-agent manual review.

Keep this block short. Detailed routing belongs in installed skill metadata and model judgment, not in a long checked-in catalog.

## Subagent Delegation

- Repo-mandated bounded fresh-eye subagent reviews are **already delegated** by this repo contract; this is the repo owner's explicit user delegation request for the named bounded reviewer scopes.
- When the host permits spawning, do not wait for a second user message. Task-completing `setup`, `quality`, `critique`, `release`, and GitHub `issue` resolution/closeout review runs spawn bounded reviewers immediately when the contract calls for them.
- A higher-priority system, developer, or host instruction may prohibit a spawn; this repo request cannot override it. If the host blocks subagent spawning at runtime (Agent tool absent, API-level rejection), stop and report the concrete host signal explicitly.
- Do not substitute a same-agent pass. Fresh-eye review means a different agent context; if that context cannot be obtained, leave the review unproven.
- **Spawn shape, for EVERY spawn — not only fresh-eye reviews.** Spawn one-shot subagents **without** a host addressing or team `name`. On at least one host a `name` silently routes the spawn onto a teammate protocol: the spawn succeeds, the agent runs correctly, and completion emits an idle notification instead of returning the result — findings the parent can never read, because the matching retrieval tool (`SendMessage`) is often not exposed in that session. Reserve a `name` for an agent you will address repeatedly, and only after confirming the retrieval tool exists in this session. A spawned agent is not a received result: an idle notification reads like success and is not one. If findings do not arrive, that is a delivery failure to report and to retry once unnamed, never a subagent that returned nothing and never grounds for a same-agent substitute. Full rule, upstream lineage, and non-claims: [skills/shared/references/fresh-eye-subagent-review.md](./skills/shared/references/fresh-eye-subagent-review.md).
- Bounded reviewers run in the **shared parent worktree**: inspect prior versions read-only (`git show <ref>:<path>`) and never run index- or worktree-mutating git ops (`git checkout`/`restore`/`reset`/`stash`, or `git add` of files touched only to inspect them). Staging a base reversion silently corrupts the closeout commit; the canonical rule lives in [skills/shared/references/fresh-eye-subagent-review.md](./skills/shared/references/fresh-eye-subagent-review.md).
- Bounded reviewers run read-only: on hosts with typed subagents, spawn them as `bounded-reviewer` ([.claude/agents/bounded-reviewer.md](./.claude/agents/bounded-reviewer.md)), and parents prove worktree+index integrity around each review with [skills/shared/scripts/reviewer_boundary_fingerprint.py](./skills/shared/scripts/reviewer_boundary_fingerprint.py) snapshot/verify; a failed verify quarantines that review's approvals.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [corca-ai/charness](https://github.com/corca-ai/charness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
