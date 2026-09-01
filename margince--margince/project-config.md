---
trigger: always_on
description: **This is the only copy of the repository-wide rules.** Every harness reads it:
---

# AGENTS.md — the rulebook

**This is the only copy of the repository-wide rules.** Every harness reads it:
`AGENTS.md` is the convention Codex and the others look for, and Claude Code
reaches it through a one-line `CLAUDE.md` that imports it. Do not add a second
copy — `backend/gates/rulebookdelegation_test.go` fails one.

A directory may carry its own `AGENTS.md` for rules that bind only inside it —
`frontend/AGENTS.md` does. Such a file only ADDS: it states what is true in that
directory and nothing more, never overriding a rule here and never restating one.
A rule that binds the whole tree belongs in this file instead.

**Rules live here; everything else lives in [docs/](docs/README.md).** Every line
here is paid for by every session — the right price for a rule that binds a change,
the wrong one for a procedure, a catalog or an explanation. `cli/craft` feeds this
file's `## Craftsmanship` section into its gate prompt, so that section must stay.
Links point one way: down into `docs/`, never back up.

Margince CRM: the running Go software, its contract, its tests and its docs are
the product, and no separate specification outranks them.

## What decides a question

1. The current request from Lars or the team.
2. Code, tests, migrations and `backend/api/crm.yaml` — what the product does today.
3. Guardrails: security, privacy, agent authority, auditability, public contract
   compatibility, licensing, data durability. Each is enforced by a gate; read the
   gate, because it states the obligation in a form that fails.
4. [docs/](docs/README.md) — how the product is built and operated.

Do not refuse or narrow ordinary product evolution because an older document
disagrees. Name the conflict, say what it costs, and keep going. If the call is
genuinely someone else's, say whose and open an issue labelled
`status: needs-decision`.

The product is **Margince**. Older documents say "Gradion CRM" — same product.

## This repository is public

- **Never name a private repository, document, path or link** — not in code,
  comments, tests, docs, issues, commits or PR bodies. A public contributor must
  be able to follow every instruction here. Write the rule out instead of citing
  somewhere they cannot reach.
- **Never commit a local machine path or a secret.**

`backend/gates/publicreferences_test.go` catches what a test can: a private repo name,
a `specs/` path, a `foundation#NNNN` reference. It does not read commit messages
or PR bodies and has no pattern for a secret — those are your judgement.

A decision number (`ADR-0054`) may appear as a label. Never cite it as though a
reader could open it; the records are not in this tree.

## How you work here

**Fix it, don't file it.** Default: fix what you find, in the same change. Open an
issue only when the fix lives in another module, needs a product or architecture
decision, or would double the diff. Say in the PR body what you fixed along the
way.

**One PR per piece of work.** Do not split related work across several small PRs
— each one costs a full CI run. One branch, one PR.

**Clean up when you are done.** Remove your worktree, delete the branch local and
remote, and stop your dev stack (`make dev-stop`).

**Start in `docs/`.** To learn how something works, read `docs/` first — it is
written for you as much as for a human. The code is still the authority on
current behaviour, so check the code before you rely on a doc for anything a
patch depends on.

**A security hole is never a public issue.** [SECURITY.md](SECURITY.md) routes an
exploitable weakness to a private advisory. The test: if you can write the
reproduction, it belongs in an advisory, not here.

Every issue you do open carries exactly one `priority:` and exactly one `area:`,
plus `status:` when it is not yet workable. Unlabelled means nobody has looked at
it yet, so filing without labels tells the next reader something false. The full
taxonomy: [docs/reference/issue-labels.md](docs/reference/issue-labels.md).

## Build and test

`make check` is the merge gate (`check-backend` + `check-fe`). Run it before you
push. `make test-integration` is the real-Postgres lane and needs `make db-up`;
it fails loudly without a database rather than skipping, because a skipped
security gate looks exactly like a passing one.

**While you iterate, run the narrowest lane that can fail**: `make check-go` for
backend Go, `make check-gates` for a gate under `backend/gates/`, `make check-fe`
or one leg (`fe-unit`, `fe-lint`) for `frontend/`, `make test-it DIR=<pkg>
[RUN=<Test>]` for one integration package. That is the inner loop and never a
substitute — a narrow run proves the part you looked at, not the part you forgot,
and `make check` prints where its time went when you want to make it faster.

All Go lives under `backend/` (one module); the root Makefile delegates there.
Three binaries, all wired through `internal/compose`: `cmd/api`, `cmd/worker`,
`cmd/migrate`.

Commands and flags: [docs/reference/make-targets.md](docs/reference/make-targets.md).
Config and endpoints: [docs/reference/configuration.md](docs/reference/configuration.md).
CI: [infra/ci-pipeline.md](infra/ci-pipeline.md).

## One dev stack per worktree


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [margince/margince](https://github.com/margince/margince) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
