---
trigger: always_on
description: This file is the standing brief for an agent run against `ory/awesome-ory`. Read
---

# Keeping the examples in this repo working

This file is the standing brief for an agent run against `ory/awesome-ory`. Read
it before touching anything.

It describes **how to run a maintenance pass** — the contract, the invariants,
and the traps. It is deliberately not a work list: known follow-ups live in
[`TODO.md`](TODO.md), so that this file stays true no matter how much of that
backlog has been done.

## Scope

**In scope: the example projects that live in this tree.** Keep them building,
running, and green against current Ory. Repair and modernize — bump to current,
fix what the bump breaks, and keep each example teaching the thing it was written
to teach.

**Out of scope: the curated link lists** in `README.md` and `ARCHIVE.md`. Those
are other people's repositories; the daily `closed_references` workflow and
manual janitor passes look after them. Do not edit them as part of a maintenance
run.

## Never touch

These files are generated from [`ory/meta`](https://github.com/ory/meta) and
carry a `AUTO-GENERATED, DO NOT EDIT!` header. Edits are silently reverted on the
next template sync, so changing them wastes a run:

```bash
grep -rl "AUTO-GENERATED, DO NOT EDIT" --include="*.md" --include="*.yml" . \
  | grep -v AGENTS.md
```

Grep for the marker rather than trusting a list — `ory/meta` adds files over
time. Today it covers `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`,
most of `.github/` and most of `.github/workflows/`. Two workflows, `format.yml`
and `spellcheck.yml`, carry no marker and are this repo's own. A genuinely new
workflow has to be a new filename.

Note that `CONTRIBUTING.md` tells contributors to run `go test -tags sqlite ./...`
and to make CI pass. That is generic Ory boilerplate; it does not describe this
repo. The contract below is what actually applies.

Also off limits: `assets/`, `LICENSE`, and `git push --force`.

## The test contract

> **`make test` in any project directory exits 0 if and only if that example
> works.** It provisions what it needs, asserts, and cleans up. No credentials,
> no browser, no manual steps.

At the root:

| Command                                     | What it does                                                               |
| ------------------------------------------- | -------------------------------------------------------------------------- |
| `make test`                                 | the whole suite — the default gate                                         |
| `make -C oathkeeper/03-header-mutator test` | one example                                                                |
| `make health`                               | runs everything, tolerates failures, writes `.reports/example-health.json` |
| `make test-network`                         | the credentialed lane; skips loudly without a project                      |

The suite is serial by design — the examples share Docker resources and port
names. **`make -j` is not supported.**

`make test` must leave the working tree clean. `format.yml` gates on
`git diff --exit-code`, so run `make format` before committing.

## How the examples are tested

The examples used to be verified by opening a browser and, for the mutator ones,
reading container logs. They are not any more, and the technique is worth
understanding before you change a test.

`_common/smoke.sh` brings an example's stack up, runs
`_common/smoke/assert.sh` in a container **on the example's own Docker network**,
and tears it down. Two consequences worth knowing:

- **The stack is started with its published ports stripped**, so a run never
  collides with whatever is already listening on the host, and two examples can
  be debugged side by side.
- **The assertion runner sets an explicit `Host` header.** Oathkeeper matches
  access rules against the request URL, and every rule in this repo says
  `127.0.0.1:<port>`. The runner therefore talks to `oathkeeper:4455` while
  claiming to be `127.0.0.1:8080`.

A session is minted without a browser by driving the browser-typed registration
flow with `Accept: application/json` and reading `ory_kratos_session` out of a
curl cookie jar. This works because `_common/kratos/kratos.yml` enables the
`session` hook after password registration. If you ever remove that hook, the
harness has to register and then log in instead.

Each example declares what it expects in a `smoke.env` next to its
`docker-compose.yml`. Adding an example means writing that file, not writing a
new script.

## Invariants

Things a well-meaning change breaks silently:

- **Use `127.0.0.1`, never `localhost`.** `curl localhost:8080` does not match
  the access rules and returns a 404 from Oathkeeper.
- **Entry ports are 8080 — except 08, which is 8000.** Envoy listens on 10000
  and is published on 8000.
- **05–08 are decision-API mode**: their rules have no `upstream:`. That is the
  point of those examples. Do not "fix" it by adding one.
- **nginx rewrites Oathkeeper's 401 into a 302** via `error_page 401`, so in
  05/06 the JSON error handler is only observable on the decision API at `:4456`.
- **Oathkeeper's `redirect` error handler has a `when: accept: text/html`
  clause.** API clients get the `json` fallback and a 401; browsers get the 302.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ory/awesome-ory](https://github.com/ory/awesome-ory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
