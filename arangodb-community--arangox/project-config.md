---
trigger: always_on
description: enables both so plain `docker compose up` is unchanged.
---

# arangox — agent notes

An Elixir driver for ArangoDB, implemented on `DBConnection` (Elixir's pooled
database-connection behaviour). The default transport is HTTP via Mint;
VelocyStream (ArangoDB's binary protocol, removed by the server in 3.12) is an
explicit opt-in for 3.11 deployments via `client: Arangox.VelocyClient`.

## Current work

A breaking modernization is in progress on `feat/v0-8-modernization`, shipping
as **0.8.0** — the plan's "1.0" names this release (KD13). 1.0 itself is
reserved for the later release that replaces DBConnection with
`http_connection`. The implementation plan at
`docs/plans/2026-08-06-001-feat-arangox-1-0-modernization-plan.md`
is the authority for that work — requirements (R-numbers), key decisions
(KD/KTD-numbers), and implementation units (U-numbers) are all defined there.
Decisions marked `session-settled` were made with alternatives in view; a real
defect found inside one still gets surfaced, but the preference itself is not
reopened without evidence.

## Code comments

Comments are for whoever has to change the code next, not for whoever reviews
the diff. Write what constrains the code, what breaks if it changes, and what
cannot be seen by reading it — a protocol rule, a required ordering, a
surprising return shape, a reason an obvious simplification is wrong. Then stop.

Do not narrate history ("this used to throw", "the old version pinned"), argue
that a decision was correct, restate what the code plainly says, or close on a
rhetorical flourish. That material belongs in the commit message, where it is
addressed to someone reading history deliberately. Plan identifiers (R-, KD-,
KTD-, U-, AE-, Q-numbers) must never appear in code, comments, docstrings,
test names, or commit messages — they reference internal planning documents a
library user cannot read. State the constraint itself in plain words instead.

The same applies to test comments. A `describe` block may say what question the
block answers; individual tests should not re-argue it.

## Tests

Three tiers:

- `mix test` — unit and protocol tiers, no Docker needed. The protocol tier
  runs against `Arangox.ProtocolServer` (`test/support/protocol_server.ex`),
  a local harness that can serve arbitrary routes, hang, truncate, speak TLS,
  and record what reached it. Read its moduledoc before writing socket-level
  tests.
- `mix test.integration` — needs the containers from `docker-compose.yml`
  (`docker compose up --detach --wait`). Its preflight probe expects the full
  stack; `ARANGOX_SKIP_DOCKER_CHECK=1 mix test --only integration <file>`
  runs a subset against whatever is up. Integration tests carry a *valued*
  tag — `integration: true` (3.12 tier) and `integration: :arango_3_11` (the
  3.11 trio) — so CI legs select by value (`--only integration:arango_3_11`)
  while the bare `--only integration` behind `mix test.integration` matches
  both. A test must carry the tag of the server line whose service it reaches,
  not the one whose feature it is about: each CI leg starts a single compose
  profile, so a test tagged for the other line finds nothing listening.
  Compose profiles mirror the split (`3.12`/`3.11`); the committed `.env`
  enables both so plain `docker compose up` is unchanged.

Container topology (host ports): `single_no_auth` 8529 and `single_auth`
8001/8002 (TLS) on 3.12; `resilient_single` 8003–8005, a 3.11 active-failover
trio deliberately pinned to 3.11 (VelocyStream and active failover are
3.11-only concerns); `cluster` 8006–8008, three coordinators for
cross-coordinator behavior (stream-transaction identity, leader questions).

Hygiene expected before claiming work done: `mix test`,
`mix compile --warnings-as-errors`, `mix format --check-formatted` on touched
files, and red-before-green evidence for behavior-bearing changes.

## Documented solutions

`docs/solutions/` holds documented solutions to past problems (bugs,
architecture patterns), organized by category with YAML frontmatter
(`module`, `tags`, `problem_type`). Relevant when implementing or debugging
in areas it covers — currently DBConnection mechanics (timeouts, deadlines,
callback processes), transaction-state ordering, and URL construction in the
API surface. These ship with the library; see _Publishing to origin_ for what
that requires of them.

`CONCEPTS.md` at the repo root is the shared domain vocabulary — entities,
named processes, and status concepts with project-specific meaning ("Client"
means a transport implementation here, not a consumer). Relevant when
orienting or naming things.

## Generated code: errno

`lib/arangox/errno.ex` is generated — edit `priv/arangodb/gen_errno.exs` and
regenerate (`mix run priv/arangodb/gen_errno.exs`); never hand-edit the
module. The source table `priv/arangodb/errors-3.12.10.dat` is vendored and
SHA-pinned (see `priv/arangodb/README.md`). Its tag is the driver's single
version pin: `Arangox.Errno.tag/0`, which must equal the compose default in
`docker-compose.yml` — the conformance gate asserts it against the live
server before comparing anything.

## The API surface

`lib/arangox/api/` is owned, hand-maintained source. It was derived from
ArangoDB's OpenAPI description once, but no generator stands behind it and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArangoDB-Community/arangox](https://github.com/ArangoDB-Community/arangox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
