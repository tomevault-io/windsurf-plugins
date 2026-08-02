---
trigger: always_on
description: Operating manual for AI agents working in `tests/e2e/`. These tests run **real Glue jobs against real DynamoDB tables in a real AWS account**. They are not unit tests — the usual "mock awsglue/pyspark" rules from the parent `AGENTS.md` do not apply here. Read this before adding or changing anything under `tests/e2e/`.
---

# AGENTS.md — bulk_executor e2e harness

Operating manual for AI agents working in `tests/e2e/`. These tests run **real Glue jobs against real DynamoDB tables in a real AWS account**. They are not unit tests — the usual "mock awsglue/pyspark" rules from the parent `AGENTS.md` do not apply here. Read this before adding or changing anything under `tests/e2e/`.

## What lives here

```
tests/e2e/
  conftest.py          Shared: e2e_config fixture (prompts once, caches to .e2e-config), account guard
  helpers/
    command_runner.py  Shell out to ./bulk; capture stdout/stderr/exit + scrape job_run_id  (run_command / run_command_raw / CommandResult)
    assertions.py      assert_glue_succeeded / assert_table_has_items / require_write_capable_job  ← the truthful-assertion layer
    transient_table.py Context manager: create+PITR a throwaway table, delete in finally
    capacity.py        fetch_consumed_write_capacity → observed WCU/s from CloudWatch (the rate-enforcement oracle)
    perf.py            fetch_perf(job_run_id) → JobRunPerf (real DPUSeconds + JobRunState)
    glue_bucket.py     discover the bootstrap S3 bucket; cleanup.py: orphan sweeper
  connector/           count/find/sql/load SMOKES against the live DynamoDB DataFrame connector
  commands/            fill/update/delete/copy/diff SMOKES, each on its own transient table
  security/            real-bootstrap IAM tests + job_state_guard.py (shared-job snapshot/restore)
  whole_system/        TRUE end-to-end: real datasets through the full pipeline with behavioral assertions
  results/             generated smoke reports (gitignored)
```

## Smoke vs. true e2e — they prove different things

Both hit real AWS, but do not conflate them:

- **Smoke** (`connector/`, `commands/`): early detection. Runs a command with a *small* input and asserts the Glue job reached SUCCEEDED and *something* landed. It proves the wiring doesn't crash and the connector accepts its options. It does **not** prove behavior — a smoke with 10 items passes whether or not a rate limit, ordering guarantee, or type-preservation actually works, because the dataset is too small to exercise the behavior.
- **True e2e** (`whole_system/`): drives a *realistically sized* dataset through the whole pipeline and asserts the **behavior** the feature promises — round-trip fidelity (every item, exact values, back out), an *observed* rate ceiling from CloudWatch, etc. The fixture must be large enough that the behavior is actually exercised, and the test must **guard against a vacuous pass** (see `test_load_rate_roundtrip.py::_assert_fixture_is_a_real_test`: if the load finished before the rate ceiling could bind, fail loudly instead of green).

When you add coverage for a behavioral guarantee (a rate, a limit, an ordering, a type round-trip), a smoke is not enough — add a `whole_system/` test that observes the guarantee, or you are shipping an assertion with no teeth.

## Running

```sh
make test-e2e-connector      # ~10 min
make test-e2e-commands       # ~15 min
make test-e2e-security       # ~3 min
make test-e2e-whole-system   # ~7 min (60k load: round-trip + observed rate enforcement)
make test-e2e-max-rate       # EXPENSIVE, opt-in: prove load beats the old 60k connector ceiling
make test-e2e-cleanup        # sweep orphaned transient tables
```

Requires AWS creds + a one-time `./bulk bootstrap`. First run prompts for account/region/test tables → cached in `.e2e-config` (gitignored). **Never run these in tight loops** — each Glue job costs real money and ~2 min of cold start.

### The `max_rate` tier is expensive and opt-in

`whole_system/test_load_exceeds_legacy_ceiling.py` proves the DataFrame connector sustains a write rate **above the legacy 60k WCU/s ceiling** (the old connector's 40k-on-demand-assumption × 1.5x percent cap). To *observe* >60k it must write **millions of tiny items** (1 WCU each) across thousands of partition keys into a table **pre-warmed** to the target rate — so it costs real money (order of a few dollars of DynamoDB write + Glue DPU per run) and takes several minutes. It is marked `max_rate` and **excluded from `make test-e2e-whole-system`** (`-m "not max_rate"`); run it deliberately via `make test-e2e-max-rate`. Tune cost/volume with `BULK_E2E_MAXRATE_WCU` / `_ITEMS` / `_PARTITIONS`. It guards against a vacuous pass the same way the round-trip test does: if warm throughput never provisioned or the fixture was too small to fill a hot CloudWatch minute, it fails loudly instead of green.

### Deploying the branch-under-test to Glue first

The e2e suites trigger the **deployed** Glue job — whatever code was last uploaded to S3, *not* your working tree. Before running e2e on a branch that changes `server/src/`, you must push that code to the job, or you are testing stale code and calling it a pass:

- Full deploy: `./bulk bootstrap --XRole READ-WRITE` (rebuilds + uploads the server zip; also repoints the shared job's role — see invariant #3).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/amazon-dynamodb-tools](https://github.com/awslabs/amazon-dynamodb-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
