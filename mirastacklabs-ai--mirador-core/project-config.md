---
trigger: always_on
description: This document defines how **agents, tests, and SDKs** must be used when working
---

# AGENTS.md

## 0. Purpose & Scope

This document defines how **agents, tests, and SDKs** must be used when working
on Mirador Core, with a focus on:

- Stage-01 **Correlation & RCA** engine behaviour.
- **Testing procedures** and required `make` targets.
- **GPT5 Mini** (coding agent) and **Raptor** (verification agent).
- **Weaviate** client usage and versioning.
- The **canonical folder** for Correlation/RCA engine work:
  - `dev/correlation-RCA-engine/current`

It is the guardrail for both humans and automation (agents, CI, DevX pipelines).

Key references that MUST be consulted before making changes to the Correlation
and RCA engine:

- `dev/correlation-RCA-engine/current/01-correlation-rca-approach-final.md`
- `dev/correlation-RCA-engine/current/01-correlation-rca-code-implementation-final.md`
- `dev/correlation-RCA-engine/current/correlation-rca-action-tracker.yml` (or the
  project-wide tracker at `dev/correlation-RCA-engine/current/correlation-rca-action-tracker.yml`)
- `dev/correlation-RCA-engine/current/dev-README-correlation-rca.md`

Whenever GPT5 Mini or a human developer works on Correlation/RCA code, these
documents in `dev/correlation-RCA-engine/current` MUST be treated as the
immediate context and source of truth.

---

## 1. Agents

### 1.1 GPT5 Mini – Coding Agent

**Role:** Primary coding assistant.

**Allowed tasks:**

- Generate or refactor Go/Python/Helm/config code for:
  - Correlation & RCA engines,
  - HTTP handlers,
  - EngineConfig wiring,
  - Tests (unit, integration, API).
- Produce documentation drafts, comments, and design notes.
- Propose patches that align with:
  - Stage-01 correlation/RCA design,
  - This `AGENTS.md`,
  - Project style guides.

**Required context for Correlation/RCA work:**

Before generating or modifying any code related to the Correlation or RCA
Engine, GPT5 Mini (and human authors) must conceptually load and respect:

- Approach & algorithmic guide:
  - `dev/correlation-RCA-engine/current/01-correlation-rca-approach-final.md`
- Code-level implementation guide:
  - `dev/correlation-RCA-engine/current/01-correlation-rca-code-implementation-final.md`
- Action tracker:
  - `dev/correlation-RCA-engine/current/correlation-rca-action-tracker.yml`
    or `dev/correlation-rca-action-tracker.yml`
- Usage guide:
  - `dev/correlation-RCA-engine/current/README-correlation-rca.md`

In practice, this means: when designing or changing Correlation/RCA behaviour,
always cross-check with these files in `dev/correlation-RCA-engine/current`
before proposing changes.

**Constraints:**

- Must treat the documents above as **non-negotiable contracts** for Stage-01.
- Must not introduce new API fields for `/unified/correlate` or `/unified/rca`
  beyond `{ "startTime", "endTime" }`.
- For Weaviate access, must only modify or add code within approved
  store/repo layers (see Section 4).

### 1.2 Raptor – Verification & Live Testing Agent

**Role:** Code verifier and test runner.

**Allowed tasks:**

- Run **static** checks (linting, formatting).
- Run **unit** and **integration** tests.
- Execute **API tests** against a locally running Mirador Core.
- Provide structured feedback:
  - Failing tests,
  - Stack traces,
  - Logs,
  - Suggested fix points.

**Required commands:**

Raptor must use these targets when verifying correlation/RCA changes:

```bash
make localdev-test-all-api     # Full checks: code quality + API tests
make localdev-test-api-only    # API endpoint tests only
```

Raptor may run narrower commands (e.g. `go test ./...`) for fast feedback,
but Stage-01 correlation/RCA changes are not considered verified until the two
`make` targets above pass.

**Constraints:**

- Must not auto-merge changes; always produce reports that humans review.
- When invoked for correlation/RCA work (AT-00x items), it must:
  - Confirm time-window-only API contract is respected,
  - Exercise buckets/rings logic,
  - Exercise statistical correlation and narrative output.

---

## 2. Testing Procedures for Mirador Core

Mirador Core uses a comprehensive testing strategy:

- Unit tests (with race detection where applicable)
- Integration tests and E2E tests (full environment)
- Code quality checks:
  - Linting
  - Formatting
  - Vulnerability scanning
- API endpoint tests
- Local development environment tests

### 2.1 Standard Make Targets

For any local code testing, always use 
```bash
make localdev-up
```

To tear down the local build completely, use. MIND YOU WE WILL HAVE TO SEED THE DATA ALL AGAIN
```bash
make localdev-down
```

For only code changes done in `mirador-core`, we will always prefer:
```bash
docker stop mirador-core && docker rm mirador-core && docker rmi localdev-mirador-core && \
docker compose -f deployments/localdev/docker-compose.yaml up -d --build mirador-core
```
This will save a lot of time in terms of not to ship the data again and again

To feed KPIs to mirador-core:
```bash
make localdev-seed-data
```


To send otel synthetic testing data, use the command:
```bash
cd /Users/aarvee/repos/github/public/miradorstack/otel-fintrans-simulator && ./bin/otel-fintrans-simulator --config ./simulator-config.yaml --data-interval 100ms --failure-mode mixed --transactions 50000 --concurrency 2000 --time-window 10m --start-time-offset -15m
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mirastacklabs-ai/mirador-core](https://github.com/mirastacklabs-ai/mirador-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
