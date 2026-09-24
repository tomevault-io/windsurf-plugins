---
trigger: always_on
description: Instructions for coding agents and contributors.
---

# Working in this repository

Instructions for coding agents and contributors.

## Work from Linear

All repository work lives in Linear's `stablemates` workspace, under the `SM` team identifier
and the `Workhorse Development` project. Linear is authoritative for priorities, blockers, and
completion.
Use the connected Linear tools or the hosted workspace at `https://linear.app/stablemates`.
Scope every issue search and creation to this workspace, team, and `Workhorse Development` project;
the `SM` team can contain other projects.

If a request names an exact `SM-*` issue, that issue is the target. If a request names an outcome,
search open issues in `Workhorse Development` for one that owns it. If none matches, create an issue with
checkable acceptance criteria. When asked for the next piece of work, select the highest-priority,
oldest, unblocked Todo issue without an active owner.

Before changing tracked files:

1. Read the target issue, its comments, and linked dependencies. Verify that it belongs to
   `stablemates`, the `SM` team, and the `Workhorse Development` project.
2. Establish ownership through the issue's assignee and move it to In Progress. Re-read the issue
   before starting; if another contributor owns the work, choose another eligible issue or report
   the conflict. Assignment is coordination, not an atomic lease.
3. Read `CONTEXT.md` and relevant decision records when the work changes domain behavior.

For iterative work, especially UI, design, copy, or exploratory changes, keep one review loop on
the original issue. After implementation and verification, keep the issue In Progress until the
requester accepts the result or explicitly asks to finish. Apply refinements toward the same outcome
to that issue; if it was closed prematurely, reopen it. Create a separate issue only when feedback
defines an independently deliverable outcome.

Keep the issue current with comments for material decisions, scope changes, and verification
evidence. Finish only after every acceptance item is verified and relevant repository checks pass.
Record the exact evidence, update the checklist, and move the issue to Done in the same task.
When another contributor can continue the work as-is, record the handoff and move it to Todo.
When a human decision or action is required, record the boundary and move it to Backlog.
Clear your assignment when handing off or waiting. Use the team's configured workflow states
corresponding to these stages.

Linear starts fresh: do not migrate old tickets or translate `WH-*` numbers into `SM-*` numbers.
When following historical issue references in commits or decision records, read
[tracker history](docs/tracker-history.md). Use `SM-*` identifiers for new work and commit subjects.

## Sign agent commits with the model

A commit an agent makes ends with one `Co-Authored-By:` trailer per model that produced the change,
naming the exact model ID. That trailer is the message's only agent attribution: it names no
harness, product, or session.

```
Co-Authored-By: claude-fable-5-1 <noreply@anthropic.com>
```

## Do not run the demo server

Do not start the demo. Not `pnpm demo`, not `pnpm demo:app`, and not a variant in the background.
The dashboard is a data-driven single-page app, so a person with a browser should start it and
assess changes that need visual verification.

A long-lived proxy serves the local demo hostname. If the demo stops, the proxy returns `504
Gateway Timeout` rather than a connection error. Stopping Vite during dependency pre-bundling can
also corrupt `typescript/dashboard/app/node_modules/.vite`; remove that cache and restart the demo
if a running server returns a `504` for a `.vite/deps` chunk.

The demo writes continuously to its database. Run repository commands from the checkout that owns
their data so one checkout cannot change another checkout's test state.

Measuring the demo container is the exception. When an issue asks for its resident set or its CPU
cost, build the image and run it under the limits the deployment gives it. Point it at the
databases of the checkout you work in. Publish it on a port the proxy does not serve, and remove
the container when the measurement ends. Nothing visual comes from that run, so it needs no browser.

## Maintainers own public deployment

Do not run a production setup, deploy, rollback, or container lifecycle command unless a maintainer
explicitly asks for that operation.

Nothing in this repository deploys anything. The live deployment runs from a private operations
repository, and it consumes only `Dockerfile`, `Dockerfile.site`, and what they copy. This
repository once carried parameterized copies of the deployment orchestration, which read as the real
thing and sent work to files that could not affect any deployment; [ADR
0060](docs/decisions/0060-describe-the-deployment-contract-instead-of-shipping-an-example.md)
deleted them. Do not reintroduce a deploy script, a Kamal configuration, or a `.kamal/` directory
here.

If a change affects the public deployment contract, runtime configuration, image publishing, host
prerequisites, or deployment procedure, update `typescript/demo/DEPLOYMENT.md` in the same commit.

## Run commands from the checkout they belong to


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stablemates/workhorse](https://github.com/stablemates/workhorse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
