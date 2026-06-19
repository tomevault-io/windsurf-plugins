---
trigger: always_on
description: <!-- GENERATED FILE — do not edit by hand.
---

<!-- GENERATED FILE — do not edit by hand.
     Regenerate: npm run generate:skill   (verified against the ajisai CLI)
     Source of truth for semantics: SPECIFICATION.html.
     Generator: scripts/generate-skill-md.mjs -->

# Ajisai — Agent Writing Protocol (SKILL.md)

How to *write working Ajisai on the first try*. Every code line below was
executed by the generator against the real interpreter; results shown are
actual outputs. **If a word is not in the §9 table, it does not exist — when
unsure, grep §9 before writing.**

## 1. Run loop

```sh
ajisai run program.ajisai --json     # exit 0 = ok, 1 = language error, 2 = usage
ajisai check program.ajisai --json   # parse + resolve only, no execution
```

Read the JSON in this order (contract: docs/dev/agent-cli-output-contract.md):
1. `status` / exit code. On ok: `stackDisplay` (final stack, bottom→top) and `output` (PRINT lines).
2. On error: `diagnosis.why` + `diagnosis.where` locate the failure; follow `diagnosis.nextChecks` in order; `aiDiagnostic.recoverability` says what kind of change fixes it (`fixProgram` / `fixInput` / `fixHost` ...).
3. Even on ok, scan `errorFlowTrace` for `nilProduced` events if a NIL surprised you.

## 2. Minimal syntax

- Postfix, stack-based. Operands first, word last: `[ 1 ] [ 2 ] +`.
- Numbers are **exact rationals** (`1/3`, `3.14` → 157/50). No floats. Display shows `3/1` for 3.
- Data lives in vectors: `[ 1 2 3 ]`. Nest for tensors: `[ [ 1 2 ] [ 3 4 ] ]`. A lone number like `42` is allowed but `[ 42 ]` is the idiomatic scalar.
- Strings: `'single quotes'` (a codepoint vector with text role). Booleans: `TRUE` / `FALSE`. Absence: `NIL`.
- Code blocks: `{ ... }` — quoted programs passed to MAP / FILTER / FOLD / COND / DEF.
- User word: `{ body } 'NAME' DEF` then call `NAME`. Words are case-insensitive (canonicalized to upper case).
- Comments: `#` to end of line.
- Modifiers prefix the *next word only*: `,,` (KEEP: don't consume operands), `..` (STAK: apply to whole stack), `,` (EAT, default), `.` (TOP, default).
- One word does one thing to the stack; there are **no** DUP/SWAP-style shufflers (§8).

## 3. Control and iteration

- Branch: `value { guard } { body } { guard } { body } ... COND`. Guards see the value (it stays for each guard) and must leave TRUE/FALSE; use `{ TRUE }` as the final else-guard. The value remains on the stack after COND.
- Iterate data, not counters: `MAP` / `FILTER` / `FOLD` / `SCAN` / `UNFOLD` with `{ }` blocks (examples in §6). `FOLD` requires an explicit `[ init ]`.
- Predicates: `ANY` / `ALL` / `COUNT` with a `{ predicate }` block.
- Recursion is allowed in user words (execution-step and depth limits apply; exceeding them is a diagnosed error, not a hang).

## 4. NIL — absence is a value, not an exception

Failed partial operations *bubble*: `[ 1 ] [ 0 ] DIV` succeeds (exit 0) and
pushes `NIL` (reason: `divisionByZero`). The projection is recorded in
`errorFlowTrace` as a `nilProduced` event with a full diagnosis, and the NIL
value itself carries `semantics.absence.reason` on the stack.

- Provide a fallback with `^`: `[ 1 ] [ 0 ] DIV ^ [ 99 ]` → stack `[ 99/1 ]`.
- NIL flows through later operations (bubble rule); check for it where it matters instead of letting it propagate to the end.

## 5. UNKNOWN — the third truth value

Comparisons of lazy exact reals are *budgeted*. When the budget is exhausted
without a decision, the result is the logical `UNKNOWN`, not an error and not NIL:

```ajisai
'MATH' IMPORT
2 SQRT 8 SQRT 2 DIV 3 COMPARE-WITHIN   # √2 vs √8/2 within 3 partial quotients
```

→ stack `UNKNOWN` (exit 0). In JSON the value serializes as
`{ "type": "truthValue", "value": "unknown" }` and carries
`agreedPrefix: 3` (leading partial quotients that matched) in
`semantics.absence.diagnosis`. Raise the budget or restructure the comparison
to decide; AND/OR/NOT follow Kleene three-valued logic over UNKNOWN.

## 6. Canonical examples (all verified by the generator)

- Push a number (always inside a vector)
  `[ 42 ]` → stack: `[ 42/1 ]`
- Exact rational division — no floats, ever
  `[ 1 ] [ 3 ] /` → stack: `[ 1/3 ]`
- Elementwise vector arithmetic
  `[ 1 2 3 ] [ 4 5 6 ] +` → stack: `[ 5/1 7/1 9/1 ]`
- Scalar broadcast over a vector
  `[ 5 ] [ 1 2 3 ] *` → stack: `[ 5/1 10/1 15/1 ]`
- Remainder
  `[ 10 ] [ 3 ] %` → stack: `[ 1/1 ]`
- Comparison pushes a boolean
  `[ 1 ] [ 2 ] <` → stack: `TRUE`
- Range: one vector [ start end ] (inclusive)
  `[ 0 5 ] RANGE` → stack: `[ 0/1 1/1 2/1 3/1 4/1 5/1 ]`
- Range with step: [ start end step ]
  `[ 0 10 2 ] RANGE` → stack: `[ 0/1 2/1 4/1 6/1 8/1 10/1 ]`
- Fill a tensor: [ shape... value ]
  `[ 2 2 7 ] FILL` → stack: `[ [ 7/1 7/1 ] [ 7/1 7/1 ] ]`
- Reshape
  `[ 1 2 3 4 5 6 ] [ 2 3 ] RESHAPE` → stack: `[ [ 1/1 2/1 3/1 ] [ 4/1 5/1 6/1 ] ]`
- MAP with a { } code block
  `[ 0 4 ] RANGE { [ 2 ] * } MAP` → stack: `[ 0/1 2/1 4/1 6/1 8/1 ]`
- FILTER keeps matching elements
  `[ 0 10 ] RANGE { [ 5 ] > } FILTER` → stack: `[ 6/1 7/1 8/1 9/1 10/1 ]`
- FOLD needs an explicit initial value
  `[ 1 2 3 ] [ 0 ] { + } FOLD` → stack: `[ 6/1 ]`
- ANY / ALL / COUNT take predicate blocks
  `[ 1 2 3 ] { [ 1 ] > } ANY` → stack: `TRUE`
- Define a user word: { body } then name, then DEF
  `{ [ 1 ] [ 2 ] + } 'MY-SUM' DEF MY-SUM` → stack: `[ 3/1 ]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masamoto1982/Ajisai](https://github.com/masamoto1982/Ajisai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
