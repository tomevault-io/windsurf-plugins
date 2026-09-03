---
trigger: always_on
description: Instructions for AI coding agents working in this repository.
---

# CLAUDE.md

Instructions for AI coding agents working in this repository.

## What this project is

Plimsoll is a verification agent for AI-generated Infrastructure-as-Code. It sits in CI between a code-generating agent and a cloud account, finds policy violations, drafts fixes, **re-verifies those fixes with deterministic tools**, and emits a provenance record for every decision.

Read `architecture.md` before writing code. Read `specifications.md` before changing any schema.

## The invariants

These are not style preferences. Breaking one of them breaks the project's reason to exist. If a task appears to require breaking one, stop and say so instead of proceeding.

1. **The LLM never decides whether something is compliant.** Only `Check` adapters decide. The model's output is a *proposal*; the gate is the *judge*. There is no code path where a model's assessment sets `Decision.status`.

2. **Every LLM-authored patch is re-run through the same gate that produced the finding.** Same code path, same adapters, same config. Not a lighter check, not a second model, not a heuristic.

3. **A patch that fails re-verification never reaches a human as a suggestion.** It is discarded. After `max_retries` the run escalates with the *original* finding and an explicit "could not fix" status.

4. **Deterministic-only mode must always work.** Running with `--no-llm` gives a functioning policy scanner. If the LLM provider is unreachable, the tool degrades to scanning — it never fails open, and it never blocks on inference.

5. **`terraform apply` is never invoked.** Not in code, not in tests, not in examples. This is a pre-deployment tool. If you find yourself needing `apply`, the design is wrong.

6. **No secrets in provenance records.** Provenance captures rule IDs, resource addresses, before/after values, and approver identity. Redact anything matching a secret pattern before writing.

## Repository layout

```
src/plimsoll/
  checks/         Check adapters: terraform_validate, terraform_plan, checkov, opa
    terraform_cli.py  Shared init/plan/show plumbing, plugin cache, provider override
    locator.py    Resource address -> file and line; plan JSON carries neither
    rule_map.py   Checkov id -> rule_class, and the twenty PLIMSOLL rule classes
  llm/            LLMProvider implementations (openai_compat, null) + prompts/*.txt
  core/
    models.py     Pydantic schemas — Finding, Patch, Decision, ProvenanceRecord
    gate.py       Runs checks, aggregates findings
    deadline.py   The monotonic run budget behind --timeout and exit code 4
    context.py    Builds every prompt; the single redacted egress point
    verifier.py   Applies a patch to a scratch copy and re-runs the gate
    loop.py       The verify -> reason -> re-verify -> escalate loop
    redaction.py  Secret patterns, one-way and reversible
    exceptions.py ExceptionStore
    provenance.py Record construction and hashing
  policies/       The bundled Rego pack: lib, iam, s3, network, datastore, hygiene,
                  plus *_test.rego run by `opa test` and excluded from the wheel
  reporters/      terminal, json_report, sarif, github
  cli.py          Typer entrypoint
action/           Dockerfile, entrypoint.sh, post_comment.py for the GitHub Action
action.yml        The Action's metadata; at the root because a container action's
                  build context is the directory holding it
benchmarks/       runner.py, scenarios.yaml, BASELINE.md, RESULTS.md
schemas/          Published JSON Schema for the provenance record
tests/
  fixtures/       Terraform configs with known violations, and recorded tool output
```

The rule pack lives at `src/plimsoll/policies/`, not a top-level `policies/`. A bundled
pack that does not ship in the wheel is not bundled.

## Commands

```bash
uv sync --all-extras          # install
uv run pytest                 # tests
uv run pytest -m "not slow"   # skip tests that shell out to terraform
uv run ruff check . --fix     # lint
uv run ruff format .          # format
uv run mypy src/              # types
uv run plimsoll doctor                              # which external tools and which model
uv run plimsoll scan ./tests/fixtures/insecure_s3   # manual smoke test, exits 1
uv run plimsoll scan ./tests/fixtures/compliant_s3  # the false-positive check, exits 0
uv run plimsoll verify ./tests/fixtures/insecure_s3 # the full loop; needs PLIMSOLL_LLM_*

opa check --strict src/plimsoll/policies            # the rule pack parses
opa test src/plimsoll/policies                      # the rule pack's own unit tests
uv run python benchmarks/runner.py --dry-run        # the benchmark wiring, spends nothing
```

A cold `terraform init` downloads the AWS provider, which can exceed the 120s default on
a slow link. Raise `--check-timeout` rather than assuming the configuration is wrong;
`--timeout` is the whole-run budget and defaults to 600.

Model configuration is three environment variables. Any OpenAI-compatible endpoint:

```bash
PLIMSOLL_LLM_BASE_URL=https://your-router.example/v1
PLIMSOLL_LLM_MODEL=your-model-id
PLIMSOLL_LLM_API_KEY=...
```

Run `ruff`, `mypy`, and `pytest` before declaring any task complete.

## Code conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trh-ds/plimsoll](https://github.com/trh-ds/plimsoll) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
