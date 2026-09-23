---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

**This is a public, open-source repository** (`github.com/Netflix/mantis`, Apache
License 2.0). Everything committed here — code, comments, test fixtures, commit
messages, and pull request text — is world-readable and permanent. Read
[Publishing a pull request](#publishing-a-pull-request) before opening a PR.

## Reference docs

Longer-form context lives in the repo and should be consulted before making
non-trivial changes:

| Doc | Use it for |
|---|---|
| [`.claude/test-cheatsheet.md`](.claude/test-cheatsheet.md) | Testing cheat sheet: gradle test commands, where reports land, common mock/test failure patterns, key test helper files |
| [`context/context-unit-test-learnings.md`](context/context-unit-test-learnings.md) | Principles for refactoring and extending tests around actor/worker state |
| [`context/context-te-state-sync.md`](context/context-te-state-sync.md) | Task Executor ↔ control plane state sync behavior and its failure modes |
| [`context/inter-job-stages-connection-mechanisms.md`](context/inter-job-stages-connection-mechanisms.md) | Stage-to-stage (intra-job) and job-to-job connection mechanisms |
| [`context/reservation-scheduling-context.md`](context/reservation-scheduling-context.md) | Reservation-based scheduling across master, scheduler, resource cluster, and scaler |
| [`context/scaling-optimization-analysis.md`](context/scaling-optimization-analysis.md) | Scaling impact analysis: downstream disruption and connection timing gaps |
| [`context/archive/`](context/archive/) | Superseded design/plan docs; historical intent only — verify against current code |

Start with `.claude/test-cheatsheet.md` for anything test-related. These docs are
snapshots of intent at the time they were written; when a doc and the code
disagree, the code wins — and say so in your change.

Also see [`README.md`](README.md) (build/release), [`CONTRIBUTING.md`](CONTRIBUTING.md),
the user-facing docs site sources under [`docs/`](docs/), and the templates in
[`.github/`](.github/).

## Build and test

JDK 17 (CI builds and tests on Java 17 only). Gradle wrapper 8.11.1 — always use
`./gradlew`.

```sh
./gradlew clean build          # compile + standard tests
./gradlew clean test           # standard unit tests
./gradlew akkaTest             # Akka actor tests (see below)
./gradlew build akkatest       # what CI runs (.github/workflows/nebula-ci.yml)
```

The root `build.gradle` splits tests into two suites, so **`test` alone does not
run everything**:

- `test` — excludes any class matching `*AkkaTest`.
- `akkaTest` — only classes matching `*AkkaTest`, with `maxParallelForks = 1`.

Run both before claiming tests pass. Narrow a run with
`--tests "*.SomeTest.someMethod"`, and add `--rerun-tasks` to bypass Gradle's
up-to-date checks.

Docker is required for tests that use Testcontainers (`mantis-testcontainers`).

## Repository layout

Modules are declared in `settings.gradle`. The ones changes usually touch:

- `mantis-control-plane/` — control plane `client`, `core`, `server`, `dynamodb`.
  Most scheduling, job, and resource-cluster logic lives in `-server`.
- `mantis-server/` — `mantis-server-agent` (Task Executor) and
  `mantis-server-worker-client`.
- `mantis-runtime/`, `mantis-runtime-executor/`, `mantis-runtime-loader/`,
  `mantis-runtime-autoscaler-api/` — job runtime and autoscaler API.
- `mantis-client/`, `mantis-common/`, `mantis-common-akka/`, `mantis-common-serde/`,
  `mantis-network/`, `mantis-remote-observable/`, `mantis-rxcontrol/` — shared libraries.
- `mantis-publish/`, `mantis-connectors/`, `mantis-source-jobs/` — publish client,
  connectors (Iceberg, Kafka, publish, job-source), and source jobs.
- `mantis-examples/` — runnable sample jobs; keep these working, they are the
  onboarding path for new users.
- `docs/` — MkDocs sources for the public documentation site.

Follow the conventions of the file you are editing (`CONTRIBUTING.md`), keep the
Apache license header on new source files, and prefer adding a failing test that
reproduces a bug before fixing it.

## Publishing a pull request

This repository is public. A commit or PR body that leaks internal information
cannot be truly retracted: it survives in forks, mirrors, notification emails,
and the GitHub API even after a force-push, edit, or delete. Treat the
public/internal boundary as a hard gate, not a style preference.

The rule: **a PR here must stand entirely on public, reproducible evidence.**
A reader with no access to any internal system must be able to understand the
motivation, verify the reasoning, and reproduce the results using only this
repository and other public sources.

### Never include

- **Internal document links or references** — internal wikis, knowledge bases,
  intranet pages, internal Google Docs/Sheets/Slides/Drive links, internal
  design or planning docs, internal short/vanity links, internal issue tracker
  URLs or ticket IDs, internal code-host URLs, or internal repository and
  service names.
- **Chat and meeting context** — Slack (or any internal chat) channel names,
  message permalinks, quoted threads, screenshots of conversations, DM content,
  internal user handles, and phrasings like "as discussed in the channel" or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Netflix/mantis](https://github.com/Netflix/mantis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
