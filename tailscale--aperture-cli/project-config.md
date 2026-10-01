---
trigger: always_on
description: Domain-driven design is the primary design paradigm for this project. New work
---

# aperture-cli

## Design

Domain-driven design is the primary design paradigm for this project. New work
that introduces or reshapes a domain concept is modelled before it is written.

The domain model, the data model and the contracts are one first step, not
three stages. A domain model without its contracts is a description; contracts
without a model have nothing to be complete about. Define them together, before
the code, and let each correct the other.

- Bounded contexts are sized by language, not by responsibility. Splitting a
  context because two halves feel like different jobs is the usual mistake;
  if the user experiences one thing, it is one context.
- `domain` is never a package name. Packages and types are named after the
  thing they are, by what they do in this program rather than by their
  technical role. `Machine`, not `NodeManager`. Where the thing already has a
  name in the system it wraps, take that name: a tailnet node registered by
  `POST /machine/register` is a `Machine`.
- Every domain object is classified entity, value object or enumeration, and
  every field is listed. Behaviour lives on the object.
- Vendor types never appear in domain signatures. `tsnet`, `ipn` and
  `ipnstate` are confined to `internal/bridges`, which is the anti-corruption
  layer for the tailnet.

Artifacts, written before the code:

- `docs/specs/<context>-context-map.md` — ubiquitous language, contexts,
  relationships, ambiguous terms.
- `docs/specs/<context>-domain-model.md` — one section per object, with
  fields, behaviours, invariants, states and relationships.
- `docs/specs/<context>-contracts.md` — the API, the domain events and the data
  model, each to 100% and cross-checked against each other, so every aggregate
  transition can be traced through all three. A contract this project does not
  have is recorded as absent with its reason, never left blank.
- `docs/adr/NNNN-<slug>.md` — the decision and the forcing reason.

Mermaid diagrams in those files are rendered before the commit that adds them.

### Writing them

Progressive disclosure. Every document answers in its first paragraph and
deepens from there, so a reader who stops early still leaves with the decision.
Detail belongs in the spec; the ADR links to it.

An ADR is one page, Nygard's shape with the reason made explicit:

```
# NNNN. Title            <- the decision, not the topic
Status / Date
## Why?                  <- the concrete failure, never the category
## Decision              <- numbered, each one testable
## Consequences          <- what this costs, not what it wins
## Rejected              <- option, then the cost of taking it
## Revisit when          <- the condition that reopens this
```

An ADR longer than that has a spec trying to get out of it. Both are read by
someone with the code in front of them, so neither restates what the diff
already says.

Current: [Connection](docs/adr/0001-connection-bounded-context.md).

## Conventions

- Commit prefixes match the package touched: `tui:`, `bridges:`, `config:`.
- `make check` is the gate, and it is what CI runs: lint, build, then the suite
  under the race detector. `make test` is the fast loop, not the bar.

## Workflow

- Never commit to `main`. Every change gets a branch.
- Prove a bug with a failing test first: reproduce it, watch it fail, fix to
  green. No harness at that layer means adding the smallest one and wiring it
  into `make check`.
- Test through the real path before calling it fixed. Here that is the built
  binary in a terminal, not only `go test`. State what was verified and what
  could not be.
- Reply to every addressed PR comment with the commit that fixed it: backticked
  short hash, linked. No "done" without a hash.
- Commit bodies and ADRs carry the why, because the what is in the diff: the
  concrete failure, what the obvious alternative would have cost, and what
  would justify revisiting. Same for PR descriptions.
- Security-review any diff touching login links, tailnet identity, credentials
  or the bridge state directory before it merges, as a fresh-context
  adversarial pass by someone other than the author. Verify each finding; the
  build is the arbiter.
- Documents go in content-typed paths (`docs/specs/`, `docs/adr/`), never in a
  path named after whatever produced them.

## Writing the code

Least code that solves it. Before writing any, in order: does it need to exist
at all (skip it, and say so), is it already here (reuse it), does the stdlib do
it, does a dependency already in `go.mod` do it, can it be one line, and only
then the minimum new code. A new dependency has to be maintained and
license-compatible, no GPL/AGPL; name the one chosen, or why none fit.

- No abstraction nobody asked for: no interface with one implementation, no
  wrapper around a single call, no config for a value that never changes.
- Deletion over addition. Boring over clever.
- Never grow a God object. When the natural home for new state is the struct
  everything already hangs off, that is the signal to give it its own type.
  Tells: unrelated field clusters, methods that ignore most of the fields, a
  name that is a role rather than a thing, tests that cannot construct it
  without stubbing the world.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tailscale/aperture-cli](https://github.com/tailscale/aperture-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
