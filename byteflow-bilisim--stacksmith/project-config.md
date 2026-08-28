---
trigger: always_on
description: StackSmith is an infrastructure delivery agent plugin.
---

# AGENTS.md

## Project

StackSmith is an infrastructure delivery agent plugin.

Its purpose is simple:

> Turn infrastructure work items into tested, reviewable OpenTofu pull requests.

The agent may implement and verify infrastructure changes, but production changes remain a human responsibility.

## MVP Scope

The current MVP is intentionally narrow.

Supported stack:

- DeepSeek Harness as the target plugin runtime
- OpenTofu
- AWS
- Floci as the disposable AWS-compatible test environment
- ClickUp as the work-item source
- GitHub as the SCM provider
- TypeScript / Node.js for StackSmith itself

Do not add support for additional clouds, IaC engines, task trackers, or SCM providers unless explicitly requested.

In particular, do not introduce:

- Terraform-specific dependencies
- Pulumi
- Azure
- GCP
- Kubernetes orchestration
- Testcontainers
- databases
- Redis
- message brokers
- backend services
- web UI
- control-plane infrastructure

unless the task explicitly requires them.

## Core Workflow

StackSmith targets this workflow:

```text
ClickUp task
    ↓
inspect existing repository
    ↓
implement OpenTofu change
    ↓
tofu fmt
    ↓
tofu validate
    ↓
sandbox plan/apply against Floci
    ↓
verify resulting infrastructure
    ↓
run OpenTofu plan against target directory
    ↓
review resulting diff
    ↓
branch / commit / push
    ↓
create pull request
    ↓
STOP
```

Production apply is deliberately outside StackSmith's responsibility.

## Safety Boundary

The most important project rule is:

> StackSmith must never apply infrastructure changes to a real target environment.

Allowed against Floci:

- `tofu init`
- `tofu fmt`
- `tofu validate`
- `tofu plan`
- `tofu apply`
- `tofu destroy`
- AWS API/CLI inspection

Allowed against a real target:

- `tofu init`
- `tofu validate`
- `tofu plan`
- read-only inspection required to produce or interpret a plan

Forbidden against a real target:

- `tofu apply`
- `tofu destroy`
- destructive state manipulation
- resource deletion through cloud APIs
- mutating AWS CLI/API commands
- any equivalent operation that modifies real infrastructure

Do not rely only on prompts or comments to enforce this boundary.

Where practical, dangerous capabilities should not exist in the production execution interface at all.

## Architecture

Keep StackSmith divided into two conceptual areas.

```text
StackSmith Core
    |
    +-- ClickUp
    +-- OpenTofu
    +-- Floci
    +-- Verification
    +-- Git
    +-- GitHub
    |
    +-- DeepSeek Harness adapter
```

The core should remain as independent from DeepSeek Harness internals as reasonably possible.

DeepSeek Harness is currently an early/developer-preview dependency. Its APIs may change.

Therefore:

- isolate Harness-specific code under `src/dsh/`
- keep business/domain logic outside `src/dsh/`
- avoid leaking Harness-specific types through the core
- prefer thin adapters over deep framework coupling

A future Harness API change should ideally require changes only in the adapter layer.

## Repository Structure

Prefer this structure:

```text
src/
  core/
  dsh/
  clickup/
  opentofu/
  floci/
  verification/
  git/
  github/

skills/
  stacksmith/

tests/
  unit/
  fixtures/

examples/
docs/
```

Do not create new top-level packages without a clear need.

StackSmith is currently a single plugin.

Do not split it into multiple npm packages or plugins unless explicitly requested.

## Implementation Principles

### Keep the MVP small

Prefer the smallest implementation that proves the workflow.

Avoid speculative abstractions.

Do not create extension points merely because another provider might be added later.

A small interface is appropriate where there is already a real boundary, such as:

- work-item providers
- IaC execution
- SCM interaction

Do not generalize unrelated code prematurely.

### Prefer existing tools

Use existing command-line tools where they already solve the problem well.

Current preferred tools:

- `tofu`
- `git`
- `gh`
- `aws`

Do not replace them with large SDK dependencies without a concrete reason.

For example:

- prefer `gh` over introducing Octokit for the MVP
- prefer AWS CLI inspection over importing many AWS SDK packages
- prefer OpenTofu CLI over building a custom IaC execution engine

### Structured output over stdout parsing

Never parse human-oriented OpenTofu plan output if structured data is available.

Preferred plan flow:

```bash
tofu plan -out=stacksmith.tfplan
tofu show -json stacksmith.tfplan
```

Parse the JSON representation.

At minimum, distinguish:

- create
- update
- delete
- replace

Unexpected deletes or replacements must prevent automatic PR completion unless explicitly handled by the task or future policy configuration.

For the MVP, treat deletes and replacements conservatively.

### Validate external data

External boundaries should be schema validated.

Examples:

- ClickUp API responses
- StackSmith configuration
- OpenTofu JSON output
- verification results

Prefer Zod for runtime validation.

### Process execution

Use the project's process execution abstraction rather than scattering raw `child_process` usage throughout the codebase.

`execa` is the preferred process execution library.

Commands should:

- capture stdout
- capture stderr

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ByteFlow-Bilisim/stacksmith](https://github.com/ByteFlow-Bilisim/stacksmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
