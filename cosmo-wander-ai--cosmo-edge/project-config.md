---
trigger: always_on
description: This file gives coding agents stable repository facts. Customer- or task-specific
---

# CosmoEdge Agent Guide

This file gives coding agents stable repository facts. Customer- or task-specific
information belongs only in the ignored `output/agent-runs/` directory. These
rules support agent-assisted work without changing the normal contributor
workflow.

## Repository map

- `src/` — C++ engine, split by subsystem.
- `src/web/` — Vue 3 frontend.
- `docs/` — VitePress site; Chinese pages are at the root and English mirrors
  are under `docs/en/`. Navigation is in `docs/.vitepress/config.mts`.
- `scripts/` — build, run, validation, and packaging scripts.
  `scripts/agent/` contains environment admission, measured evidence, and
  implemented task-specific executors.
- `tools/` — Python and Node validation utilities.
- `test/` — Catch2 tests and the HTTP/MQTT push test service.
- `data/resource/aiboxresource/` and `data/resource/aiboxresource_x86/` —
  Sophon and x86 model resources and templates.
- `3rd/` and `prebuild/` — third-party and prebuilt dependencies; do not edit.

## Mandatory agent-assistance trigger

For a task that combines customer/task model material with model conversion,
remote execution, model transfer, or test-device work, activate this repository's
workflow before the first external or environment-changing command:

1. Restate the business outcome and intended deliverables, then run
   `scripts/agent/start.sh` (or `start.ps1`) with the objective and named
   materials. This creates the private task contract and route assessment; the
   customer does not write JSON or select an implementation recipe.
2. Resolve `needsInput`. A user message that supplies a development host and
   credentials and asks the agent to connect/check/develop is already explicit
   task-scoped `remote-execution` authority; when model material is named for
   that host it also grants `model-transfer`. Pass
   `--user-requested-remote-access` to `start` and do not ask the same question
   again. If intent is unclear, ask one natural-language question, then use
   `authorize.sh --confirmed-by-user` and reassess.
3. A `READY` assessment is required before task `doctor`; that admitted doctor
   report is required before conversion; both are rechecked by verification.
   Changing the contract or assessment invalidates downstream admission.

After the private record and explicit remote grant exist, use `connect.sh` (or
`connect.ps1`) to open interactive SSH; OpenSSH receives the password through
its terminal prompt and the repository stores only a sanitized event. Invoke it
in an interactive terminal/PTY and answer the prompt without repeating the
password in commentary or a command. Raw `ssh`,
`scp`, installation, `sudo`, conversion, or deployment before the record/grant is
outside the assisted workflow. This trigger does not alter ordinary local
repository development that needs none of these task-specific capabilities.

## Working order

1. Start from the user's business outcome, available materials, target context,
   and acceptance. Restate the intended deliverables in ordinary language.
   Do not ask the user to choose a chip flag, precision, compiler version,
   container, script, or repository example when those facts can be inspected
   or recommended from the material, target device, repository, and current
   official documentation.
2. Use `./scripts/agent/start.sh --objective ...` to create the private task
   contract under `output/agent-runs/<run-id>/` and perform the first assessment.
   The contract is an agent-owned execution record, not a customer form. Use
   `assess.sh --contract ...` again after task or authority changes. Translate
   `needsInput` into one concise user question; do not expose internal JSON for
   confirmation.
3. Run `./scripts/agent/doctor.sh --baseline` for a local read-only inventory.
   If the user supplied a remote development machine and explicitly requested
   connection, open it through `connect.sh` even when other business input is
   still unresolved, inspect it read-only, and run task admission on the machine
   that will actually execute the work.
4. Select relevant tutorials, examples, templates, tests, and code facts. When
   reusing an example or template, record why it applies and how the task
   differs. A direct code or documentation task may record these sources in its
   evidence instead of creating a separate selection file.
5. Treat an upstream-supported route as eligible to try, not as proof that the
   current machine is ready. Admit by required capabilities and callable tools;
   after admission, freeze the actual Python/package/image/tool identities and
   commands used by this run. Do not require one globally fixed recipe.
6. Execute only inside the granted scope. Infer no authority from credentials
   alone, but treat credentials plus an explicit connect/check/develop request
   as confirmation and record it without another round trip. Otherwise use
   `authorize.sh`, then reassess. Use four coarse gates when applicable:
   `environment-change`, `remote-execution`, `model-transfer`, and
   `device-deployment`. Keep the exact target, read/write scope, impact, and
   recovery plan in the task context without turning every implementation
   detail into a blocking standard.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cosmo-wander-ai/cosmo-edge](https://github.com/cosmo-wander-ai/cosmo-edge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
