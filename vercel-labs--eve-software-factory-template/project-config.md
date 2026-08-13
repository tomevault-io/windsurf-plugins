---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project overview

The eve Software Factory template: Foreman, an orchestrator agent built on the [eve](https://eve.dev) framework that turns work items into reviewed draft pull requests on the configured repository (`FACTORY_REPO`). Work arrives from GitHub (an issue labeled `factory` runs unattended; @Foreman mentions from owners/members/collaborators run attended) and from Linear Agent Sessions. The orchestrator moves every item through four declared subagent stations in order: **classifier** (triage, fast model) → **analyst** (plan + acceptance criteria, own repo checkout) → **implementer** (codes, verifies, pushes a feature branch from its own checkout) → **reviewer** (independent verdict on the pushed branch, different model vendor, max 2 revision cycles). The orchestrator then opens a draft PR; marking it ready parks on human approval, closing or reopening issues runs as reversible triage, and merging is not in the tool surface. Per-user preferences live in **Vercel Blob**, alongside a shared, per-repo **factory brain** (durable notes about the target repository) under a reserved Blob prefix, readable by every run but writable only by trusted callers. The pipeline lives in `agent/instructions.ts`.

The whole agent is defined under `agent/`. eve discovers capabilities from the filesystem. See [`ARCHITECTURE.md`](./ARCHITECTURE.md) for the component map, data flow, trust model, and boundaries.

## Setup & commands

```bash
pnpm install        # install dependencies (Node 24.x)
pnpm dev            # eve dev — local TUI; run /model once to link a model provider
pnpm typecheck      # tsc (TypeScript, no emit)
pnpm check          # ultracite (Biome) lint + format check
pnpm fix            # ultracite (Biome) auto-fix
pnpm build          # eve build
pnpm eval           # eve eval — run the evals suite (see tags below; costs real tokens)
eve deploy          # deploy to Vercel production (use this, not raw `vercel deploy`)
npx eve info        # print the discovered surface + discovery diagnostics
pnpm validate       # check + typecheck + eve info in one command
```

**Verify changes with `pnpm validate` (lint, typecheck, and discovery diagnostics must all report 0 errors / 0 warnings), then exercise the agent in the `pnpm dev` TUI.** The evals suite (`pnpm eval --tag fast`) guards routing and safety behavior; `pnpm eval pipeline/full-pipeline` runs the whole line and pushes a real branch, so run it deliberately and against a scratch repo.

## eve conventions

- **Read the relevant guide in `node_modules/eve/docs/` before writing code.** Don't invent framework APIs; confirm them against the docs.
- **Identity comes from the filesystem, never a `name` field.** A tool at `agent/tools/agent.ts` is the tool `agent`; a connection at `agent/connections/linear.ts` registers as `linear`; a subagent directory `agent/subagents/classifier/` lowers into the tool `classifier`.
- Authored slots: `agent/agent.ts` (model + session budget), `agent/instructions.ts` (`defineInstructions`, the orchestrator prompt; resolved at build time, injecting `FACTORY_REPO`), `agent/tools/*.ts` (`defineTool`), `agent/connections/*.ts`, `agent/extensions/*.ts`, `agent/channels/*.ts`, `agent/skills/<name>/SKILL.md`, `agent/subagents/<id>/agent.ts` (`defineAgent`), per-agent `sandbox.ts`.
- **Model assignments are centralized** in `agent/lib/models.ts` (the `MODELS` map). Every `agent.ts` reads its entry from there (`model: MODELS.<agent>`) instead of hardcoding a gateway id, so a model swap is a one-line edit in that file. One split is deliberate: `implementer` runs the strongest coding model on a different vendor than `reviewer`, so the review stays independent; keep those two on different vendors.
- **Extensions:** `agent/extensions/<ns>.ts` mounts a prebuilt eve extension; the filename is the namespace and its tools appear to the model as `<ns>__<tool>` (here: `github__*` from `@github-tools/eve-extension`). Config keys (`include`, `requireApproval`) use bare tool names.
- **Channels:** `github` (eve GitHub channel via Vercel Connect, botName "Foreman"; `onComment` keeps the built-in mention/ignore rules, dispatches only for OWNER/MEMBER/COLLABORATOR commenters, and stamps `attributes.trusted`; `onIssue` dispatches only on the `labeled` action for the `factory` label and rewrites the principal to the autonomous one, so the run is unattended by construction; `onCheckSuite` dispatches only for failed suites on `factory/*` pull requests, unattended, with the fix loop capped by counting attempt comments on the PR thread; `onPullRequest` posts a summary comment on opened PRs, bot senders skipped), `linear` (Agent Sessions; every session stamped trusted since workspace membership is the gate), plus the `eve` route-auth channel.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/eve-software-factory-template](https://github.com/vercel-labs/eve-software-factory-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
