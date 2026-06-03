---
trigger: always_on
description: This repository builds the Terraform provider for Cato Networks, published as
---

# Cato Terraform Provider — Agent Guide

## Project Overview
This repository builds the Terraform provider for Cato Networks, published as
`registry.terraform.io/catonetworks/cato`. It is a Go module using HashiCorp's
Terraform Plugin Framework and the Cato Go SDK from `github.com/catonetworks/cato-go-sdk` to be able to call Cato public APIs.

## Architecture
- Each Terraform resource has a full CRUD implementation.
- The top-level Terraform model for a resource is defined in `type_*.go` with
  strongly-typed fields. Nested objects are mapped via framework attribute types
  (`attrTypes` maps / `ObjectAsOptions`), not plain Go structs - exception here are well defined shared types, e.g. GroupMembers in `type_group.go`. 
- Read, Create, and Update implementations share state-mapping logic. For complex resources  extracted into `hydrate_<name>_state.go` (API response → Terraform state) and `hydrate_<name>_api.go` (Terraform state → API input). Simple resources may map API responses through resource-local helpers.

## Structure
- `main.go` — provider entry point, wires `internal/provider.New(version)`.
- `internal/provider/` — provider config, resources, data sources, shared types,
  validators, plan modifiers, hydration helpers, and generated mocks.
  - `resource_*.go` — resource CRUD implementations
  - `resource_*_test.go` — unit test for resource implementations
  - `datasource_*.go` — data source read implementations
  - `datasource_*_test.go` — data source for resource implementations
  - `type_*.go` — Terraform schema model types
  - `hydrate_*_api.go` / `hydrate_*_state.go` — shared conversion helpers
  - `planmodifiers/` — custom plan modifier implementations
  - `validators/` — custom attribute validators
  - `mocks/` — mockery-generated mocks (do not edit manually)
- `internal/acctests/` — acceptance tests that call real Cato APIs.
- `internal/accmock/` and `test_data/` — mocked API integration tests.
- `examples/` — Terraform examples per resource and data source.
- `templates/` — provider documentation templates (used by `tfplugindocs`).
- `docs/` - generated provider docs output

## Setup
- Go `1.26.3` (see `go.mod`).
- Terraform CLI for running examples and acceptance tests.
- Required env vars for API access: `CATO_BASEURL`, `CATO_TOKEN`, provider
  `account_id`. Examples also accept `TF_VAR_token` / `TF_VAR_account_id`.
- Optional retry tuning: `CATO_RETRY_MAX`, `CATO_RETRY_WAIT_MIN_SECONDS`,
  `CATO_RETRY_WAIT_MAX_SECONDS`.

## Agentic loop for implementing changes
- Start with unit tests for behavioral changes; get user sign-off if requested.
- After implementation, iterate on unit tests until green, then add/update acceptance tests when API behavior changed, run lint and vulnerability check, update docs and examples. Don't run the acceptance tests in this loop unless specifically asked.

## Definition of Done for agentic loop
- Change is implemented with clear, maintainable code that follows existing patterns in internal/provider/.
- Unit tests are added/updated for the behavior change and pass (make test).
- Acceptance tests are added/updated when real API behavior changed.
- Imports and style checks pass (make sort-imports, make lint).
- Vulnerability check passes
- Documentation is updated and generated (make docs) to reflect the final behavior.
- Example configuration in examples/ is updated to match the implementation and docs.
- New/changed resources or data sources are registered in `internal/provider/provider.go`.
- No secrets, state files, debug logs, or manual edits to generated mocks are included.
- Scope is minimal and focused; no unrelated refactors or behavior changes.
- Final verification summary is provided to the user (what changed, tests run, any residual risks).

## Commands

```sh
make build          # compile terraform-provider-cato
make install        # build + install dev provider override (~/.terraformrc-dev)
make install-mirror # build + install filesystem mirror (needed for terraform test)
make test           # run unit tests (excludes acceptance tests)
make lint           # run golangci-lint (includes acctest build tag)
make sort-imports   # run goimports + gci on internal/
make docs           # regenerate provider docs with tfplugindocs
make mocks          # regenerate mockery mocks
make vul            # vulnerability check with govulncheck
make acctest        # run acceptance tests (real API — see Testing section)
```

Target a single package or test during development:

```sh
go test ./internal/provider/... -run TestName
```

## Testing

**Unit tests** live alongside the code in `internal/provider/`. They mock Cato
backend API calls and cover business logic and Terraform state mapping. Run with
`make test`.

**Acceptance tests** in `internal/acctests/` call real Cato APIs and create real
resources. Run with:

```sh
make acctest
```

Set `DISABLE_POLICY_RULE_CLEANUP=true` when running acceptance tests to prevent
the provider from discarding draft policy revisions during `Configure()` (already
included in `make acctest`).

## Import Ordering
Imports must follow the order enforced by `make sort-imports`:
1. Standard library
2. Third-party packages
3. `github.com/softopus-io/*` packages
4. Local module packages


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [catonetworks/terraform-provider-cato](https://github.com/catonetworks/terraform-provider-cato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
