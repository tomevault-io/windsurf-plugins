---
trigger: always_on
description: Notes for an AI agent working in this repository. Short on purpose: `specs/` is the authority, and this file
---

# Working on pi-dispatch

Notes for an AI agent working in this repository. Short on purpose: `specs/` is the authority, and this file
exists to stop you from having to discover the load-bearing parts by breaking them.

## What this project is

pi-dispatch runs the [pi](https://github.com/earendil-works/pi) coding agent as a service. It **lives in the
background** and, on a cron schedule or a forge event (an issue, a comment, a pull request), opens a
container that runs a flow against a repository, does the work, and shuts the container down. A panel shows
the triggers, the history of every past run and the spend, and can turn the whole thing off.

pi has no queue, no concurrency control, no spend limit and, by its own README, no permission system.
**This project is exactly that missing operational layer and nothing else.** When a change would make
pi-dispatch smarter about *what the agent does*, it probably belongs in a skill or a flow, not here.

Two consequences worth internalising before you design anything:

- **The container is the boundary.** Isolation is built by the worker's own `docker run` argv, so nothing an
  image contains can weaken it.
- **Money is the other boundary.** Every gate that costs nothing runs before every gate that costs
  something, and a paid container starts only after all of them pass.

## The shape

| Path | What it is |
|---|---|
| `worker/` | the queue consumer, the CLI (`pi-dispatch`), the forge hosts, doctor, service installer |
| `receiver/` | the always-on trigger edge (`pi-dispatch-receiver`): webhook routes and the poller |
| `admin/` | the operator console, a pi extension (`/dispatch`), TypeScript, bundled to `dist/` |
| `image/` | the job image and the in-container runner that implements the exit-code protocol |
| `deploy/` | service units, wrappers and the compose file; `worker/deploy/` is the published mirror |
| `specs/` | constitution, requirements, design, interfaces, open questions. The source of truth |
| `docs/` | operator-facing reference, one file per feature or forge |
| `launch/` | launch copy and the demo recording recipe. Maintainer material, deliberately not in `docs/` |

## Read these before changing behaviour

1. `specs/constitution.md` — the non-negotiables. A change that violates one must justify **the
   constraint**, not the code, and amend that file in the same PR.
2. `specs/design.md` — decisions that could have gone another way, and what was rejected.
3. `specs/interfaces.md` — the file and container contracts, including the run-record shape.
4. The revision history at the end of each spec file. It records corrections, not just additions, and
   several entries exist because a previous claim was refuted.

## Rules that bite

- **Specs change in the same PR as the code**, with a revision-history row. When a spec entry is unaffected,
  say so explicitly ("UNCHANGED, checked") rather than silently leaving it. Cite spec IDs
  (`CONST-*`, `REQ-*`, `DES-*`, `INT-*`) in commit bodies; they are permanent addresses.
- **Verify against the pinned artifact, not against HEAD.** pi is pinned to an exact npm version. A sha is
  not a version, and this rule is in the constitution because ignoring it once nearly shipped a runner that
  imported an export the pinned release did not have. Docs are a hint; source at the pin is evidence.
- **`CONST-MERGE-NEVER-AUTOMATIC`.** Nothing in this project merges anything, ever. CI greps
  `worker/src`, `receiver/src` and `image/runner` for `pulls.merge`, `gh pr merge`, `autoMerge` and friends,
  so even a comment mentioning one fails the build.
- **`CONST-BUDGET-BEFORE-TOKENS`.** Free, determinate refusals go before anything that spends: before the
  token mint, the clone, the token-cap read and the budget reservation.
- **`CONST-RETRY-INFRA-ONLY`.** A determinate policy refusal **returns** a result; only infrastructure
  failure **throws** so the queue retries. Getting this backwards means either paying to retry something
  that can never succeed, or dropping real work behind a silent success.
- **Exact pins only** (`CONST-PI-VERSION-PINNED`). No `^`, `~`, `latest` or a floating tag for pi, for
  staged pi packages, or for image bases. A floating range turns an upstream release into every queued job
  quietly losing a tool while the queue still reports success.
- **No secrets or PII in logs.** Log key *names*, never values, and never payload text. The run record is
  PII-free by construction: it holds no attacker-chosen string.
- **Fail loudly, or fail open and say which.** A silent no-op is the worst outcome available here. If a
  feature cannot do what it was asked, it refuses with a reason an operator can act on. Where it fails open,
  the reason is named in the record.

## Style

- **Tabs** in `worker/`, `receiver/`, `image/`. **Two spaces** in `admin/`. Double quotes, semicolons.
  There is no linter, so match the file you are in.
- Node 22.19 or newer.
- Tests are `node:test` with hand-rolled, dependency-injected fakes. No mocking framework, no network, no
  Docker in unit tests. Inject a seam rather than reaching for a global.
- Comments explain **why**, and especially why the obvious alternative is wrong. This codebase is dense with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edgehero/pi-dispatch](https://github.com/edgehero/pi-dispatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
