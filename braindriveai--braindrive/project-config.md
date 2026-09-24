---
trigger: always_on
description: Coding agents must read this before working in this repo. Working code only. Plausibility is not correctness.
---

# BrainDrive Agent Instructions

Coding agents must read this before working in this repo. Working code only. Plausibility is not correctness.

This is the canonical root agent instruction file. Do not create or maintain a parallel root `AGENT.md`; starter-pack template `AGENT.md` files under `builds/typescript/memory/starter-pack/` are product artifacts and are separate.

## 0. Non-Negotiables

1. No flattery or filler. Start with the answer or action.
2. Never fabricate file paths, commands, test results, API names, commit hashes, or runtime behavior. Read the repo or run the command.
3. Touch only what the task requires. No drive-by refactors, broad formatting, or unrelated cleanup.
4. If there are two plausible interpretations and the choice affects code, ask before editing.
5. Do not claim done without verification. If verification cannot run, say why.

## 1. Before Editing

- State the goal and verification plan before making non-trivial edits.
- Read the files you will touch and the files that call them.
- Prefer repo-local evidence over assumptions from older conversations, external summaries, or private planning context.
- Match existing TypeScript, React, Tauri, and test patterns.
- Check the current branch and working tree before changing files.
- Treat provider configuration, owner keys, and gateway URLs as security-sensitive.

## 2. Implementation Rules

- Make the smallest diff that satisfies the request.
- Write or update focused tests before implementation where practical.
- Keep behavior scoped to the accepted spec or user request.
- Clean up only artifacts created by your own changes.
- Do not suppress errors to make tests pass. Fix the root cause.

## 3. Verification Rules

- Run focused checks during iteration and broader checks before handoff.
- Read test output; do not summarize from memory.
- For UI changes, run typecheck/build and inspect the affected surface when practical.
- For provider changes, verify no BrainDrive-owned provider keys are introduced into client config.
- If a command cannot run because dependencies or platform tooling are missing, report the blocker exactly.

## 4. Communication

- Be direct and concise.
- Surface assumptions explicitly.
- Report files changed, checks run, results, and remaining risk.
- If blocked by secrets, production/staging access, missing environment, or an unresolved product decision, stop and say exactly what is needed.

## 5. Project Context

### What This Repo Is

This is the main BrainDrive product implementation repo.

BrainDrive is a user-owned AI system built on the Personal AI Architecture. This repo contains the running product: gateway, agent loop, auth, memory, web client, MCP tool services, and Docker installer/deployment assets.

Use `README.md`, `ROADMAP.md`, and `CONTRIBUTING.md` for public product and contribution context. Do not assume access to private planning repositories, maintainer notes, or internal task trackers.

### Stack

- Main TypeScript app/runtime: `builds/typescript/`.
- Web client: React + TypeScript + Vite in `builds/typescript/client_web/`.
- Desktop shell: Tauri in `builds/typescript/src-tauri/`.
- MCP release package: TypeScript in `builds/mcp_release/`.
- Test runner: Vitest for TypeScript/web tests; Playwright exists for web E2E; Cargo tests for Tauri/Rust where relevant.

### Branching

- Primary development base branch: `dev`.
- For the OpenRouter/GLM 5.2 related work, use `model/glm-5.2`; it is based on `dev`.
- This repo is not the Managed-Hosting staging/main deploy pipeline. Do not assume pushing here deploys Hosted.

### Commands

- Common dev server:
  ```bash
  cd builds/typescript
  npm run dev
  ```
- Gateway/server only:
  ```bash
  cd builds/typescript
  npm run dev:server
  ```
- Main TypeScript build:
  ```bash
  cd builds/typescript
  npm run build
  ```
- Main TypeScript tests:
  ```bash
  cd builds/typescript
  npm run test
  ```
- Web typecheck:
  ```bash
  cd builds/typescript
  npm run web:typecheck
  ```
- Web tests:
  ```bash
  cd builds/typescript
  npm run web:test
  ```
- Web build:
  ```bash
  cd builds/typescript
  npm run web:build
  ```
- Desktop preflight:
  ```bash
  cd builds/typescript
  npm run desktop:preflight
  ```
- MCP release build:
  ```bash
  cd builds/mcp_release
  npm run build
  ```
- MCP release tests:
  ```bash
  cd builds/mcp_release
  npm run test
  ```
- Docker dev mode:
  ```bash
  ./scripts/start.sh dev
  ```

### Layout

- `README.md`: external-facing repo overview.
- `CONTRIBUTING.md`: contributor workflow and baseline checks.
- `ROADMAP.md`: public roadmap.
- `builds/typescript/adapters/openai-compatible.json`: provider profiles, including `braindrive-models`, `openrouter`, and `ollama`.
- `builds/typescript/config.ts`: runtime config surface.
- `builds/typescript/gateway/`: Fastify server routes and API behavior.
- `builds/typescript/gateway/server.ts`: main HTTP API and app wiring.
- `builds/typescript/engine/`: model loop, streaming, approvals, and tool-call flow.
- `builds/typescript/engine/loop.ts`: agent loop implementation.
- `builds/typescript/tools.ts`: built-in tool discovery and MCP registration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrainDriveAI/BrainDrive](https://github.com/BrainDriveAI/BrainDrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
