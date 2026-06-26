---
trigger: always_on
description: > Universal onboarding document for any AI agent (Claude Code, Codex, Gemini CLI, Cursor, etc.)
---

# AGENTS.md — pallete-maker

> Universal onboarding document for any AI agent (Claude Code, Codex, Gemini CLI, Cursor, etc.)

## What Is pallete-maker?

**pallete-maker** is a personal color palette creator. It lets a user pick a
base color, build a harmonious palette of up to **15 colors (12 chromatic + 3
achromatic; constants `MAX_TOTAL`, `MAX_CHROMATIC` in `src/scripts/harmony.mjs`)**
using internal group-based harmony rules, preview the result on a grid, and
export it as a PNG image.

**Current implementation:** static modular HTML/CSS/JS app
**Core dependencies:** html2canvas 1.4.1 (PNG export); harmony logic is local in `src/scripts/harmony.mjs`
**Deploy target:** Vercel via Git integration
**Owner:** personal project, single user

## Current Phase & Status

| Area                                      | Status                              |
| ----------------------------------------- | ----------------------------------- |
| Product prototype                         | COMPLETE                            |
| Static palette grid + export              | COMPLETE                            |
| Mobile adaptation                         | PARTIAL — ongoing iteration         |
| Frontend architecture cleanup             | UPCOMING                            |
| Repository memory and feature-memory flow | COMPLETE                            |
| CI / AI review orchestration              | COMPLETE                            |
| Production deploy flow                    | COMPLETE via Vercel Git integration |

## Project Structure

```
pallete-maker/
├── .specify/
│   └── memory/constitution.md          # Process contract and non-negotiable rules
├── specs/
│   └── <feature-id>/                   # Feature memory: spec.md, plan.md, tasks.md
├── index.html                          # Current app shell
├── package.json                        # Repo tooling for CI, local orchestration, and build
├── vercel.json                         # Vercel build/output configuration
├── scripts/
│   ├── build-static.mjs                # Static build to dist/
│   ├── check-static-baseline.mjs       # Repository baseline checks
│   ├── check-feature-memory.mjs        # Product change -> complete specs folder enforcement
│   ├── set-implementation-agent.mjs    # Local + GitHub agent policy helper
│   ├── new-worktree.mjs                # macOS local worktree helper
│   ├── start-implementation-worker.mjs # Prompt preparation helper
│   ├── publish-branch.mjs              # Push branch and open or reuse PR
│   ├── resolve-pr-context.mjs          # Pull request context resolver for workflows
│   ├── ai-command-policy.mjs           # Trusted AI command validation + review rerun marker
│   ├── ai-review-gate.mjs              # Review gate for Codex/Claude/Gemini
│   ├── ai-review-helpers.mjs           # Shared review evidence helpers
│   ├── ai-review-rerun.mjs             # Event-driven AI Review rerun helper
│   └── switch-review-agent.mjs         # One-shot review backend switcher (posts human trigger comment for all three agents)
├── docs_pallete_maker/
│   ├── README.md                       # Durable docs index
│   ├── adr/                            # Architecture decision records
│   ├── project-idea.md                 # Product overview and roadmap
│   └── project/
│       ├── frontend/frontend-docs.md   # Frontend architecture notes
│       └── devops/                     # CI/CD and orchestration contract
└── .github/workflows/                  # CI, guard, AI review, Claude, deploy policy
```

## Delivery Workflow

- All code changes land through pull requests.
- Product-code work starts from an active `specs/<feature-id>/` folder.
- One implementation loop uses one worktree, one branch, and one PR.
- Required GitHub checks are `baseline-checks`, `guard`, and `AI Review`.
- Vercel handles preview deployments for pull requests and production deployment for `main` through Git integration.
- Durable workflow docs live under `docs_pallete_maker/project/devops/`.
- Optional Unicorn Hub adaptation metadata lives in `.unicorn-hub/config.json`;
  it describes local docs/specs/product paths but does not replace
  `docs_pallete_maker/` or the app-specific baseline checker.
- Local orchestration state lives under `.claude/` and is gitignored.
- Local worktrees are created inside `<repoRoot>/.claude/worktrees/<slug>/` so they stay inside the repository.
- Agent selection is policy-driven through repository variables:
  - `AI_IMPLEMENTATION_AGENT`
  - `AI_REVIEW_AGENT`
- Default policy for this repository is:
  - implementation: `claude`
  - review: `codex` (switched from `gemini` on 2026-04-17; see `docs_pallete_maker/project/devops/ai-orchestration-protocol.md` for the canonical description)
- Claude is the default implementation agent because it owns architecture, orchestration, CI/CD health, and repository memory, and is driven from the user's local Claude Code terminal session.
- Codex is the current default review backend via `@codex review` triggers on PR comments.
- `AI Command Policy` records trusted review-request markers and
  `AI Review Rerun` reruns the required check when trusted reviewer evidence
  appears.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kiaquila/pallete-maker](https://github.com/kiaquila/pallete-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
