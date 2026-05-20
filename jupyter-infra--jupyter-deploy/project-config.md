---
trigger: always_on
description: <!-- CLAUDE.md is a symlink to this file -->
---

<!-- CLAUDE.md is a symlink to this file -->

# Project Context
This is a monorepo to deploy Jupyter or IDE types of application to the Cloud.
It consists in several packages, all managed as uv workspace members.

## The CLI package
Code: `./libs/jupyter-deploy`
CLI tool for deploying Jupyter server to the cloud.

It's cloud-provider and infrastructure-as-code agnostic. The CLI code MUST NOT:
- depend directly on any cloud provider-specific libraries (e.g. `boto3` for AWS)
- assume that an infrastructure-as-code engine is selected (e.g. it MUST remain extensible to other engines than `terraform`)
- create custom dataclasses for AWS API types; use boto3 type stubs directly (e.g. `ObjectTypeDef`, `TagTypeDef`)

To access cloud-provider specific dependencies, we use optional installs such as `pip install juptyer-deploy[aws]` 
Then module `provider/instruction_runner_factory` handles these optional imports.
You MUST NOT break that pattern with import statements to cloud-provider or infrastructure-as-code specific libraries
outside of the instruction runner code paths.

### Three-Layer Architecture

**1. Core Layer** (handlers, engine, provider):
- Handlers provide abstraction for each `jupyter-deploy` commands
- Raise exceptions from `jupyter_deploy/exceptions.py` for errors
- Accept `DisplayManager` instance from CLI, then use its method: `info()`, `warning()`, `success()`, `hint()`
- Defines `SupervisedExecutor` class for managing infrastructure-as-code subprocesses
  - Located in `engine/supervised_execution`
  - Emits progress events that `DisplayManager` handles
  - Supports switching between `stdin` and `stdout` when subprocess prompts for input
- Defines the abstract, provider-agnostic command runner: `/provider/manifest_command_runner`
  - Run commands declared in a template manifest
  - Use a specific provider module (e.g. `/provider/aws`), which calls provider-SDK (e.g. `boto3`)
  - Optional install thanks to lazy import in factory module `/provider/instruction_runner_factory`

**2. Provider and Engine Implementations** (unified abstraction):
- Engine implement specific command Handlers for a specific infrastructure-as-code engine; current engines:
  - `terraform`
- Engine {Config|Up|Down}Handlers leverage `SupervisedExecutor` to run the infrastructure-as-code subprocess calls
- Provider instruction runners implement the `InstructionRunner` interface to make API calls with the specific provider SDK; current providers:
  - `aws`

**3. CLI Layer** (cli/):
- Instantiate Console and error handler
- Call core handlers and catch exceptions
- Format and display results using rich/typer
- Implement `DisplayManager` protocol with display managers; implementations:
  - `SimpleDisplayManager` (cli/simple_display.py) - Spinners, status messages for SDK-style operations
  - `ProgressDisplayManager` (cli/progress_display.py) - Progress bars, log boxes for long operations
  - `NullDisplay` (engine/supervised_execution.py) - No-op for programmatic/test usage

### Key Principles

1. **Exception Handling**: All custom exceptions in `jupyter_deploy/exceptions.py`
2. **Keep Core Generic**: Core defines interfaces, instantiates engine-specific and provider-specific instance as needed
3. **No Terminal-specific Dependencies in Core**: rich/typer only in cli/ module
4. **No Engine-specific implementation in Core**: use the `/engine/<engine-name>` module
5. **Not Provider-specific implementation in Core**: use the `/provider/<provider-name>` or `/api/<api-name>` modules

## Base template package
Code: `./libs/jupyter-deploy-tf-ec2-base`

Primary template used by the CLI, referred to as "base template".
- infrastructure-as-code engine: `terraform`
- cloud provider: `aws`
- identity provider: `github`

All variables MUST be defined in `variables.tf` without default values.
Default values MUST be set in `presets/defaults-all.tfvars`.
There MUST NOT BE be any `variable` blocks in files other than `variables.tf`.

IMPORTANT: Do not copy files to `/home/jovyan` during Docker build time.
The EBS volume for Jupyter data is mounted at runtime, and any files copied during build will be hidden by this mount.
Instead, copy files to a location like `/opt` during build and then copy them to `/home/jovyan` in startup scripts.

## CI infrastructure template package
Code: `./libs/jupyter-infra-tf-aws-iam-ci`

Template that manages AWS resources for GitHub Actions CI.
- infrastructure-as-code engine: `terraform`
- cloud provider: `aws`
- no host/server resources — IAM roles, SSM parameters and secrets only

**IMPORTANT:** The GitHub Actions OIDC provider is a singleton per AWS account.
The `create_oidc_provider` variable controls whether to create it or reference an existing one.
Set to `false` if another deployment in the same account already created it.

## E2E Pytest plugin package
Code: `./libs/pytest-jupyter-deploy`

A set of pytest fixtures to run end-to-end tests for templates, referred to as "pytest plugin".

It bundles the E2E container image (Dockerfile + docker-compose.yml) used by the justfile to run E2E tests. The image is template-independent — it provides base tooling (Python, Terraform, AWS CLI, Playwright) while template-specific tests are synced at runtime.

# Development Workflow

## After code changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jupyter-infra/jupyter-deploy](https://github.com/jupyter-infra/jupyter-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
