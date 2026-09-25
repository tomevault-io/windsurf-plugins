---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Dockstore is the web service (backend) component of dockstore.org — a registry for sharing tools and
workflows described in CWL, WDL, Nextflow, or Galaxy, packaged in Docker. It's a leading implementor of
GA4GH's Tool Registry Service (TRS) API. The related Angular UI lives in a separate repo
(`dockstore/dockstore-ui2`), as does the CLI (`dockstore/cli`) — this repo only contains the backend. User-facing
documentation lives in a separate repo, `dockstore/dockstore-documentation`, and the project has a discussion
forum at https://discuss.dockstore.org/.

## Dependency conventions

The webservice is built on Dropwizard — favor Dropwizard's own recommended libraries/patterns for things it has
an opinion on (configuration, health checks, metrics, bundles, etc.) over ad hoc alternatives. Beyond that,
prefer, in order: (1) built-in Java 17/21 features, (2) a third-party library already pulled in via Maven
elsewhere in the project, (3) a new third-party dependency — only reach for a new one when neither of the above
covers the need.

## Branching

The repo follows Hubflow (gitflow) conventions: `develop` is the main integration branch (this repo's default
branch for PRs), with work done on `feature/*` branches (e.g. `feature/http5_aws`) branched from and merged back
into `develop`, `hotfix/*` branches for urgent fixes, and `release/*` branches cut for releases.

## Pull requests

When creating a PR, always create it in draft mode. A human developer must be the one to mark it ready for
review/move it out of draft state — Claude Code should not do this itself.

Always check with the user before pushing changes to GitHub, even to a branch/PR already being worked on in
the conversation — a push can kick off a long CI build or interrupt one that's already running.

When a GitHub MCP server or `gh` is available, diff the current work against `develop` (or whatever branch the
PR targets) and try to minimize stylistic or otherwise-minor changes that inflate the diff and make it harder
to review, unless those changes fix something a Codacy finding or other code-quality check actually flagged.

Keep the freeform "Description" and "Review Instructions" sections brief — one paragraph each, or two for a
genuinely complicated fix, not multi-paragraph writeups. (The "Security and Privacy" checklist section is
separate and must still be copied verbatim per the section below.)

### Using CI and review feedback to guide work

When diagnosing a failing build or iterating on an open PR, pull in whatever signal is actually available
rather than guessing:

- If `gh` or a GitHub MCP server is available, use GitHub Actions build results (check runs, job logs) to
  guide diagnosis and fixes.
- If a CircleCI MCP server is available, use its results (workflow/job status, test failures, logs) to guide
  diagnosis and fixes.
- Codacy findings aren't reliably fetchable through available tooling. If Codacy results seem significant to
  the task, prompt the user to copy-paste them rather than guessing at what Codacy flagged.
- Code review comments left by human developers are high-priority direction — investigate each one and
  propose concrete solutions, even without an explicit instruction to do so. Bot-authored comments (Codacy,
  Copilot Autofix, etc.) are useful but secondary to human reviewer comments.

## Build

This is a multi-module Maven project (Java 21). Always invoke the Maven wrapper (`./mvnw`), never a system-installed
`mvn`, so everyone builds with the project's pinned Maven version.

```
./mvnw clean install                                     # build all modules
./mvnw clean install -Punit-tests                         # build + run only unit tests (fast, no confidential data needed)
./mvnw clean install -Pintegration-tests                  # requires the confidential test data bundle (CI / team members only)
./mvnw clean install -Dtest=SomeClassName test             # run a single test class
./mvnw clean install -Dtest=SomeClassName#someMethod test  # run a single test method
```

Modules (in `pom.xml`, build order matters): `bom-internal`, `dockstore-common`, `dockstore-language-plugin-parent`,
`dockstore-webservice`, `swagger-java-client`, `openapi-java-client`, `dockstore-integration-testing`, `reports`.

`swagger-java-client` and `openapi-java-client` are largely generated (via swagger-codegen-maven-plugin) from
`dockstore-webservice/src/main/resources/swagger.yaml` / the OpenAPI spec — don't hand-edit generated sources under
`generated/`; regenerate by pointing the plugin's `inputSpec` at the updated spec instead. More generally, any
`generated/` directory anywhere in the repo (e.g. `dockstore-common/generated/`, `dockstore-webservice/generated/`)
is build output, not source — this includes the `pom.xml` files under `generated/src/main/resources/`, which are
produced from each submodule's own root `pom.xml`. To change a dependency/version that flows into a generated
`pom.xml`, edit it in `bom-internal` first (the shared bill-of-materials), then in the specific submodule's root

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dockstore/dockstore](https://github.com/dockstore/dockstore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
