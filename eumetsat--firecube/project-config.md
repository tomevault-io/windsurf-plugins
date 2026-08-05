---
trigger: always_on
description: This repo is a **batch ingestion worker** for EO datasets that writes Zarr/Parquet and maintains a product-local `.firecube/` control-plane root used for idempotency, resume safety, cleanup, and write coordination.
---

# Firecube ingestor (AGENTS.md)

This repo is a **batch ingestion worker** for EO datasets that writes Zarr/Parquet and maintains a product-local `.firecube/` control-plane root used for idempotency, resume safety, cleanup, and write coordination.

## Quick start

- For full test suite: `uv sync --extra test`
- Install deps: `uv sync`
- Run CLI: `uv run firecube --help`
- Tests: `uv run pytest`
- Lint: `uv run ruff check .`
- Format check: `uv run ruff format --check .`
- Type check: `uv run pyright`

Before running the test suite, install the CLI integration test fixture plugin:

```bash
uv pip install -e tests/fixtures/cli_test_plugin
uv pip install -e tests/fixtures/direct_zarr_capable_test_plugin
uv pip install -e tests/fixtures/direct_zarr_non_capable_test_plugin
uv pip install -e tests/fixtures/multi_group_capable_test_plugin
uv pip install -e tests/fixtures/cf_time_dim_test_plugin
uv pip install -e tests/fixtures/slot_shape_test_plugin
```

This is required for `tests/integration/test_ingest_command_typed.py` and related CLI tests.

## Test Discipline

Test skip policy, CI invocation, and pytest markers. → See [plans/TEST.md](plans/TEST.md)
Behavior-first testing standards, static-test limits, and the test-suite overhaul plan. → See [plans/TESTING_STANDARDS.md](plans/TESTING_STANDARDS.md)

## Documentation prompts

Before editing documentation, read [.prompts/docs-policy.md](.prompts/docs-policy.md).

When editing public docs:

- Identify the primary audience first: user, plugin author, operator, or contributor.
- Prefer commands, examples, expected output, verification steps, and troubleshooting.
- Explain user consequences before architecture.
- Do not add phase history, audit findings, commit labels, reviewer notes, internal service names, private module paths, line numbers, `.sisyphus/`, or `plans/` references to public task pages.
- Use public CLI flags and public SDK imports (`firecube.ingestor.api`, `firecube.core.api`) unless the page is explicitly internal.
- Do not create or substantially rewrite a docs page without choosing the matching prompt from `.prompts/` (`write-user-doc`, `write-plugin-doc`, `write-operator-doc`, or `write-internal-doc`).
- Put implementation history, design rationale, and maintenance evidence in `plans/`, `.sisyphus/`, or an explicitly internal page, not in the user-facing path.

## Workflow prompts

Reusable project workflows live in `.prompts/` (docs fact-checking, review-comment
handling, coverage, edge cases, error handling, release preparation, and PR
preflight). Prefer these prompt files over inventing a new checklist for repeated
maintenance tasks.

## CLI requirements (explicit, no inference)

The CLI infers storage settings from the URI scheme for commands that take a
product URI (`file://`→local, `s3://`→s3; driver defaults to `fsspec`). Pass
`--storage-type` and `--storage-driver` explicitly to override.

**All flags below are required explicitly:**

- `--product-name <name>` — logical product name (or set `PRODUCT_NAME` on your plugin class)
- `--target <uri>` — where to write the output product
- `--write-mode [staged|direct]` — write strategy (NOT inferred from local vs remote)

Write-tier commands require `--write-mode`, except `zarr multires`, which
derives pyramid levels in place and has no staging path.

**Removed behaviors (migration required):**
- `output_name` no longer inferred from target URI basename — use `--product-name` or plugin `PRODUCT_NAME`
- `storage_type` no longer inferred from `s3://` vs `file://` — pass `--storage-type` explicitly
- `write_mode` no longer defaults to `direct` for local targets — pass `--write-mode` explicitly
- Config key `default_output_name` is rejected — use `default_product_name` instead

**Product name precedence:** CLI `--product-name` > config `default_product_name` > plugin `PRODUCT_NAME` > hard fail.

`--source`/`--target` are always strict product URIs; `--archive`/`--output`
are always strict artifact URIs; `--input-data`, `--config-file`,
`--target-dir`, and `--workspace` are exempt from the strict-URI policy
(`--input-data` accepts a local path or an `s3://` prefix, interpreted by the
plugin; the rest are local paths).

## Architectural invariants

Core design rules for this batch ingestor, including control-plane model and observability rules. → See [plans/DESIGN.md](plans/DESIGN.md)

## Where things live

- Plugin contract: `src/firecube/ingestor/runtime/base.py` (`BaseIngestor`) and `src/firecube/ingestor/contracts/interfaces.py` (Protocols).
- Engine/pipeline runner: `src/firecube/ingestor/runtime/engine.py`.
- Public surfaces for plugins: `src/firecube/ingestor/api.py` and `src/firecube/core/api.py` (prefer these over deep imports).
- External plugins: installed via entry points under `firecube.plugins`; plugin-specific usage belongs in each plugin repository.
- Storage/fs: `src/firecube/core/filesystem/` + `src/firecube/core/storage/` (fsspec-based facade).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eumetsat/firecube](https://github.com/eumetsat/firecube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
