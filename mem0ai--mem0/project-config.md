---
trigger: always_on
description: > **Do not modify any workflow without explicit approval from a maintainer.** Publishing
---

# CI/CD and repository automation (`.github/`)

> **Do not modify any workflow without explicit approval from a maintainer.** Publishing
> credentials are bound to workflow filenames, and the gate workflows decide whether
> contributions are accepted. Read this file before proposing any change here.

## CI: one gate, many pipelines

`ci-gate.yml` (**CI Gate**) is the single entry point. It runs on every PR, detects which packages changed, and calls only the relevant package workflows as reusable workflows (`workflow_call`). Its final `CI Gate` job aggregates the results: skipped pipelines pass, failed or cancelled ones fail. It is the **only CI status check that needs to be required** in branch protection.

Package workflows keep their own push-to-main and manual triggers. Their `pull_request` triggers live in the gate's path filters instead.

| Workflow | File | Standalone triggers | Runs |
|----------|------|---------------------|------|
| CI Gate | `ci-gate.yml` | All PRs | Routes to and aggregates everything below |
| Python SDK | `ci.yml` | Push to main | Ruff + pytest on Python 3.10, 3.11, 3.12 |
| TypeScript SDK | `ts-sdk-ci.yml` | Push to main (`mem0-ts/`) | Prettier + build + jest on Node 20, 22 |
| Python CLI | `cli-python-ci.yml` | Push to main (`cli/python/`), manual | Ruff + pytest + hatch build on Python 3.10, 3.11, 3.12 |
| Node CLI | `cli-node-ci.yml` | Push to main (`cli/node/`), manual | Biome + tsc + vitest + tsup on Node 20, 22 |
| OpenClaw | `openclaw-checks.yml` | Push to main (`integrations/openclaw/`), manual | tsc + vitest (Codecov) + tsup on Node 20, 22 |
| Mem0 Plugin | `mem0-plugin-checks.yml` | Push to main (`integrations/mem0-plugin/`, excluding `.opencode-plugin/`), manual | pytest + hook exec bits + JSON manifest validation on Python 3.10, 3.11, 3.12 |
| OpenCode Plugin | `opencode-plugin-checks.yml` | Push to main (`.opencode-plugin/`), manual | Bun: tsc + build + dist artifact check |
| Pi Agent Plugin | `pi-agent-plugin-checks.yml` | Push to main (`integrations/pi-agent-plugin/`), manual | tsc + vitest + tsup on Node 20, 22 |
| n8n Node | `n8n-nodes-mem0-checks.yml` | Push to main (`integrations/n8n-nodes-mem0/`), manual | ESLint + tsc build on Node 20 |
| Zapier App | `zapier-mem0-checks.yml` | Push to main (`integrations/zapier-mem0/`), manual | tsc + `zapier validate` + offline unit tests on Node 22 |
| docs llms.txt | `docs-llms-txt-check.yml` | Manual | `docs/llms.txt` coverage |

Adding a package CI workflow: give it `workflow_call` plus `push` / `workflow_dispatch` as needed but **no `pull_request` trigger**, then register it in `ci-gate.yml` with a path filter under the `changes` job, a call job, and an entry in the gate job's `needs` list.

## Branch protection on `main`

A repository ruleset named `Main Branch Rule`, id `11813754`. It enforces squash-only merges, linear history, no deletion, no force-push, and one approving review. Two status checks belong in its `required_status_checks` rule:

| Context | Posted by | Why |
|---------|-----------|-----|
| `CI Gate` | `ci-gate.yml` | Aggregates every package pipeline |
| `license/cla` | CLA Assistant | Proves the CLA is signed, not merely requested |

Editing the ruleset requires repo **admin**. `maintain` is not enough, and the API returns 404 rather than 403 in that case. Until `license/cla` is required, the claim in `CONTRIBUTING.md` that unsigned PRs are blocked from merging holds by convention only.

Requiring `CI Gate` also means fork PRs from first-time contributors cannot merge until a maintainer approves the workflow run. Those sit at `action_required`, which is intended behavior.

## CD: one router, many publishers

`release.yml` (**Release Router**) is the only workflow listening to `release: published`. It matches the tag prefix and dispatches the matching package workflow through `workflow_dispatch`, so one release produces exactly one routed run.

| Workflow | File | Tag prefix | Target |
|----------|------|------------|--------|
| Release Router | `release.yml` | all releases | dispatches the rows below |
| Python SDK | `cd.yml` | `v*` | PyPI (`mem0ai`) |
| TypeScript SDK | `ts-sdk-cd.yml` | `ts-v*` | npm (`mem0ai`) |
| Python CLI | `cli-python-cd.yml` | `cli-v*` | PyPI (`mem0-cli`) |
| Node CLI | `cli-node-cd.yml` | `cli-node-v*` | npm (`@mem0/cli`) |
| Vercel AI SDK | `vercel-ai-cd.yml` | `vercel-ai-v*` | npm (`@mem0/vercel-ai-provider`) |
| OpenClaw | `openclaw-cd.yml` | `openclaw-v*` | npm (`@mem0/openclaw-mem0`) |
| OpenCode Plugin | `opencode-plugin-cd.yml` | `opencode-v*` | npm (`@mem0/opencode-plugin`) |
| Pi Agent Plugin | `pi-agent-plugin-cd.yml` | `pi-agent-v*` | npm (`@mem0/pi-agent-plugin`) |
| n8n Node | `n8n-nodes-mem0-cd.yml` | `n8n-nodes-mem0-v*` | npm (`@mem0/n8n-nodes-mem0`) |

- Package CD workflows are `workflow_dispatch`-only, with `tag` and `prerelease` inputs. They check out and build the given tag.
- All publishing uses **OIDC trusted publishing**. No tokens, no secrets.
- Registry trusted-publisher settings are pinned to each package's own workflow **filename**. Renaming a CD workflow breaks publishing for that package.
- First publish of a new npm package must be done manually. OIDC works from the second version onward.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mem0ai/mem0](https://github.com/mem0ai/mem0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
