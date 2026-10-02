---
trigger: always_on
description: All product, domain, data-model, code, API, UI, security, workflow, and architecture decisions in this repository must meet financial-institution-grade enterprise ERP standards. Prefer financial integrity, explicit controls, auditability, deterministic behavior, and long-term operability over implementation convenience.
---

# Repository Engineering Standards

## Financial-institution-grade ERP standard

All product, domain, data-model, code, API, UI, security, workflow, and architecture decisions in this repository must meet financial-institution-grade enterprise ERP standards. Prefer financial integrity, explicit controls, auditability, deterministic behavior, and long-term operability over implementation convenience.

At minimum, designs and implementations must preserve:

- strict organization and legal-entity isolation;
- balanced, deterministic, idempotent accounting and posting behavior;
- immutable posted history, with corrections performed through controlled reversals or adjusting entries;
- complete audit evidence for material configuration and transactional changes, including actor, timestamp, before/after state, and reason where appropriate;
- explicit lifecycle states, transition rules, approvals, permissions, segregation of duties, and safe concurrency controls;
- effective-dated configuration where changing a rule could otherwise reinterpret historical transactions;
- precise decimal and currency handling with no floating-point financial arithmetic;
- enforced invariants and feature dependencies at the domain/service and API boundaries, not only by hiding UI;
- backward-compatible migrations, preserved tenant data, and reversible operational rollout plans;
- clear ownership and a single source of truth for every financial policy and configuration value.

Do not introduce silent financial fallbacks, ambiguous overlapping configuration, UI-only enforcement, destructive feature toggles, or parallel sources of truth.

## A refusal that is computed must be raised

This repository's most common defect is not a wrong answer. It is a CORRECT
answer that never reaches anyone. The code detects the problem, composes an
accurate message naming the remedy, and then the refusal is dropped on the way
out — so the operator sees success, or silence, or an error about something
else entirely. Six instances were found in six unrelated subsystems in a single
night:

- a partially-refused pay run committed and posted anyway;
- an unconfigured SUI rate accrued 0.00 instead of refusing by name;
- a mis-scoped statutory rate saved, reported `{ok}`, and resolved to `null`,
  so the engine priced the levy as unconfigured forever;
- one country's payroll component silently absorbed another's via
  `on conflict do nothing`, and each remap erased the other's;
- a New York certificate could be filed against a California employee, so the
  employee withheld by the wrong state's table;
- a delete refusal was delivered as a 500 and the client called `res.json()`
  before checking `res.ok`, so the operator read a JSON parse error and no
  human has ever seen the message.

The rules that follow are not style. Each one is a place a refusal was lost.

- **A write that matches zero rows is a failure, not a success.** Check the
  affected row count and throw. Under RLS an unscoped `UPDATE`/`DELETE`
  silently matches nothing and reports success — the most dangerous shape here.
- **Never report `{ok}` for work whose effect no read can observe.** If a save
  stores a row that resolution cannot find, the save was not a save.
- **`on conflict do nothing` must be justified in a comment or not used.** It
  is the quietest way to drop a write. Say why a conflict is expected and
  benign, or handle it.
- **Fail closed on unconfigured inputs that are always owed.** Prefer refusing
  by name over accruing zero. Zero is indistinguishable from "correctly nil".
- **Validate against the subject, not only the declaration.** A form scoped to
  a jurisdiction must be checked against the EMPLOYEE's jurisdiction, not only
  against its own metadata.
- **Error bodies are checked before they are parsed.** `if (!res.ok)` first,
  always; `await res.json()` on an error path turns a refusal into a parse
  error.
- **Prose asserting a guarantee is a claim about code.** A comment saying a
  lock prevents X, or a message saying "do X instead", is load-bearing: it is
  the only evidence a reader has that the mechanism exists. Verify it against
  the code, and when the code changes, THE CLAIM IS PART OF THE CHANGE. These
  survive review because a claim about an ABSENT mechanism contradicts nothing
  — there is no code to compare it against, and absence is invisible. That is
  why "review more carefully" does not catch them: the reviewer is checking for
  contradiction, and there is none to find. Confirm the mechanism exists.
- **A refusal must name the remedy, and the remedy must exist.** This is the
  sharpest case of the rule above, because a user ACTS on a remedy: a wrong
  comment misleads the next author, but a wrong remedy makes the operator
  destroy the thing the refusal was protecting, holding the product's own
  instructions. Before writing "do X instead", read the code that does X.

Corresponding test rules, because every one of the above was green somewhere:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [braedonsaunders/openbooks](https://github.com/braedonsaunders/openbooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
