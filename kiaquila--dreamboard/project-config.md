---
trigger: always_on
description: > Universal onboarding document for any AI agent (Claude Code, Codex, Gemini CLI, Cursor, etc.)
---

# AGENTS.md — dreamboard

> Universal onboarding document for any AI agent (Claude Code, Codex, Gemini CLI, Cursor, etc.)

## What Is dreamboard?

**dreamboard** is a personal visual goal-board creator. It combines a marketing landing page and a canvas-based editor where a user can place images and text, style objects, and export the result as a composed dream board.

**Current implementation:** static single-file web app  
**Core editor dependency:** Fabric.js  
**Deploy target:** Vercel via Git integration  
**Owner:** personal project, single user

## Current Phase & Status

| Area                                      | Status                              |
| ----------------------------------------- | ----------------------------------- |
| Product prototype                         | COMPLETE                            |
| Static landing + editor                   | COMPLETE                            |
| Mobile adaptation                         | PARTIAL — ongoing iteration         |
| Frontend architecture cleanup             | UPCOMING                            |
| Repository memory and feature-memory flow | IN PROGRESS                         |
| CI / AI review orchestration              | COMPLETE                            |
| Production deploy flow                    | COMPLETE via Vercel Git integration |

## Project Structure

```
dreamboard/
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
│   └── ai-review-gate.mjs              # Review gate for Codex/Claude
├── docs_dreamboard/
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
- Durable workflow docs live under `docs_dreamboard/project/devops/`.
- Local orchestration state lives under `.claude/` and is gitignored.
- Local worktrees are created inside `<repoRoot>/.claude/worktrees/<slug>/` so they stay inside the repository.
- Agent selection is policy-driven through repository variables:
  - `AI_IMPLEMENTATION_AGENT`
  - `AI_REVIEW_AGENT`
- Default policy for this repository is:
  - implementation: `claude`
  - review: `codex`
- Claude is the default implementation agent because it owns architecture, orchestration, CI/CD health, and repository memory, and is driven from the user's local Claude Code terminal session.
- Codex is the default review backend because it runs natively on GitHub pull requests and emits `P0`–`P3` inline severity markers.
- Gemini stays available as the fallback review backend when `AI_REVIEW_AGENT=gemini`. Claude review is a third-tier option behind an explicit `AI_REVIEW_AGENT=claude` override and still requires `ANTHROPIC_API_KEY` when used.
- Trusted human review commands still follow the shared `AI Review` contract, but only `claude` uses `workflow_dispatch`; `gemini` and `codex` stay native-only so they do not cancel the PR-linked gate.
- Only trusted repository actors may trigger AI workflows.
- Trusted actors are `OWNER`, `MEMBER`, and `COLLABORATOR`.

## Review Guidelines

- Codex review uses native GitHub PR review output plus `P0-P3` inline severity badges.
- Claude review uses a top-level `claude[bot]` comment with marker lines, not a formal GitHub PR review.
- Gemini review uses native GitHub PR review output from `gemini-code-assist[bot]` plus inline severity markers such as `Critical`, `High`, `Medium`, and `Low`.
- When a Claude review request includes `AI_REVIEW_AGENT`, `AI_REVIEW_SHA`, and `AI_REVIEW_OUTCOME`, preserve those lines exactly at the start of the final top-level Claude comment.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kiaquila/dreamboard](https://github.com/kiaquila/dreamboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
