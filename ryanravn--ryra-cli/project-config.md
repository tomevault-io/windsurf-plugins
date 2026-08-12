---
trigger: always_on
description: Prefer a small, tight, obvious codebase over a large one full of knobs. Every feature, abstraction, error variant, config field, and Step variant is a long-term support burden — it needs docs, tests, migration paths, and has to keep working through every future refactor. Ship less; polish what you ship.
---

# Ryra Development Guidelines

## Core Principle: Fewer Features, Well Implemented

Prefer a small, tight, obvious codebase over a large one full of knobs. Every feature, abstraction, error variant, config field, and Step variant is a long-term support burden — it needs docs, tests, migration paths, and has to keep working through every future refactor. Ship less; polish what you ship.

Applied:
- Fix the bug in front of you; don't surrounding-cleanup or "while I'm here" refactor.
- Add the smallest thing that works end-to-end. No speculative flexibility, no "might need this later" parameters.
- Prefer extending existing types over introducing parallel ones. Two enum variants < new enum. One new function < new module.
- A symmetric code path in core (planning) is better than a new Step variant when the existing ones already express it (e.g., `Step::WriteFile` for Caddyfile edits — don't add `Step::EditCaddyfile`).
- When in doubt, delete.

## Core Principle: Design for the Right Shape, Not the Smallest Diff

"Fewer features" is about feature surface, not change surface. When the *right* design for a problem requires touching many files, restructuring an enum across multiple crates, or unifying parallel code paths into one, do that — don't ship a narrow patch that leaves the architecture worse and call it done because it's smaller. Design for scalability, not MVP.

A holistic refactor that gets the abstractions right is cheaper long-term than a sequence of minimal patches that each accrete a new special case. If a config field, enum variant, or call site is in the wrong shape for what the system needs to do, fix the shape — even if it ripples through the codebase.

Applied:
- Don't choose a worse design because it's a smaller change. The size of the diff is not the cost; the size of the resulting support burden is.
- Don't preserve a parallel code path "for now" when one unified path is more correct.
- When a new feature exposes that an existing abstraction is wrong, restructure the abstraction. Don't bolt the feature onto the wrong shape and leave the cleanup as future work.
- Design for the system you'll have in a year, not the smallest thing that works today.

This complements "Fewer Features, Well Implemented" — it does not contradict it. Ship less, but make what you ship structurally right. "Smallest thing that works end-to-end" applies to *feature scope* (no speculative knobs, no hypothetical-future parameters); it does not give you license to pick a design you know is wrong because the corrected one is more code.

## Core Principle: Make Invalid State Unrepresentable

Use enums and pattern matching everywhere instead of string comparisons, boolean flags, or if-chains. This applies at every layer:

- **Config values**: DNS, SSL, SMTP, auth providers are enums with associated data, not string fields with optional companions
- **Commands/actions**: Operations returned from core to CLI are typed enums (e.g., `Step::WriteFile { .. }`, `Step::StartService { .. }`), not string commands that get parsed with `.contains()`
- **Service status**: `Available | Installed`, not a bool flag
- **Service kind**: `Application | Infrastructure`, not a string

When adding new functionality, ask: "Can this state be invalid?" If yes, restructure with enums so the type system prevents it. Pattern matching (`match`) must be exhaustive — the compiler enforces that every case is handled.

**Anti-patterns to avoid:**
- `if config.provider == "letsencrypt"` → use `match config.ssl { SslConfig::Letsencrypt { .. } => .. }`
- `if cmd.contains("start")` → use `match step { Step::StartService { .. } => .. }`
- Optional fields that are only valid in certain states → put them inside enum variants

### Validate at the boundary

All TOML files are validated immediately after deserialization — if parsing succeeds, the data is safe to use without further checks:

- **`ServiceDef::validate()`** — duplicate names (ports, env), env var name format, env kind consistency, RAM consistency (recommended >= minimum)
- **`TestToml::validate()`** — mutually exclusive `[[tests]]`/`[[steps]]`, required fields per step action type
- **`Config::validate()`** — no duplicate service names
- **`StepAction`** is an enum, not a string — serde rejects unknown actions at parse time

When adding new fields or service definitions, the compiler and `validate()` should catch structural errors. Never silently default on missing/invalid data — error loudly at load time.

## No Unwraps, No Silent Failures

Never use `.unwrap()`, `.expect()`, or `panic!()`. Every fallible operation must be handled with `?`, `match`, or a meaningful default. This includes:

- `Option` values — use `?`, `ok_or()`, `unwrap_or_default()`, or pattern match
- `Result` values — propagate with `?` or handle explicitly
- Indexing — use `.get()` instead of `[]` where bounds aren't guaranteed

If something truly cannot fail, explain why in a comment and use `unwrap_or_else(|| unreachable!("reason"))` so the reasoning is documented.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryanravn/ryra-cli](https://github.com/ryanravn/ryra-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
