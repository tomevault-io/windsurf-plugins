---
trigger: always_on
description: macOS native worktree + terminal manager for multi-agent Claude Code workflows.
---

# TBD

macOS native worktree + terminal manager for multi-agent Claude Code workflows.

Use the `tbd-project` skill for architecture, conventions, and file reference.

## ⚠️ Public repo, multi-tenant product

**This repository is public, and TBD is used by people across different organizations.** Never commit private context — real employer, org, repo, host, account, person, or ticket names, internal URLs, or machine-specific paths (use `acme` / `acme-prod` placeholders). **Whatever a session is working on elsewhere — product features, roadmaps, ticket and task descriptions — is likely confidential to that org and must not leak here**, in docs, specs, tests, fixtures, or commit messages. Never build or document for one repo, one project, or one person's workflow: features, defaults, and docs must generalize.

## Nightwatch is being replaced — two paths until cutover

The fleet-supervision subsystem ("Nightwatch"/"Daywatch") is being redesigned
from scratch. The target design is
[`docs/specs/2026-07-26-fleet-supervision-design.md`](docs/specs/2026-07-26-fleet-supervision-design.md)
(requirements: [`docs/specs/2026-07-26-fleet-supervision-requirements.md`](docs/specs/2026-07-26-fleet-supervision-requirements.md)).
The redesign builds a second path alongside the existing implementation;
cutover happens when the new path holds up, and both paths are live until
then. New supervision features should ideally land in both: in the existing
implementation where they're needed now, and in the new design's structures —
where the new-path half is usually user-land (a playbook, sweep-program, or
wake-program change) rather than compiled (see "Compile only what user-land
cannot do well" below).

## Main Session Agent

The main chat session agent should not write code directly. Delegate all implementation work to suitable subagents (Agent tool). The main session focuses on planning, coordination, and reviewing subagent results.

## Workflow

- Only stage and commit files you actually changed — never commit unrelated or other agents' modifications.
- Always commit after completing work. Don't wait to be asked.
- Use conventional commit messages: `feat:`, `fix:`, `docs:`, `refactor:`
- PR descriptions follow [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md): pick the variant that matches the change (refactors, DX, and ops use the feature skeleton, with Summary carrying the motivation) and replace the guidance comments with prose.
- A PR (or GitLab MR) opened from a TBD worktree ends its body with that worktree's deep-link, so the PR can be traced back to the session that produced it: get the UUID with `tbd link 2>/dev/null || tbd link "$(basename "$(git rev-parse --show-toplevel)")"`, take the `?worktree=<uuid>` portion, and make the last line `[<worktree display name>](https://cheapsteak.github.io/tbd/open/?worktree=<uuid>)`. Outside TBD, or when `tbd link` fails, skip it silently — it is never a blocker. (A guardrail rule, `pr-worktree-link`, nudges about this on `gh pr create` and `glab mr create`.)
- Banned words — never use these in code or in PR titles, descriptions, or commit messages: "blessed", "golden"
- No prose tables in markdown docs — use lists (bolded lead term, en-dash, prose; nested bullets when a row has several fields). A table is acceptable only when its cells are mostly short numerical or scannable values: counts, defaults, thresholds, old→new line numbers. Prose crammed into cells gets squished and unreadable.
- Spec and doc edits must leave a document that stands alone for a first-time reader. Never write revision history into prose: no "Amended \<date\>" notes, no "an earlier draft…" retracing, no reversal narratives — rewrite the superseded passage to state the current design as if it were always so; git history is the archive. Keep **evidence** ("field measurement showed X"), drop **chronology** ("removed \<date\> after review Y"). A rejected-alternatives section is welcome as timeless why-not rationale — written as rationale, never as a revision log. The one exemption is an **as-built audit record** of an existing system (e.g. [`docs/nightwatch.md`](docs/nightwatch.md)): there, dated banners recording what was measured against which tree are the evidence, and the document must declare that purpose at the top.
- Verify your changes compile (`scripts/swift-safe build`) before committing.
- **`swift package resolve` can print `error:` on a run that succeeded.** SwiftPM removes a dependency checkout by unlinking it in place; a removal that doesn't complete leaves a tree the next resolve reads as "has uncommitted changes" and refuses to finish removing — gone from `Package.resolved`, still on disk, `error:` on stderr, **exit 0**. So an `error:` line in resolve output is not by itself evidence that the resolve failed: check the exit code, and to confirm the workspace really is clean compare `.build/checkouts` against the `identity` values in `Package.resolved` — they should match exactly (case-insensitively).
- Run `scripts/test.sh` if you changed daemon or shared code. It fences the run
  against the developer's real `~/tbd` and `~/.claude` (see "Tests must not
  touch ~/tbd") and invokes SwiftPM through `scripts/swift-safe`, which

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cheapsteak/tbd](https://github.com/cheapsteak/tbd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
