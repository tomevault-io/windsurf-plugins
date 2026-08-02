---
trigger: always_on
description: The repository's committed contributor docs are the canonical source for schema-modeling and process conventions. This file does not duplicate them; consult them and keep them authoritative:
---

# NMDC-Schema Development Guide

## Authoritative conventions (see also)

The repository's committed contributor docs are the canonical source for schema-modeling and process conventions. This file does not duplicate them; consult them and keep them authoritative:

- `CONTRIBUTING.md`: Modeling Best Practice (naming, documentation, examples and counter-examples, enums for categorical values, PV `is_a`, reuse, placing classes under upper-level classes, ID patterning), the Current Policy vs Legacy Guidance table, and the ADR log pointer for recording decisions.
- `DEVELOPMENT.md`: prerequisites and local development environment.
- `MAINTAINERS.md` and the [infra-admin release runbook](https://github.com/microbiomedata/infra-admin/blob/main/releases/nmdc-schema.md) (in the separate `microbiomedata/infra-admin` repo): release procedure.
- `nmdc_schema/migrators/README.md`: migrator authoring and testing.

This guide adds the operational details and gotchas that are not (yet) captured in those files.

## Build/Lint/Test Commands
- Install: `poetry install --with dev,deps`
- Build: `make all` or `make site`
- Clean: `make squeaky-clean`
- All tests: `make test`
- Python tests: `make test-python`
- Schema tests: `make test-schema`
- Run single test: `poetry run pytest tests/test_file.py -v`
- Run doctest: `poetry run python -m doctest -v path/to/file.py`
- Lint: `make linkml-lint` or `make linkml-lint-all` (outputs to local/linkml-lint-all.log)
- Check schema patterns: `poetry run python src/scripts/schema_pattern_linting.py --schema-file src/schema/nmdc.yaml`
- View docs locally: `make serve` (in Docker: `poetry run mkdocs serve --dev-addr 0.0.0.0:8000`)

Build prerequisites and gotchas:
- `make all` needs `yq` (mikefarah/yq) on PATH (`brew install yq`); it is used to strip readonly metaslots and apply MIxS customizations. Java/ROBOT is NOT needed: the OWL artifact is produced by `linkml generate owl`.
- `squeaky-clean` runs `clean examples-clean shuttle-clean site-clean`; it deliberately leaves `src/schema/mixs.yaml` in place. Removing that file is a separate fast target (`mixs-yaml-clean`, just an `rm`); regenerating it (`make src/schema/mixs.yaml`) is the slow, network-dependent step. For a full-from-scratch rebuild including a fresh MIxS pull: `make squeaky-clean mixs-yaml-clean && make src/schema/mixs.yaml && make squeaky-clean all test`.
- CI tests only Python 3.10 and 3.13 (other workflows pin 3.12). `requires-python` is `>=3.10,<3.14`, so a too-new system Python (e.g. 3.14) is out of range and will break installs. Build and test against a supported interpreter.
- `local/.env` is optional. Its credential blocks (BioPortal key, NCBI BioSample Postgres, source MongoDB) are only consumed by specific scripts; `make all` and `make test` run without any of them.

## Script Entry Point Policy
- Keep `[project.scripts]` limited to package-backed, stable CLIs intended for default installs.
- Do not add ad-hoc aliases for `src/scripts/*` into `[project.scripts]`.
- For repo-local automation scripts, call them explicitly from Makefiles, e.g. `poetry run python src/scripts/<name>.py`.
- Do not add new modules under `nmdc_schema/` (the published package) without a client-facing purpose; experimental and developer-only scripts go under `src/scripts/`.

## Dependency Management

This is a Poetry repo. Sibling `submission-schema` migrated to `uv`; do not confuse the two.

- **Use poetry 2.4.1** (pinned in every CI workflow via `pipx install poetry==2.4.1`). The `poetry.lock` content-hash is poetry-version-sensitive: a different poetry version can report the lock as "out of sync with pyproject.toml" even when the dependencies are unchanged. Align dev/container poetry to the pinned version. CI runs `poetry check` (in `main.yaml`) to catch real pyproject/lock drift in PRs rather than on a teammate's fresh install. To sync after editing pyproject: `poetry lock` (no `--regenerate` unless you intend to bump versions).

- Never create a `uv.lock` here, and never run `uv add`, `uv sync`, or `uv lock`. If a stray `uv.lock` appears, delete it before committing.
- Before tightening any pin in `pyproject.toml` (especially `linkml`), check that `submission-schema` still resolves. nmdc-schema is a library, so a tighter pin propagates to consumers even when nmdc-schema itself does not exercise the feature. `linkml` and `linkml-runtime` versions must be paired (a runtime release can drop something the matching `linkml` references).
- `pyproject.toml` conventions: upper-bound policy (packages at 1.0 or above cap at the next major, pre-1.0 cap at the next minor); PEP 508 form (`"pkg>=min,<max"`, not the Poetry caret); alphabetical ordering within each dependency group; runtime deps in `[project.dependencies]`, dev tools in `[dependency-groups] dev`, `deptry` in its own `deps` group; transitive packages pinned for security are also added to `[tool.deptry] per_rule_ignores.DEP002`; comments cite the issue number that motivated a pin.

## Released migrators


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microbiomedata/nmdc-schema](https://github.com/microbiomedata/nmdc-schema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
