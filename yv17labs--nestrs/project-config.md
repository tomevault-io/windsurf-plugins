---
trigger: always_on
description: generates: the minor is the floor we build against, the patch is the
---

# CLAUDE.md — nestrs

Durable decisions. The code says what *is*; this file says what was
**decided** and must be **respected**. Not a code map — layout,
signatures and versions live in the code.

Public repo. No machine-local paths, no private references.

Zone-specific rules load on demand from `.claude/rules/` when you touch
a matching file. This file is the always-loaded core: thesis,
invariants, naming, and what "done" means.

## Thesis

nestrs is an opinionated Rust framework whose thesis is **the developer
writes business logic; the framework carries the rest**. Cross-cutting,
error-prone concerns — **authn, authz, row-level filtering,
transactions, edge validation, discovery, lifecycle** — must be
*transparent*. Forcing the developer to hand-manage any of them is a
framework defect.

The leverage is **procedural macros** — decorators, as declarative in
Rust as in TS. Reach for one first.

A framework has **no local change**. Every name, default, error sentence
and declaration is met by every feature that will ever use it, so the
unit of design is the whole surface, never the corner that motivated the
work — see *The ask names a site; the design answers the family*.

## Naming is the pillar — read this before anything else

**A name and its path say the same thing.** From a path you know the type; from
a type you know where the file is. Every other rule in this file assumes that
property, because a reader who cannot navigate cannot check anything else — a
misplaced concern, a missing guard, a silent failure are all invisible to
someone who cannot find the file. **This is the single most important rule in
the project.**

It is also the rule most reliably under-weighted, and the failure mode is
specific: a name reads fine *in isolation* and is only wrong *against its
location*. `ThrottlerModule` is a perfectly good name; `ThrottlerModule` inside
`nest-rs-redis` is a defect, because nothing in it says which backend you are
looking at when it appears in a stack trace at three in the morning. So a name
is **never judged alone** — always as the qualified path a caller types, and
always against its siblings at the same level.

Three consequences, and they are not negotiable:

1. **The framework's crate name is a subject, and a driver carries it.** The
   bare name of a capability belongs to the crate that defines the port. A crate
   that *implements* somebody else's port prefixes every module it declares with
   its own subject — `nest_rs::throttler::ThrottlerModule` is the port's own
   binding, `nest_rs::redis::RedisThrottlerModule` is Redis's. This mirrors what
   the implementations already do (`InMemoryThrottler`, `RedisThrottler`), and
   the stutter at the path is an accepted cost: a name that is unambiguous in a
   log outranks a name that is short in an import.
2. **The stem is the path.** A module's type name is the crate subject plus
   every folder below `src/`, joined — `redis/queue/module.rs` is
   `RedisQueueModule`, `audio/http/module.rs` is `AudioHttpModule`. Adapters
   read the same way: `posts/http/controller.rs` is `PostsController`.
3. **Siblings follow one scheme.** One odd member means either it or the scheme
   is wrong, and deciding which is the finding — never a shrug. A type renamed
   without its `*Setup`, its `*Host` or its config is half a rename.
4. **A variable is a path too.** A `#[config]`'s namespace is the same stem its
   type is named from — crate subject, then binding folders, joined by `__` — so
   `SeaOrmConfig` reads `NESTRS_SEAORM__URL` and `RedisWorkerConfig` reads
   `NESTRS_REDIS__WORKER__*`. From a variable a reader finds the type; from a
   module they find the variable. A resource word that names neither
   (`NESTRS_DATABASE__URL`) is the defect, however universal the convention.

The composition root follows from the same law and has **three module shapes,
no fourth** — `<Vendor>Module::for_root` opens a resource, `<Port>Module::for_root`
carries a capability's policy, `<Vendor><Port>Module` binds one to the other —
and the framework is *Ports & Adapters*: the port owns the semantics, the
adapter owns only the transport. The model is in `.claude/rules/architecture.md`.

The full model, the tables and the one documented precedence live in
`.claude/rules/architecture.md`; `naming.rs` in `nest-rs-conformance` enforces
what is mechanical, with an empty baseline that only shrinks. **When a naming
question and any other question compete for the answer, naming wins** — settle
it first, then build.

## Rule priority — Rust first, conventions second

Both, in order. When they conflict, **Rust wins** — adapt the
convention, don't bend Rust.

1. **Rust (non-negotiable).** Idiomatic, reviewable: orphan/coherence,
   explicit errors (`thiserror` in libs — no silent failure, no
   swallowed `DbErr`), **zero `unwrap`/`expect` on framework hot paths**
   (tests and one-shot bootstraps may use them), honest APIs
   (`Type::new(deps)` when tests need it), `Result` propagated to the
   transport boundary. Macro-emitted `impl` blocks don't excuse hiding
   errors or bypassing `Repo`.
2. **Conventions (second).** Module/feature folders, decorator names,
   thin handlers, one `service.rs` per feature. Conventions = *where*;
   Rust = *how*.

## Hard "no" list


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YV17labs/NestRS](https://github.com/YV17labs/NestRS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
