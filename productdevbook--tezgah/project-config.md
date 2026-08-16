---
trigger: always_on
description: A commerce engine as a Rust library. Read `README.md` first — it carries the
---

# Working in tezgah

A commerce engine as a Rust library. Read `README.md` first — it carries the
decisions and the reasons, and this file does not repeat them.

## This repository is public

Everything here is readable by anyone, forever: code, comments, tests, commit
messages, fixtures. Nothing about whoever runs it goes in. No customer names,
addresses, hostnames or e-mails; nothing out of a live database; no
credentials; no server addresses. Test data uses the reserved domains —
`example.com`, `example.test`, `example.invalid` — and nothing else.

tezgah is a library and does not know who is using it. Keep it that way: no
host's name in the code, and no feature shaped for one caller.

## Nothing is built or tested on the machine this was written on

That machine serves other people's sites and a build taking every core has
taken it off the air before. Do not run `cargo build`, `cargo test`,
`cargo nextest run`, `cargo clippy` or `cargo check` there. Branch, write, commit, open the pull
request, and read what CI says. `.github/workflows/ci.yml` runs the formatter,
clippy with warnings denied, the tests against a real Postgres, the doctests,
a dependency audit and a secret scan.

## What the code has to keep true

**Ports ask, they do not answer.** `src/ports.rs` is the whole of what tezgah
wants from a host. Adding a trait there is a real decision — it becomes work
for everybody embedding this. Prefer a parameter.

**Every public function that reaches data asks first.** `ctx.permit(..)` puts
the question to the host's `Authorizer`, and a denial is an error rather than a
`false`. This is a convention with a test behind it rather than a type the
compiler makes you carry: no function takes a `Permit` as a parameter, and
`tests/permit_asked.rs` reads `src/` and fails when a public function runs a
query with no `ctx.permit(..)` above it and no reason in its `TOLERATED` list.
If a code path does not need permission, say so there, where a reader can see
it.

"Public" there means reachable from outside the crate: `pub`, not
`pub(crate)`. A host never calls a `pub(crate)` function directly, so asking
it to hold a permit is asking twice — the crate-external entry point it sits
behind already asked. Keep it `pub(crate)` on purpose, not `pub`, if the only
reason it is visible past its module is another function in this crate.

**A workflow step can say it had nothing to do.** `workflow_step.state` permits
`'skipped'` for exactly this: `Outcome::skipped(output)` carries the step's
input forward the way `Outcome::new` does, records `state = 'skipped'`
instead of `'done'`, and the run does not call that step's `compensate` when
it later unwinds — a skipped step wrote nothing, so there is nothing to undo.
Reach for it only when a step's behaviour is genuinely conditional — spending
credit a cart does not have, authorizing a charge for nothing once credit
covered the total — not as a way to make a step's return type more
interesting.

**Audit rows, events and jobs are written in the caller's transaction.** Never
after the commit. A change that rolls back takes them with it, and an event
that was never delivered is still in the outbox to deliver.

**Money is `Money`.** No `f64`, no minor-unit integers, no multiplying by a
hundred. An allocation across lines must add back up to the whole, and there
is a test that says so.

**Every table carries a scope and has row-level security forced on.** Not
enabled — forced, so a table owner does not bypass it. A migration adding a
table without both fails the schema test.

**Amounts, quantities and state transitions belong to the database too.** Check
constraints, not comments. Two writers always turn up.

**A migration is append-only, so a bug in one cannot be edited away — but it
can be corrected.** `tests/migration_dml.rs` reads a migration's own text, and
a bad backfill sits there forever even after a later migration fixes the rows
it left wrong. Its `TOLERATED` list distinguishes the two: an entry names the
migration that corrected it, and the test checks that migration is still in
the tree. A hole nobody has dealt with yet cannot cite one — that is what
keeps the list from becoming a place fixed bugs go to be remembered as open
ones.

## Mistakes this codebase has made more than once

Every one of these was found by running the code or counting its callers, never
by reading it and finding it wrong. They are written down because each has
recurred, and because each is invisible in review.

**A number scoped to a part, used against the whole.** A capture's slice and the
order's total in one expression; a fixed fee clamped against one line when it is
defined against the order. Identical whenever there is exactly one part, wrong
the moment there are two — so every test that captures in full passes. When an
expression mixes two totals, ask which one each came from.

**Written, tested, and reachable from nothing.** Five features shipped this way:
correct modules with no route, no caller, and no way for a shop to touch them.
`tests/reachable.rs` catches it now. A domain function without a route is not
finished, and neither is a table only tests write to.

**A constraint left out because a fixture could not satisfy it.** Twice. The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [productdevbook/tezgah](https://github.com/productdevbook/tezgah) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
