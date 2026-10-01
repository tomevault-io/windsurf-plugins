---
trigger: always_on
description: Development skills live in `.codex/skills` (aliased by `.claude/skills`) and
---

# Comma Agent Instructions

Development skills live in `.codex/skills` (aliased by `.claude/skills`) and
`systems/.claude/skills`. Product Agent skills under
`resources/salix-system-files/skills` do not govern repository development.

## Domain concepts

Read [the domain concept inventory](docs/architecture/DOMAIN_CONCEPTS.md) before adding or changing a domain entity.
Do not add an entity unless an existing concept cannot represent the required behavior.
Prefer existing entities, relationships, values, or explicit projections over duplicate identities and lifecycle authorities.
When a new entity is necessary, update the inventory in the same change.
Record why reuse is insufficient, its identity and scope, its authoritative owner, its lifecycle, and its relationships.
Include implementation links. Update affected entries when renaming, merging, retiring, or changing ownership of existing concepts.
A new page, provider, transport, DTO, or storage layout alone does not justify a new domain entity.

## Deployment source

Deploy staging only after the PR is merged into `main`, using the image and
chart built and published from that mainline commit. Production uses the
existing staging-proven mainline promotion flow. Never deploy an unmerged
feature/PR branch artifact, including through an image-tag override, and never
deploy a branch first with a plan to merge it afterward. Manually selected
rollback artifacts must also come from the approved mainline release flow.
Existing failed-transaction recovery may restore the pre-release serving
snapshot before an incompatible cutover; it does not authorize choosing that
snapshot as a later release candidate. An environment still serving a
historical branch artifact is not reconciled until a mainline release commits
successfully. Merge authorization, CI/review gates, stored-data safety, and
post-rollout convergence still apply.

## Salix

For Salix conversations, participants, messages, or delivery, use
`docs/README.md` to find the contract for the changed behavior. Mutate only through
`ConversationServer -> ConversationActor`. Participant mutation is separate
from message append; participants are delivery targets, not senders. Never
persist the in-memory `participant_count` aggregate.

## Release convergence and migrations

Rollouts and provider migrations may interrupt service between phases. Preserve
owned data. After cutover, restore service within the documented release or
provider budget. A failed dependency may instead return a bounded, actionable
error. Do not add a rolling-window reader, dual write, fallback, adapter,
shadow state, or extra rollout solely for intermediate availability. Transform
durable facts once, rebuild only owner-approved disposable state, or fail closed
until the completed release reconciles the affected path.

For example, if the Sprites Device disconnects during a Group migration's
`preparing` phase, read the saved operation and source seal. Retry that phase
or cancel it when the source is reachable. Do not add another carrier to keep
the Group online during `preparing`. The migration is complete only when the
Cloudflare Device connects, retained Sessions and input resume, and the archive
hold clears after a durability check. A committed provider field is insufficient.

Graceful rolling updates are the default.
The authoritative policy is `docs/release-operations.md#rollout-policy-and-human-shutdown-approval`.
Staging scale-to-zero or deployment-wide shutdown requires a human issue comment
under that policy. The issue title and body must say
`HUMAN APPROVAL REQUIRED - AGENTS MUST NOT APPROVE`.
Agents must never approve, impersonate a human approver, or bypass this requirement
through direct cluster commands. Temporary mixed-version unavailability alone
is not a reason to choose shutdown.

An incompatible schema cutover may use the existing `exclusive` release mode
only with the required human approval.
The PR must identify the durable facts that survive, the exact disposable scope,
the post-cutover convergence condition and budget, and whether recovery is
pre-cutover rollback or post-cutover forward repair. Do not add compatibility
state merely to make an old binary a post-cutover rollback target. Backups and
restore evidence are required only for durable data that the cutover can alter;
they are not required for an owner-approved rebuild of disposable projections.

Destructive scope must be resolved with a bounded read before execution. Never
infer that a cache, VM disk, workload volume, credential store, or projection is
disposable: the owning product decision must name it. Preserve unrelated data,
use only approved mainline artifacts, and report any required re-enrollment or
reauthentication as part of the convergence result.

## Documentation style

Use the project `ste-writing` skill for documentation, READMEs, runbooks, release
notes, and pull request text. Use STE-flavored mode by default. Use strict mode
for procedures and safety text. Do not use this default when the user requests
detailed writing or a different style.

## Documentation scope and budget

Keep general documentation under `docs/` to at most 20 tracked files, recursively.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AFK-surf/Comma](https://github.com/AFK-surf/Comma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
