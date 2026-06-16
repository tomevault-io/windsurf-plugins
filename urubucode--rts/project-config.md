---
trigger: always_on
description: **Before starting ANY task, you MUST read this CLAUDE.md in full and follow
---

# CLAUDE.md

## RULE #0 — MANDATORY ABSOLUTE META-RULE

**Before starting ANY task, you MUST read this CLAUDE.md in full and follow
ALL rules it defines — no exceptions, no omissions, no "picking the important
ones". Every rule in this file is binding.**

This is the first and most important rule. It governs all others.

### How to apply

1. On the first message of each session (and whenever this file changes), read
   `CLAUDE.md` end to end before touching code.
2. Each `## MANDATORY RULE:` section is binding even when the task context seems
   not to require it.
3. Each `## Conventions`, `## Rules`, `## ABI ...`, `## Structure ...` section
   defines conventions that must be respected in any code change.
4. If a rule conflicts with a user instruction, ask for confirmation before
   violating the rule. Do not decide alone.
5. If a rule is stale (code no longer matches), update CLAUDE.md in the same PR —
   never leave a lying rule in effect.

### Current mandatory rules in this file

- **RULE #0** (this) — read and follow everything
- **MANDATORY REQUIREMENT: local-rules.md** (check and read if it exists)
- **MANDATORY RULE: REGRESS WHEN NECESSARY (EXPLICITLY)**
- **MANDATORY RULE: FOLLOW `../ROADMAP-CORRECAO.md`** (topological order for
  cross-runtime bug fixes; detailed in `.claude/rules/00-meta.md`)
- **CROSS-RUNTIME PUSH MODE (parity ≥ 90%)** — while active, the process
  constraints above are suspended to push parity to 100%; the honesty + build
  floor never lift

Keep this list in sync with the sections below.

## MANDATORY REQUIREMENT: local-rules.md

Before starting any task, you **MUST** check whether `local-rules.md` exists at
the project root. **If it exists, reading it is mandatory** — not optional, do
not skip, do not assume content, do not proceed without reading. If it does not
exist, proceed normally. When present, treat its content as additional rules set
by the developer working on this local copy; they take priority over generic
preferences for the whole session. `local-rules.md` is per-developer and is not
versioned (already in `.gitignore`).

## MANDATORY RULE: REGRESS WHEN NECESSARY (EXPLICITLY)

Regression is allowed when necessary — but it must **always be explicit and
justified**, never silent. This replaces the old "zero regression" rule.

Minimum suite before merge:

```bash
cargo build --release             # clean build
cargo test --release --lib        # unit + integration
target/release/rts.exe test       # TS suite (if PR touches runtime/codegen/GC)
```

### Practical rules

- **Run the full suite before merge.** You must know exactly which tests pass and
  which regress. "It broke and I don't know why" is never acceptable.
- **A regression is acceptable only when** (a) it is intentional (changed
  behavior / removed feature) or a necessary tradeoff for the change, **and**
  (b) it is documented explicitly in the commit/PR with justification.
- **Silent or unexplained regression still blocks merge.** Each regressing test
  must be either updated to the new expected behavior, or listed explicitly as a
  known regression with reason + tracking issue.
- **A broken build blocks merge** unless explicitly justified in the same PR.
- **Codegen tests/fixtures (`tests/*.test.ts`, `tests/fixtures/*`) are part of
  the suite.** If behavior changed on purpose, update them and justify.
- **Large multi-area PRs run the suite incrementally** during development, not
  only at the end.

### Why this rule exists

With 2 devs + AI accelerating velocity, the danger is *silent* regression
piling up until the suite becomes a lie (green tests, broken uncovered paths).
The discipline here is not "never break a test" — it is "never break a test
without knowing and saying so". Explicit, justified regression is fine;
invisible regression rots the project.

## CROSS-RUNTIME PUSH MODE (parity ≥ 90%) — process constraints suspended

**Active when** cross-runtime parity ≥ 90% (currently 94.3%, badge in
`README.md`). The goal flips to reaching **100%**, and the *process* constraints
below are SUSPENDED so change can land at any cost — except the honesty + build
floor, which never lifts. When parity drops back below 90%, this mode
deactivates and the suspended rules resume automatically.

### Suspended while active
- **Roadmap topological order** (`FOLLOW ../ROADMAP-CORRECAO.md`) — pick any
  fixture / feature / epic in any order.
- **Ask-before-regression** — regressions may land without per-change
  confirmation. Still *logged* in the commit/PR body; net parity across a work
  session must not drop.
- **Small-PR scope** — large multi-crate refactors and the deferred epics
  (#195 mutable closures, #207 async event loop, #216/#222 Symbol, #218 Proxy,
  #219 BigInt, #223 dynamic import) are now in scope.
- **Ceremony** — progress-bar / read-everything ritual is optional.

### Never suspended (honesty + build floor)
- **The parity number stays real.** No deleting, disabling, skipping,
  hardcoding, or input-special-casing a fixture to inflate parity. A fixture
  counts as passing only when the runtime genuinely produces the correct output
  through the same code path any other input would take.
- **No crashing / hanging code committed as "pass".** ACCESS_VIOLATION /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UrubuCode/rts](https://github.com/UrubuCode/rts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
