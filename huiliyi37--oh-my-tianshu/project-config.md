---
trigger: always_on
description: Tianshu Harness SDK is a plugin-based agent harness on vendored Cordis: **everything is a plugin**. Read [docs/architecture.md](docs/architecture.md) before changing `packages/`; follow [docs/AGENTS.md](docs/AGENTS.md) for documentation.
---

# AGENTS.md

Tianshu Harness SDK is a plugin-based agent harness on vendored Cordis: **everything is a plugin**. Read [docs/architecture.md](docs/architecture.md) before changing `packages/`; follow [docs/AGENTS.md](docs/AGENTS.md) for documentation.

## Pre-release stance: foundation over blast radius

**Remove this section at the first tagged release.** With no external consumers, prefer the correct foundation over compatibility shims: rename or repackage freely and update every reference together. Backends reject old on-disk formats. SQLite uses monotonic `SCHEMA_VERSION`; `dsh-session` keeps `SESSION_FORMAT_VERSION` at `0` with no compatibility promise.

## Repository layout

Directory tree and per-group purpose moved to [docs/development.md](docs/development.md#repository-layout); package groups are indexed in packages/README.md.

## Commands

Daily command inventory moved to [docs/development.md](docs/development.md#daily-commands); script and gate inventories live in `package.json` and scripts/run-gates.ts.


### Host sandbox failures

When required `gh`, `pnpm`, build, test, or generator commands fail because the agent sandbox blocks credentials, network, IPC, file watching, or nested `sandbox-exec`, retry unchanged with the narrowest host escalation before diagnosing authentication or project failure. Require sandbox evidence; never bypass genuine test failures or the product sandbox under test.

### Run relevant checks locally

Run checks before pushes via [dsh-pre-push-checks](.agents/skills/dsh-pre-push-checks/SKILL.md); report only commands run. After `gh stack sync`, validate immediately; do not merge before checks pass.

- Match evidence to the surface: focused tests for behavior, snapshots for model or user output, `doc-sync` for docs, build/hygiene and built smokes for published paths, and real-API e2e for provider behavior.
- Never default to the full suite or repeat a passing check for commit or push. CI owns exhaustive coverage and the platform matrix; rehearse all locally only by explicit request, for CI diagnosis, or for an irreducibly repository-wide change.
- `test:coverage`, not `test`, is the CI coverage gate ([why](docs/testing.md)).

## High-risk reminders（高危提醒）

Push permission is strictly controlled: **never push anything to any remote by default.** Local commits are the default end state; pushing requires an explicit, current request from the user in this session.

When the user personally asks to push, do not push until you have verified **both sides of the push**:

- **Local side** — confirm the working tree and branch state you intend to publish: no unintended commits, no stray or secret files staged.
- **Remote side** — confirm the target remote's identity (`git remote -v`) and its **visibility: the repository must be private, never public.** Establish visibility from the remote host (e.g. GitHub repo page, API, or `gh repo view`), never from local assumptions.
- Confirm the local repository and the target remote are the **same repository** (matching owner/name) so nothing is pushed to an unrelated or wrong remote.

**Never push before establishing the repository's nature.** If visibility cannot be confirmed, do not push — stop and ask the user. A push to a public repository is irreversible exposure; treat it as a critical incident.

## High-risk reminders（高危提醒）

Push permission is strictly controlled: **never push anything to any remote by default.** Local commits are the default end state; pushing requires an explicit, current request from the user in this session.

When the user personally asks to push, do not push until you have verified **both sides of the push**:

- **Local side** — confirm the working tree and branch state you intend to publish: no unintended commits, no stray or secret files staged.
- **Remote side** — confirm the target remote's identity (`git remote -v`) and its **visibility: the repository must be private, never public.** Establish visibility from the remote host (e.g. GitHub repo page, API, or `gh repo view`), never from local assumptions.
- Confirm the local repository and the target remote are the **same repository** (matching owner/name) so nothing is pushed to an unrelated or wrong remote.

**Never push before establishing the repository's nature.** If visibility cannot be confirmed, do not push — stop and ask the user. A push to a public repository is irreversible exposure; treat it as a critical incident.

## Secrets / .env

Real-API tests and demos read `DEEPSEEK_API_KEY`, optional `DEEPSEEK_BASE_URL`, and root `.env`. cordis.yml allows `!!js` (never `!js`) only under plugin `config`; Loader metadata is static, so conditional composition uses overlays ([primer](docs/cordis-primer.md#loader-configuration)). Never commit credentials. CI e2e skips without a key; [testing.md](docs/testing.md) owns key policy.

## Conventions

- Every npm package is `@huiliyi37/dsh-<name>`; vendored packages keep upstream names and are `private: true`. `cordis` is a peerDependency (+ dev) of every harness package.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huiliyi37/oh-my-tianshu](https://github.com/huiliyi37/oh-my-tianshu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
