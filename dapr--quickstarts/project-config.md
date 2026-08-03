---
trigger: always_on
description: This file provides guidance to AI coding assistants (Claude Code, Cursor, GitHub Copilot, and other tools that read repo-root context files) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to AI coding assistants (Claude Code, Cursor, GitHub Copilot, and other tools that read repo-root context files) when working with code in this repository.

## What this repo is

The **Dapr Quickstarts and Tutorials** — runnable, doc-driven samples that teach Dapr building block APIs (pub/sub, state, service invocation, bindings, secrets, actors, configuration, cryptography, resiliency, jobs, workflows, conversation/AI) across five language SDKs. Each sample is self-contained and is validated end-to-end in CI. These samples back the [Dapr docs Quickstarts](https://docs.dapr.io/getting-started/quickstarts/) and are often a reader's *first* hands-on experience with Dapr, so clarity for newcomers, correctness, and consistency matter far more than cleverness.

## The single most important convention: the README *is* the test

Every quickstart's `README.md` is an executable document. `make validate` runs [`mechanical-markdown`](https://github.com/dapr/mechanical-markdown) (`mm.py`) over the README: it executes the shell commands inside `<!-- STEP ... -->` / `<!-- END_STEP -->` blocks and asserts that the program's output matches the annotated expectations. CI runs exactly this. Consequences:

- **A quickstart with no validated STEP blocks is not done.** New or changed behavior must be reflected in STEP annotations, or CI won't actually exercise it.
- **Outputs must be deterministic.** Samples are wired to deterministic components (e.g. the `conversation.echo` component, fixed message loops) so `expected_stdout_lines` is stable. Don't assert on values that vary run-to-run.
- **Editing prose is fine; editing commands or expected output means re-validating.** If you touch a fenced command block or the program it runs, re-run `make validate` for that quickstart.

A STEP block looks like this — the keys are the contract CI checks:

```markdown
<!-- STEP
name: Run multi app run template
expected_stdout_lines:
  - 'Published data: {"orderId":1}'
  - 'Subscriber received: {"orderId":1}'
expected_stderr_lines:
output_match_mode: substring   # match substrings, not whole lines
match_order: none              # lines may appear in any order
background: true               # run async (long-lived `dapr run`)
sleep: 15                      # wait before checking output
timeout_seconds: 30            # then tear the process down
-->

​```bash
dapr run -f .
​```

<!-- END_STEP -->
```

Long-running `dapr run` commands use `background: true` and are paired with a later `dapr stop` STEP. The "Run individually" sections of READMEs are usually plain (non-STEP) examples for humans; the validated path is the `dapr run -f .` multi-app section.

## Anatomy of a quickstart

An individual quickstart contains:

- `README.md` — the validated walkthrough (the test; see above).
- `makefile` — boilerplate that includes `docker.mk` + `validate.mk` (no custom logic lives here).
- `dapr.yaml` — the multi-app run template.
- One or more app folders — the actual sample code.
- A shared `components/` directory with the Dapr component YAML the sample uses, referenced by relative path rather than duplicated per language.

Two cross-cutting conventions:

- **`http` vs `sdk` variants**: `http` calls Dapr's APIs over plain HTTP; `sdk` uses the language's Dapr SDK. Many samples ship both.
- **Coverage is intentionally uneven.** Not every building block is implemented in every language or variant (an SDK may not support a feature yet). Mirror what comparable samples already do rather than assuming a full matrix.

## Commands

Run from a **single quickstart directory** (e.g. `cd pub_sub/python/sdk`):
```sh
make validate          # run mechanical-markdown over README.md (the real test)
```

Run from the **repo root** to validate everything for a language:
```sh
make test_python_quickstarts       # also: go | csharp | java | javascript
make test_all_quickstarts          # every language
```
These iterate the top-level building-block directories and run `make validate` in each `<lang>/http` and `<lang>/sdk` they find (Python wraps each in a throwaway `.venv`).

Bump SDK versions across all quickstarts (then verify + re-validate before committing):
```sh
make update_python_sdk_version DAPR_VERSION=1.16.0 FASTAPI_VERSION=1.16.0 WORKFLOW_VERSION=1.16.0
make update_gosdk_version VERSION=v1.16.0          # Go version MUST start with 'v'
make update_dotnet_sdk_version VERSION=1.15.0
make update_java_sdk_version VERSION=1.12.0
make update_javascript_sdk_version VERSION=3.4.0
```

## Prerequisites for validating locally

- **Dapr CLI + initialized runtime**: `dapr init` (pinned versions live in `.github/env/global.env`).
- **mechanical-markdown**: `uv tool install mechanical-markdown` (or `pip install mechanical-markdown`) so `mm.py` is on PATH. `make validate` will attempt to `pip install` it if missing.
- **A container runtime** (Docker or Podman) for samples that use real components — e.g. pub/sub defaults to a local Redis (`pubsub.redis` at `localhost:6379`) provisioned by `dapr init`.
- **Language toolchain** for the variant you touch (see below).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dapr/quickstarts](https://github.com/dapr/quickstarts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
