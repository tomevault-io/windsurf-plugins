---
trigger: always_on
description: Guidance for AI coding agents, and human contributors, working in this repository.
---

# AGENTS.md

Guidance for AI coding agents, and human contributors, working in this repository.

## What this is

Occurrent is a Maven multi-module JVM event-sourcing library built on CloudEvents. Java 21 baseline, Kotlin coexists in most modules (the root build adds `src/main/kotlin` and `src/test/kotlin`). It ships as small composable libraries rather than a framework: domain models stay independent of Occurrent.

## Module layout

- `test-support`: shared test/domain fixtures.
- `eventstore`: event-store APIs (`api/common`, `api/blocking`, `api/reactor`) and implementations (`inmemory`, `mongodb/native`, `mongodb/spring/blocking`, `mongodb/spring/reactor`).
- `subscription`: subscription APIs (blocking/reactor), Mongo/native/Spring/Redis/in-memory adapters, durable/catchup/competing-consumer wrappers, the CloudEvent `push` models, and the `synchronous` wrapper.
- `cloudevents-extension`: Occurrent CloudEvent stream metadata extensions.
- `common`: shared condition/filter/time/retry/Mongo utility modules.
- `application`: `ApplicationService`/`GenericApplicationService`, command composition, `CloudEventConverter`, CloudEvent type mapping, `command-dispatch` plus its `-annotation` and `-dcb` extensions.
- `dsl`: query, subscription, module, decider, Arrow decider, view, projection, DCB, snapshot, and saga DSLs.
- `framework`: Spring Boot MongoDB starter and annotation support.
- `deadline`: deadline scheduling API plus in-memory and JobRunr implementations.
- `broker`: broker-transport bridges into the push feed (`api` for the shared contracts, `rabbitmq` and `kafka` for the transports, each of those two with its own Spring Boot starter), publishing and consuming CloudEvents and domain events over a message broker without Occurrent depending on any broker client. See ADR 133.
- `library`: higher-level libraries, currently `hederlig`.
- `bom`: published dependency-management BOM.
- `example`: example applications, built by the default-enabled `examples-module` profile.

DCB (Dynamic Consistency Boundary) is a capability layered on the same CloudEvent storage, not a parallel event model. It is shipped for the in-memory, native, and both Spring MongoDB (blocking and reactive) event stores.

## Claiming a GitHub issue before working on it

Several agent sessions run against this repository at the same time, and they cannot see each other. GitHub is the only shared state, so the issue itself is the lock.

Before you start work on an issue, and before you *suggest* an issue to the user as the next thing to pick up, check that nobody else already holds it:

```
gh issue view <N> --json state,labels,assignees,comments
```

Treat the issue as taken if it carries the `in-progress` label, has an assignee, or has a recent claim comment. Say so and pick something else rather than starting in parallel.

If it is free, claim it *before* the first line of work, not after:

```
gh issue edit <N> --add-label in-progress
gh issue comment <N> --body "Claimed by an AI session on <UTC timestamp>, branch \`<branch>\`."
```

The claim is a lease, not a deed. Release it when the work is done or abandoned:

- When a pull request is opened, reference the issue from the PR body (`Fixes #N`) and drop the label. The PR is a stronger, self-updating claim than the label is.
- If you stop without a PR, remove the label and comment that you are dropping it, so the issue does not stay silently blocked.
- A claim with no branch, no PR, and no activity for a day or so is stale. Take it over, but say in a comment that you are doing it.

The check applies to any GitHub task you act on, including issues the user names directly. Claiming is cheap and a duplicated implementation is not.

Three things this protocol has already been caught out by:

**Re-check the claim immediately before your first edit, not only when you start.** A claim check is a point-in-time read, and a long planning pass easily outlives the window in which somebody else claims the issue, so a check that was honestly clean at the start can be wrong by the time you act on it. A session planning #395 read #394 as unclaimed and another session claimed it three minutes later.

**Absence of a branch is not absence of work.** `git ls-tree` and `git branch --contains` only see committed files, so a session with uncommitted work is invisible to every local check. Never conclude an issue is free because nothing in the tree mentions it.

**Picking up one phase of a multi-phase issue means checking the sibling phases too.** The phases live in one list in `.context/ORCHESTRATOR.md` but are tracked as separate issues, and the collision lands on what two phases share rather than inside either one. #394's phase 5 and #395's phase 7 both need the same reactive-only contract shape.

## Architecture Decision Records

ADRs live in `doc/architecture/decisions/`, **not** `doc/adr/`. Filenames are `NNNN-kebab-case-title.md`, numbered sequentially from the highest existing number. Write one for architectural decisions, not for minor implementation details.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johanhaleby/occurrent](https://github.com/johanhaleby/occurrent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
