---
trigger: always_on
description: This document identifies and describes all agent and agent-like components in the [Lightning-AI/utilities](https://github.com/Lightning-AI/utilities) repository.
---

# AGENTS.md

This document identifies and describes all agent and agent-like components in the [Lightning-AI/utilities](https://github.com/Lightning-AI/utilities) repository.
Agents are defined as independently-triggerable components responsible for automating tasks, orchestrating workflows, or executing delegated utility operations.
For each agent, its **name**, **purpose**, **functionality**, and **relative file location** are given.

______________________________________________________________________

## 1. GitHub Workflow Agents

### Agent: check-schema

- **Purpose**: Validates configuration or data schema on code pushes.
- **Functionality**: Automatically runs schema checks as a job for every push, using centralized logic.
- **Relative Location**: `.github/workflows/check-schema.yml`

### Agent: check-docs

- **Purpose**: Validates documentation build and style.
- **Functionality**: Builds docs and runs documentation-specific checks to keep documentation healthy.
- **Relative Location**: `.github/workflows/check-docs.yml`

### Agent: check-typing

- **Purpose**: Ensures static typing quality.
- **Functionality**: Runs type checking to maintain type correctness.
- **Relative Location**: `.github/workflows/check-typing.yml`

### Agent: check-package

- **Purpose**: Validates packaging.
- **Functionality**: Builds the distribution and verifies artifacts integrity and metadata.
- **Relative Location**: `.github/workflows/check-package.yml`

### Agent: check-md-links

- **Purpose**: Verifies Markdown links.
- **Functionality**: Scans Markdown files for broken or redirected links.
- **Relative Location**: `.github/workflows/check-md-links.yml`

### Agent: check-precommit

- **Purpose**: Enforces code style and quality gates.
- **Functionality**: Runs configured pre-commit hooks (formatting, linting, etc.).
- **Relative Location**: `.github/workflows/check-precommit.yml`

### Agent: ci-use-checks

- **Purpose**: Coordinates schema and code validation jobs for broad CI coverage.
- **Functionality**: Calls multiple reusable workflows; acts as a top-level CI orchestrator.
- **Relative Location**: `.github/workflows/ci-use-checks.yaml`

### Agent: ci-testing

- **Purpose**: Executes the test suite.
- **Functionality**: Runs unit/integration tests across supported environments.
- **Relative Location**: `.github/workflows/ci-testing.yml`

### Agent: ci-cli

- **Purpose**: Exercises CLI-related checks/tests.
- **Functionality**: Validates CLI utilities and their expected behavior.
- **Relative Location**: `.github/workflows/ci-cli.yml`

### Agent: ci-scripts

- **Purpose**: Validates repository scripts.
- **Functionality**: Runs tests and checks over automation scripts in this repo.
- **Relative Location**: `.github/workflows/ci-scripts.yml`

### Agent: deploy-docs

- **Purpose**: Publishes documentation.
- **Functionality**: Builds and deploys docs to the chosen hosting target.
- **Relative Location**: `.github/workflows/deploy-docs.yml`

### Agent: release-pypi

- **Purpose**: Publishes releases to PyPI.
- **Functionality**: Builds and uploads distribution packages upon release conditions.
- **Relative Location**: `.github/workflows/release-pypi.yml`

### Agent: cleanup-caches

- **Purpose**: Manually clears caches.
- **Functionality**: Provides a job to purge caches when necessary.
- **Relative Location**: `.github/workflows/cleanup-caches.yml`

### Agent: cron-clear-cache

- **Purpose**: Maintains clean build environments by scheduled cache clearance.
- **Functionality**: Invokes Python/environment cache clearing on a weekly schedule to keep CI fast.
- **Relative Location**: `.github/workflows/cron-clear-cache.yml`

### Agent: label-pr

- **Purpose**: Automatically labels pull requests.
- **Functionality**: Applies labels to PRs based on rules to streamline triage.
- **Relative Location**: `.github/workflows/label-pr.yml`

### Agent: ci-rtfd

- **Purpose**: Integrates with Read the Docs pipeline.
- **Functionality**: Coordinates or triggers Read the Docs builds/checks.
- **Relative Location**: `.github/workflows/ci-rtfd.yml`

______________________________________________________________________

## 2. GitHub Composite Actions

### Agent: cache

- **Purpose**: Facilitates caching of Python dependencies and environments.
- **Functionality**: Stores and restores pip/conda caches to reduce CI overhead and redundant downloads.
- **Relative Location**: `.github/actions/cache`

### Agent: pip-list

- **Purpose**: Provides environment visibility.
- **Functionality**: Prints pip-installed packages and writes a summarized section to the GitHub step summary.
- **Relative Location**: `.github/actions/pip-list`

### Agent: pkg-create

- **Purpose**: Builds and verifies distribution artifacts.
- **Functionality**: Creates source/wheel distributions and checks them with strict validation.
- **Relative Location**: `.github/actions/pkg-create`

### Agent: pkg-install

- **Purpose**: Installs the package in CI jobs.
- **Functionality**: Installs the built package and its dependencies for downstream steps.
- **Relative Location**: `.github/actions/pkg-install`

### Agent: setup-python (standard GitHub Action)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lightning-AI/utilities](https://github.com/Lightning-AI/utilities) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
