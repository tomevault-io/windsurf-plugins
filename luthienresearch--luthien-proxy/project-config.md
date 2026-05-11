---
trigger: always_on
description: > This file is the canonical agent-instructions document. `CLAUDE.md` is a symlink to it. CI enforces that every `AGENTS.md` has a matching `CLAUDE.md` symlink — edit `AGENTS.md` only.
---

# Repository Guidelines

> This file is the canonical agent-instructions document. `CLAUDE.md` is a symlink to it. CI enforces that every `AGENTS.md` has a matching `CLAUDE.md` symlink — edit `AGENTS.md` only.

## Purpose & Scope

- Core goal: implement AI Control for LLMs with integrated gateway architecture.
- Architecture: FastAPI gateway with integrated control plane, using event-driven policies. LiteLLM handles judge/LLM policy calls.
- **Read `ARCHITECTURE.md` first** when you need to understand how modules connect, how requests flow, or where to make changes. It covers the request lifecycle, module map, key abstractions, and data model.
- Select a policy via `POLICY_CONFIG` that points to a YAML file (defaults to `config/policy_config.yaml`).
  - Example: `export POLICY_CONFIG=./config/policy_config.yaml`

## Sibling Repos

- **luthien-org** (`~/build/luthien-org`): Org-level repo for feedback synthesis, requirements, planning docs, and UI mockups. Lives at `LuthienResearch/luthien-org` on GitHub.
- Cross-repo work (e.g. updating feedback synthesis docs alongside a README PR) should be tracked in the luthien-proxy PR description, not as separate PRs in luthien-org. Commit directly to luthien-org main and link from the luthien-proxy PR.
- Key paths: `ui-fb-dev/1-feedback-synthesis/` (user interview takeaways, value-prop feedback), `ui-fb-dev/2-requirements/` (live requirements docs).
- **Design system:** `ui-fb-dev/design-system.md` — tone, copy constraints, visual specs, trust signals. Load this when editing any user-facing page (landing page, README, UI).

## Development Workflow

1. You will be given an OBJECTIVE to complete (e.g. 'implement this feature', 'fix this bug', 'build this UX', 'refactor this module')
2. Claim the corresponding ticket on the [Trello board](https://trello.com/b/ehoxykPf/luthien?filter=label:luthien-proxy%20TODO) (assign yourself, move to In Progress)
3. Make sure we're on a feature branch (not `main`)
4. Commit the changes, push to origin, and open a draft PR (to `main`)
5. Implement the OBJECTIVE. Add any items that should be done but are out of scope for the current OBJECTIVE to the [Trello board](https://trello.com/b/ehoxykPf/luthien?filter=label:luthien-proxy%20TODO) (e.g. noticing an implementation bug, incorrect documentation, or code that should be refactored).
6. Regularly run `scripts/dev_checks.sh`, then commit and push any formatting/lint fixes along with your changes to origin (on the feature branch).
7. When the OBJECTIVE is complete, add a changelog fragment to `changelog.d/` (see `changelog.d/README.md`)
8. Mark the PR as ready.
9. Mark the Trello ticket as done.

### One PR = One Concern

- **Bug fix discovered while building a feature?** Separate PR.
- **Infrastructure change needed for a feature?** Separate PR, feature depends on it.
- **Ask:** "Could these be reviewed/merged independently?" If yes, split them.

This keeps PRs focused, easier to review, and allows independent merging. **Bug fixes bundled into feature PRs bypass the COE process** — no root cause analysis, no "what else could break" sweep. Always split bug fixes into their own PR so they get a COE. (Example: PR #133 bundled a probe-filtering fix into a feature, skipping COE → same class of bug recurred 38 days later in PR #277.)

### Maintaining Context

Proactively update files in `dev/context/` as you learn about the codebase:

- `codebase_learnings.md`: When you discover architectural patterns, module relationships, or how subsystems work together
- `decisions.md`: When a technical decision is made (e.g., "why we use X instead of Y", "why this API is structured this way")
- `gotchas.md`: When you encounter non-obvious behavior, edge cases, or common mistakes

These files persist across sessions and help build institutional knowledge. Update them during development, not just at the end. Include timestamps (YYYY-MM-DD) when adding entries to detect when knowledge may be stale.

**Reliability warning**: Files in `dev/context/` are written by both humans and AI agents. Agent-written content may contain inferences presented as facts. Before relying on a claim in these docs (especially around how authentication, streaming, or edge cases work), verify against the actual code. If you discover incorrect information, fix it immediately rather than working around it.

Note that both Claude Code and Codex agents work in this repo and may read from and write to context.

### Objective Workflow

1. **Start a new objective**

   ```bash
   git checkout main
   git pull
   git checkout -b <short-handle>
   ```

   - If the branch already exists, run `git checkout <short-handle>` instead of creating it.
   - Create the scratch dir if it doesn't exist, then write the objective statement into `dev/scratch/OBJECTIVE.md` (gitignored — scoped to this worktree, never merged to main). Use it to stay oriented across compactions.

     ```bash
     mkdir -p dev/scratch
     # then write dev/scratch/OBJECTIVE.md
     ```

   - Mark the branch start with an empty commit; the commit message is the objective and feeds the PR body via `--fill`:

     ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuthienResearch/luthien-proxy](https://github.com/LuthienResearch/luthien-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
