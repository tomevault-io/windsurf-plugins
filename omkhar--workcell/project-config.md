---
trigger: always_on
description: This repository exists to provide a bounded local runtime with explicit
---

# Repository Working Agreement

This repository exists to provide a bounded local runtime with explicit
isolation and policy controls for coding agents by combining the Workcell
runtime boundary with provider-specific adapters.

## Priorities

1. Developer experience
2. Simplicity
3. Security invariants
4. Performance
5. Idiomatic correctness

These priorities apply only within the defined invariant set. Do not trade away
the runtime boundary or explicit security guarantees in the name of convenience.

## Peer review default

- Treat every user request as implicitly asking for peer review unless the user
  explicitly narrows or waives that expectation.
- Peer review means continuing through review, fixes, revalidation, and another
  review pass until no actionable findings remain or a concrete blocker is
  reported.
- Treat peer review as an open-ended loop, not a single extra pass. If a peer
  or review surface returns new findings after a fix, keep iterating with that
  peer or surface until every finding is addressed, explicitly dispositioned,
  or blocked by a concrete external constraint.
- Apply that default across repo-local skills, documentation work, CI follow-up,
  publication, merge, and release actions. Do not stop at "implemented" if
  review, checks, or hosted workflows still expose actionable problems.

## Continuous improvement default

- Treat repeated friction, user correction, recurring CI churn, or manual
  workaround as a signal that the repo-local instructions should improve.
- When a task exposes a durable gap in `AGENTS.md`, a repo-local skill,
  runbook, validator, or publication workflow, capture that improvement in a
  versioned repo-local change instead of relying on conversational memory.
- Prefer explicit, reviewable skill and runbook updates over implicit habits.
- If the improvement would muddy the active review unit, cut it as a separate
  follow-on commit or PR rather than leaving the lesson unversioned.
- Do not claim the skills "learn automatically." In this repository, skill
  adaptation must happen through committed, reviewable instruction updates.

## Quality loop default

- Treat code quality and maintainability as a continuous gate, not a final
  cleanup pass.
- Before changing behavior or docs, inspect existing patterns and choose the
  smallest design that preserves the repository invariants.
- During implementation, remove speculative abstraction, vague support language,
  duplicated logic, dead code, hidden magic, and unproven claims before they
  become review burden.
- After implementation, re-review changed code, docs, policies, tests, and
  validators for simplicity, security, maintainability, and contract parity
  before moving to the next work unit.
- If validation or review exposes a recurring quality gap, fix the repo-local
  instruction, runbook, or validator in a reviewable change rather than relying
  on conversational memory.

## Mandatory rules

- Sign every commit. Do not create or rewrite commits in this repository
  without a verified signature from the maintainer identity.
- Before signing a commit that introduces or materially changes a supported
  end-to-end workflow, backend, support-tier claim, or certification-only
  validation path, run the relevant live end-to-end certification
  successfully. Do not sign "implementation first, certification later"
  commits for that class of change.
- Treat final GitHub publication as a host-side action. Prepare branch,
  signed commit message, and PR metadata inside Workcell, then use
  `workcell publish-pr` on the host rather than publishing directly from the
  Tier 1 in-container session.
- Do not treat provider config, prompt files, or rules as the sole security
  boundary.
- Preserve the dedicated VM plus container boundary as the Tier 1 design for
  all supported CLI adapters.
- Prefer explicit, auditable configuration over hidden magic.
- Mark lower-assurance modes clearly instead of overstating guarantees.
- Keep host mounts minimal. Never mount `$HOME`, host keychains, or host
  credential stores.
- Never pass through host sockets or auth state including `docker.sock`,
  `ssh-agent`, GPG agent sockets, launchd sockets, host `~/.codex`, or git
  credential-helper state.
- Keep `breakglass` paths explicit, narrow, and separately documented.
- Require explicit operator acknowledgement for `breakglass` or equivalent
  higher-trust paths.
- Treat non-git workspaces and arbitrary container commands as opt-in debugging
  paths, not the default developer flow.
- Mask repo-local provider control files and mutable git hook/config paths on
  the safe path so workspace content cannot silently take over the control
  plane.
- Ship invariant checks with new controls whenever practical.

## Pull request workflow

- For publish, PR follow-up, or merge requests in this repository, use the
  repo-local `workcell-pr-lifecycle` skill. Repo-local publication rules
  override generic GitHub publication skills.
- For `main`-based PR publication in this repository, use the host-side
  repo-local wrapper `./scripts/repo-publish-pr.sh`. It requires fresh local
  `pr-parity` evidence before delegating to the lower-level
  `./scripts/workcell publish-pr` helper.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omkhar/workcell](https://github.com/omkhar/workcell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
