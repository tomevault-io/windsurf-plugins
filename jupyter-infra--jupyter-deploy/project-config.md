---
trigger: always_on
description: | Workflow | Trigger | Purpose |
---

# .github — CI Workflows

## Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `ci.yml` | push/PR | Lint + unit tests |
| `lint.yml`, `test.yml` | `workflow_call` | Reusable lint/test jobs |
| `release-cli.yml` | `workflow_dispatch` | Release `jupyter-deploy` CLI to PyPI — pre-publish smoke gate (local wheel) → Test PyPI → E2E gate → PyPI |
| `release-base.yml` | `workflow_dispatch` | Release `jupyter-deploy-tf-aws-ec2-base` to PyPI (with E2E gate) |
| `release-plugin.yml` | `workflow_dispatch` | Release `pytest-jupyter-deploy` to PyPI |
| `e2e-cli.yml` | `workflow_call` | CLI release E2E gate — smoke tests (bare/aws/aws-k8s) + functional tests against base app #2 and EKS app #5 |
| `release-eks-oidc.yml` | `workflow_dispatch` | Release `jupyter-deploy-tf-aws-eks-oidc` to PyPI (with E2E gate) |
| `e2e-base.yml` | `workflow_dispatch` | E2E tests against an existing deployment |
| `e2e-base-fresh.yml` | `workflow_dispatch` / `workflow_call` | Deploy from scratch + full E2E chain |
| `e2e-base-release.yml` | `workflow_call` | Base template release E2E gate — calls fresh workflow with Test PyPI install |
| `e2e-base-canary.yml` | `schedule` / `workflow_dispatch` | Weekly canary — calls fresh workflow |
| `e2e-base-job.yml` | `workflow_call` | Reusable base E2E job (called by the above) |
| `e2e-eks-oidc.yml` | `workflow_dispatch` | EKS E2E tests against an existing deployment |
| `e2e-eks-oidc-fresh.yml` | `workflow_dispatch` / `workflow_call` | Deploy EKS from scratch (in-container, so pypi-mode deploys the published package) + full E2E chain |
| `e2e-eks-oidc-release.yml` | `workflow_call` | EKS template release E2E gate — calls fresh workflow with Test PyPI install |
| `e2e-eks-oidc-canary.yml` | `schedule` / `workflow_dispatch` | Weekly canary — calls fresh workflow |
| `e2e-eks-oidc-job.yml` | `workflow_call` | Reusable EKS E2E job (called by the above) |
| `e2e-build-image.yml` | `workflow_call` | Reusable build-and-push E2E image to ECR (`TEMPLATE` build-arg selects base vs eks-oidc for pypi-mode installs) |

## OAuth app slots

Fresh deploys consume a Let's Encrypt cert (limit: 5/subdomain/week), so canary and
release gates use dedicated app slots to avoid contention:

| Template | Manual/PR fresh | Release gate | Canary |
|----------|-----------------|--------------|--------|
| base | 1 | 2 | 3 |
| eks-oidc | 4 | 5 | 6 |

## Release-mode vs canary-mode deploys

Fresh-deploy workflows build the E2E image and deploy **from inside it**, so the install
mode determines what actually gets deployed:

- **workspace** (default, PR/manual) — local source via `uv sync --all-packages`.
- **pypi + release** (release gate) — the package under test pinned from Test PyPI
  (`pkg-version`), the rest from prod PyPI. Renders `.github/e2e-<template>/pyproject.release.toml`.
- **pypi + canary** (scheduled canary) — everything from prod PyPI, unpinned. Uses
  `.github/e2e-<template>/pyproject.canary.toml`.

The E2E image is **template-shared** (one `.github/e2e-shared/Dockerfile`, used by both
base and eks-oidc). The `TEMPLATE` build-arg on `e2e-build-image.yml` / the Dockerfile
selects the per-template `.github/e2e-<template>/` pyproject dir.

EKS fresh deploys diverge from base: base wraps deploy inside the `test_deployment` pytest
(one "Deploy and verify" step), but EKS runs `jd init/config/up` as explicit, log-streaming
steps via `just ci-e2e-eks-deploy` (deploy happens *in the container*, so a pypi-mode image
deploys the published package). The ~30-min deploy is readable in its own job; the
`test_deployment` verify then runs separately against the now-existing project. The job's
`timeout-minutes` bounds the deploy.

## Release ordering & gotchas

Lessons from coordinated plugin/CLI/template releases — read before releasing:

- **Release order for coupled changes: plugin → CLI → templates.** A template's
  `manifest.yaml` can require CLI features (e.g. new component/health command schema);
  the eks-oidc release gate installs the CLI *unpinned from prod PyPI*, so the CLI must
  be published **first** or the gate's deploy fails at `jd config` with a manifest schema
  error. There's no min-CLI-version check — the coupling is implicit.
- **The CLI gate's `eks-functional-test` needs a live app #5 deployment.** Don't tear
  app #5 down before a CLI release, or that job fails at restore. (App #5 is redeployed
  by the eks-oidc gate, so there's a chicken-and-egg between the two gates — deploy app #5
  before releasing the CLI.)
- **Test PyPI re-publish is a safe no-op.** `uv publish --check-url` skips identical
  files, so re-running a release gate from the same commit does NOT burn the version —
  as long as the built artifact is byte-identical (don't change the package between runs).

## Testing Workflow Changes

To iterate on E2E workflow changes, create a temporary push-triggered workflow:

```yaml
# .github/workflows/test-<name>.yml  — DO NOT merge to main
name: Test workflow (temporary)
on:
  push:
    branches: [your-branch]
permissions:
  id-token: write    # required — reusable workflows inherit caller permissions
  contents: read
jobs:
  test:
    uses: ./.github/workflows/e2e-base-job.yml
    secrets: inherit
    with:
      oauth-app-num: "1"
      test-filter: "test_server_running"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jupyter-infra/jupyter-deploy](https://github.com/jupyter-infra/jupyter-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
