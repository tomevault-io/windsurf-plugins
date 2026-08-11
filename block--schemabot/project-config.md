---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

SchemaBot is a declarative schema GitOps orchestrator for MySQL via Spirit and Vitess via PlanetScale. It makes running database schema changes safe and easy through GitHub PR comments and an Admin CLI.

## Build and Test Commands

```bash
make build                # Build binary → bin/schemabot
make test                 # Run ALL tests (unit + integration + e2e)
make test-unit            # Unit tests only (fast, no containers)
make test-integration     # Integration tests (testcontainers)
make test-e2e             # E2E tests (docker-compose: local + gRPC)
make test-e2e-local       # E2E local tests only
make test-e2e-grpc        # E2E gRPC tests only
make lint                 # golangci-lint via Docker
```

**Always run `make test` (full suite)** to validate changes before a PR push when the scope of the change is unclear or crosses package boundaries. Only use `make test-unit` or `make test-integration` when you are confident the change is scoped to a single layer.

**Never assume test failures are unrelated to your changes.** Always investigate failures deeply — even if they appear in code you didn't touch — and fix them if possible.

**Detecting flakes:** Use `scripts/test-flaky.sh <TestName> [iterations] [package]` to run a specific test multiple times and confirm stability. Always use this after fixing a flaky test to verify the fix holds (e.g., `scripts/test-flaky.sh TestMultiKeyspaceDDLDeploy 5 ./pkg/localscale/...`).

**Never increase timeouts to fix flakes.** If a test is slow or flaky in CI, find and fix the root cause — don't mask it by increasing deadlines or poll timeouts. Common root causes: resource contention, missing cleanup between tests, container startup races, unbounded retries.

**Never skip tests in CI.** All tests must run in every CI job. If a test fails because of a missing dependency (Docker image, container, service), fix the CI workflow to provide it — don't skip the test. Prefer fixing underlying issues over shortcuts or workarounds.

## Test Architecture

Three test layers, each with its own make target and CI job:

- **Unit** (`make test-unit`, `./...`) — No containers, no network. Fast. Tests individual functions and packages with `-race`.
- **Integration** (`make test-integration`, `-tags=integration ./...`) — A mix of Docker containers (MySQL via testcontainers) and in-process server/gRPC components. Tests cross-package interactions without requiring a full deployment. Lives in `pkg/` and `integration/`.
- **E2E** (`make test-e2e`, `-tags=e2e ./e2e/...`) — Full docker-compose stack. All components (SchemaBot server, MySQL instances) are real containers — nothing is in-process or mocked. Tests the CLI against a running system. Lives in `e2e/local/` and `e2e/grpc/`.

Integration tests are the workhorse — most test coverage lives here since they're cheaper to run than full e2e. E2E tests are more expensive (docker-compose setup/teardown) but essential for validating the CLI against a real running system. Robust automated tests across all three layers are the only safe way to evolve SchemaBot.

CI mirrors local dev exactly — each job runs the corresponding make target.

## Git

- **Always verify the active branch before starting work.** The deployed code may be on a feature branch (e.g., a worktree), not `main`. Before implementing, check which branch has the code you're building on: `git log --oneline <branch> -- <relevant-file>`. Never branch from `main` for features that depend on unreleased work — branch from (or commit directly to) the active development branch.
- **Never bypass pre-commit hooks.** Do not use `--no-verify` or `core.hooksPath=/dev/null`. If the hook fails, fix the issue.
- **PR summaries should be concise** — a short paragraph or bullet list highlighting key changes and why, not low-level implementation details. When a change affects flow, architecture, state transitions, or concurrency, include a small ASCII diagram if it makes the behavior easier to review; omit diagrams when they would add noise. Do not include test plans, checklists, or verification details. Never reference internal company details (specific database names, staging environments, team names, internal URLs, deployment hostnames, org names, or test repo names) in PR titles or descriptions — this is a public OSS repo.
- **Agent PR disclosures belong at the bottom.** When an agent writes or updates a PR summary/body, put the agent disclosure line after the summary content, not at the top.
- **Do not create PRs automatically.** Wait for the user to explicitly ask before running `gh pr create`. Pushing a branch is fine; creating the PR is a separate decision.
- **Create PRs in draft mode** (`gh pr create --draft`) by default. The author will mark it ready for review.
- **After pushing new commits**, check if the PR title and summary need updating to reflect the new changes. If a human has edited the summary, leave it alone.
- **Never squash after a human has reviewed** (comments or approval) — add new commits so reviewers can see incremental changes. Before human review, squash freely to keep the PR clean.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [block/schemabot](https://github.com/block/schemabot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
