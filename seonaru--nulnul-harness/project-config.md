---
trigger: always_on
description: This repository builds `plugins/nulnul-harness/`, a Codex plugin that finds proven capabilities, assembles the smallest useful project-local agent system, completes the user's work, and evolves it from measured outcomes without requiring users to operate a harness.
---

# nulnul harness working agreement

This repository builds `plugins/nulnul-harness/`, a Codex plugin that finds proven capabilities, assembles the smallest useful project-local agent system, completes the user's work, and evolves it from measured outcomes without requiring users to operate a harness.

- Treat `plugins/nulnul-harness/` as the only shipped product boundary.
- Keep the plugin skills-only until a real workflow proves that an MCP server, hook, app, or external service is necessary.
- Inspect a target repository before asking questions. Ask only for product decisions or constraints that cannot be discovered safely. A request to set the harness up on a repository that already has work is never one of those questions.
- Detect the host surface before writing setup files, and enumerate its installed skills, plugins, and agents before claiming a job is covered; on Claude Code adoption, first run the bounded `claude plugin list --json` command instead of inferring installed plugins from the session catalog. Treat Claude's `source=git` as public GitHub provenance only when the exact repository URL also matches.
- Give each host its own root session entry: Codex owns only `AGENTS.md`, Claude Code owns only `CLAUDE.md`, and both point to the same `docs/nulnul/` contract and exactly one live-state writer. On sequential host adoption, preserve the inactive entry byte-for-byte; do not claim concurrent mutation support.
- Upgrade an existing agent roster in place. Classify every existing role as kept, upgraded, merged, or removed; never recreate one that already exists.
- Search installed, official, curated, and reputable public capabilities before creating a project-local substitute. Verify fit, provenance, compatibility, maintenance, permissions, and license; popularity alone is not verification.
- Continue the user's original task after setup; setup alone is not task completion.
- Add every necessary, non-overlapping capability, but activate only what the current task needs.
- Prefer direct or single-agent execution. Add roles only from concrete independent work or verification boundaries, with one synthesis owner.
- Never register global tools, use credentials, deploy, or publish without explicit user approval.
- Keep generated setup removable. Accept an evolution only when a reproducible before/after check improves the primary outcome without violating guardrails, and observe one live cycle after promotion with an executable automatic rollback threshold; schema-v3/v4 states must run the shipped rollback executor before final validation.
- Keep ordinary resume context bounded: compact closed evolution history into the digest-bound adjacent archive, validate deterministic full-state reconstruction, and query rejected history only when a matching proposal needs it.
- Give durable projects one validated concise resume checkpoint; keep stable setup evidence outside the host-loaded entry, and convert every reproducible nonpass verdict into Coach feedback and one bounded proposal in the same run.
- Allow fast resume only from an explicitly verified checkpoint; machine-link every nonpass verdict to its feedback and proposal, and migrate legacy durable contracts without creating a second live-state writer.
- Version concise checkpoint shapes explicitly, fail release on a missing learning-verdict inventory, and restore all earlier project files when a migration replacement fails.
- Give every state file one writing process, keep `unknown` distinct from `verified` and `failed`, persist cursors on empty cycles, and prove each validity check against a negative control.
- Store checkpoint completion as an exact command, execute that field before verified fast resume, and require sanitized machine-valid evidence before a paid runtime result contributes Release Gate points.
- Fail release on a recorded setup, workflow, activation, or fast-path regression; use version-independent champion/candidate evidence, counterbalance paired order, prefer relative budgets to absolute token ceilings, and keep fast resume inside its checkpoint and directly needed task files.
- Before pushing a version-changing commit to `main`, require exact-version public Claude adoption evidence and, when personal or cross-project reuse changes, sanitized apply/skip/revocation or Meta adoption evidence; otherwise use a non-main publication candidate first. Never knowingly leave `main` red while calling the release work complete. After any approved push, watch the resulting CI to green before reporting completion.
- Treat evaluation exposure as state: preregister a frozen candidate before one-shot holdout use, retire every used holdout, reject leakage or recycling, compare a simple retry/selection baseline, and activate Generalization Gate only for personal/core transfer claims rather than ordinary project-local changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeoNaRu/nulnul-harness](https://github.com/SeoNaRu/nulnul-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
