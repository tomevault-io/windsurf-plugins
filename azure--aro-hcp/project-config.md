---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the main repository for Red Hat OpenShift on Azure (ARO) using the Hosted Control Planes (HCP) architecture. It contains some of the code for the required microservices along with most of the configuration and pipeline definition necessary to deploy it.

## Common Commands

### Build, Test, Lint (whole repo)
```bash
make test              # Run all unit tests across the Go workspace (requires envtest setup, done automatically)
make lint              # Run golangci-lint across all modules
make lint-fix          # Run lint with --fix
make fmt               # Run goimports across all modules
make verify            # Run all verification checks (deepcopy, json-format, yamlfmt, materialize, gomega, schema) — this is what CI runs
make verify-kql        # Validate KQL (Kusto Query Language) files
make generate          # Regenerate deepcopy, mocks, format, tidy
make generate-kiota    # Regenerate the ARM SDK (under test/sdk/), then run licenses + fmt
make tidy              # Run go mod tidy across all modules + go work sync
make install-tools     # Install all dev tools via bingo
```

### Running a single test
There is no workspace-wide single-test shortcut. `cd` into the module directory and run:
```bash
cd frontend && go test -run TestMyFunction -v ./...
cd internal && go test -run TestSomething -v ./path/to/package/...
```

For integration tests (require Cosmos emulator):
```bash
cd test-integration && make test    # Runs all integration tests with Cosmos emulator
go test ./test-integration/frontend/...  # Frontend integration tests only
go test ./test-integration/backend/...   # Backend integration tests only
```

### E2E tests (local)
```bash
make e2e/local                                  # Full local E2E suite
make e2e-local/run-test TEST_NAME="TestName"    # Single E2E test
```

### Building individual services
```bash
make -C frontend build   # Build frontend binary
make -C backend build    # Build backend binary
make build-services      # Build and push all in-repo service images in parallel (requires ACR login)
```

### Config changes
After modifying `config/config*.yaml` or schema files:
```bash
cd config && make materialize   # Re-render configs and update helm fixtures
```
Rendered config changes under `config/rendered/` must be committed with your PR. Run `make verify-materialize` to check.

### Personal dev environment
```bash
make personal-dev-env   # Build, push images, deploy infrastructure (DEPLOY_ENV=pers or swft required)
```

### Other useful commands
```bash
make rebase                      # Rebase on upstream and re-materialize config (hack/rebase-n-materialize.sh)
make envtest-setup               # Download kubebuilder binaries (etcd, kube-apiserver) for controller tests
make record-nonlocal-e2e         # Regenerate nonlocal-e2e-specs.txt after adding/removing/renaming E2E tests
make validate-config-pipelines   # Validate all pipeline.yaml files against topology and config
```
Note: `make test` runs `envtest-setup` automatically, but if you run `go test` directly in a module with controller tests (e.g. `kube-applier`), you need `KUBEBUILDER_ASSETS` set. Use: `export KUBEBUILDER_ASSETS=$(make -s envtest-setup)`

## Target Environments

There's a multi-layered build and deploy system supporting multiple target environments with configs under `config/`):
- **Personal dev**
  - cloud: "dev"; environment: "pers" (`DEPLOY_ENV=pers`)
  - Local development using `make` and properly-configured `az` command
  - Changes must be manually applied by the user by running proper `make` commands
- **CSPR environment** (Clusters Service PR check)
  - cloud: "dev"; environment: "cspr" (`DEPLOY_ENV=cspr`)
  - Uses Prow for CI/CD (jobs defined in [openshift/release](https://github.com/openshift/release/tree/master/ci-operator/jobs/Azure/ARO-HCP))
  - Changes get automatically applied after a PR merge
  - Note: The integrated/shared dev environment (`DEPLOY_ENV=dev`) has been decommissioned. Only global infrastructure (shared ACRs, DNS zones) is still deployed to the dev environment via the `global-pipeline-postsubmit` Prow job.
- **Production systems**
  - cloud: "public"; environments: "int", "stg", "prod"
  - Deployed via Microsoft ADO and EV2 into Microsoft Int, Stage and Prod environments (hosted in Microsoft Azure tenants)
  - For changes to be applied, a PR must be created for `sdp-pipelines` repo and updated pipelines run manually
  - After making changes, remind user about this and point at aka.ms/arohcp-pipelines for next steps


### Access

* "Dev/*" environments can be accessed with a @redhat.com account and are largely read/write. It's likely that `az` is currently logged into it.
* "Public/Int" environment can be accessed with a @microsoft account and is mostly read-only. It's possible to `az login` into, but you need to ask the user explicitly to do that.
* "Public/Stage" and "Public/Prod" require SRE-level access.

## Deployments

Loosely defined categories of deployed objects:
- Native Azure objects (using bicep files in `dev-infrastructure/`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/ARO-HCP](https://github.com/Azure/ARO-HCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
