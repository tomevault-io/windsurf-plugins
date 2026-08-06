---
trigger: always_on
description: This file applies to the entire repository. It is the repository-specific instruction source for coding agents and automated contributors.
---

# AGENTS.md

## Scope and instruction design

This file applies to the entire repository. It is the repository-specific instruction source for coding agents and automated contributors.

Keep this file durable, small, and focused on rules that should apply every time work happens in this repository. Do not add one-off task plans, temporary investigation notes, local machine details, or long procedure transcripts here. Move detailed operating procedures to `docs/`, reusable workflows to skills, and directory-specific rules to a closer nested `AGENTS.md` if that scope ever appears.

Codex loads project guidance from the repository root down to the working directory, and closer files override broader guidance. Codex also has a default project-instruction size budget, so this file should stay compact enough to load completely. If a new rule would make this file sprawl, prefer replacing repeated guidance with a pointer to the authoritative script, test, or document.

`branch-deploy` is a public GitHub Action for IssueOps-based branch deployments. Treat every source file, test, generated artifact, branch name, commit, pull request, comment, workflow log, release artifact, example, and fixture as public information.

The project prioritizes behavior preservation, a small dependency surface, strict static guarantees, reproducible committed bundles, and reviewable changes. Prefer the smallest change that completely solves the requested problem.

## Public-repository safety

- Never add credentials, tokens, cookies, private keys, authentication headers, customer data, private repository names, private URLs, internal hostnames, non-public infrastructure details, local registry or proxy configuration, or machine-specific identifiers.
- Never copy private material from another checkout, conversation, clipboard, log, browser session, or tool output into this repository.
- Do not commit absolute local paths. Pay particular attention to source maps, coverage output, archives, manifests, copied command output, and generated files.
- Keep fixtures, examples, branch names, commit messages, pull request text, comments, workflow summaries, and documentation generic and suitable for a public open-source repository.
- Before every commit, push, pull request, or public comment, review the relevant diff, staged content, untracked files, generated artifacts, commit metadata, branch history, PR title, and PR body for accidental disclosure.
- If a requested change appears to require non-public context, stop before publication and ask for explicit direction.

## Repository and contribution boundary

- Base new work on the repository's current default branch unless the maintainer specifies another base.
- Inspect `git status --short --branch` and the actual diff before editing, staging, committing, or opening a pull request.
- Preserve unrelated working-tree changes. Never discard, overwrite, amend, rebase, or force-push user or upstream work unless explicitly requested.
- Do not merge a pull request, publish a release, create or move tags, change repository settings, or bump the action version unless the current request explicitly authorizes that operation.
- Keep unrelated refactors, dependency upgrades, release bumps, base synchronization, workflow hardening, and behavior changes in separate pull requests.
- Make the intended head, base, public compatibility impact, and generated-artifact impact clear in PR work.

## Product and architecture model

The supported public product is the combination of `action.yml` and the committed JavaScript bundle under `dist/`. The TypeScript source tree is an implementation detail, not a supported package import surface.

This repository is intentionally an action-only package:

- `package.json` is private from npm's perspective.
- There is no supported npm library API.
- There are no emitted declarations, `lib/` build tree, `exports`, `types`, or source-import compatibility promise.
- Consumers are expected to reference the GitHub Action, not import `src/*.ts`.

`action.yml` declares the GitHub-hosted Node action runtime and uses `dist/index.js` for both the main and post entrypoints. GitHub Actions executes the committed bundle directly; consumer workflows do not install dependencies or compile TypeScript when invoking the action.

The package boundary is ESM. Preserve `"type": "module"`, ESM exports, import-time behavior, and the main/post lifecycle unless the maintainer explicitly authorizes a public runtime change.

## Important files

- `action.yml` defines the public action inputs, outputs, runtime, main entrypoint, and post entrypoint.
- `src/main.ts` contains import-time dispatch and the primary exported `run` entrypoint.
- `src/actions-core.ts` is the project-owned compatibility layer for the narrow GitHub Actions runner-command surface consumed by this action.
- `src/action-io.ts` centralizes typed input, output, and action-state keys.
- `src/trust-boundaries.ts` contains intentionally narrow assertions and legacy coercion boundaries that cannot be proven statically without changing runtime behavior.
- `src/types.ts` and related modules define shared domain models and discriminated unions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [github/branch-deploy](https://github.com/github/branch-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
