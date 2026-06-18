---
trigger: always_on
description: This file is operational context for AI coding agents. Human contributors should
---

# Laravel Swarm — Agent Context

This file is operational context for AI coding agents. Human contributors should
start with [README.md](README.md), [CONTRIBUTING.md](CONTRIBUTING.md),
[CHANGELOG.md](CHANGELOG.md), and the user-facing docs in [docs/](docs/).

## What This Is

`builtbyberry/laravel-swarm` is a Laravel package that adds reusable multi-agent orchestration on top of the official `laravel/ai` package. Laravel AI handles single-agent LLM interactions and low-level workflow primitives. Laravel Swarm turns repeated multi-agent workflows into first-class application objects with sync, queued, streamed, and durable execution modes.

## Package Identity

- **Packagist:** `builtbyberry/laravel-swarm`
- **Namespace:** `BuiltByBerry\LaravelSwarm`
- **GitHub:** `https://github.com/builtbyberry/laravel-swarm`
- **Author:** Daniel Berry (J Street Digital)
- **Location:** `~/Code/laravel-swarm`

## Core Design Principle — Laravel Native Feel

Every implementation decision must follow existing Laravel and Laravel AI conventions. A developer who knows Laravel AI should look at a swarm class and understand it immediately. Do not invent new framework patterns when Laravel already has a recognizable convention.

- Laravel AI uses `make:agent`; this package uses `make:swarm`.
- Laravel AI agents use `Promptable`; swarms use `Runnable`.
- Laravel AI uses PHP attributes like `#[Provider]` and `#[Model]`; swarms use `#[Topology]`, `#[MaxAgentSteps]`, and `#[Timeout]`.
- Laravel AI agents use `prompt()`, `queue()`, `stream()`, `broadcast()`, `broadcastNow()`, and `broadcastOnQueue()`; swarms use those same public verbs plus `dispatchDurable()`.
- `run()` remains available as a compatibility alias for synchronous `prompt()`.
- Laravel AI fakes with `Agent::fake()`; swarms fake with `Swarm::fake()` / `YourSwarm::fake()`.
- Config lives in `config/swarm.php`, not `config/ai.php`.
- Generated swarm classes live in `app/Ai/Swarms/`, extending the `app/Ai/` namespace Laravel AI establishes.

## Product Completeness Standard

- Build the full product behavior, not an MVP approximation.
- Do not choose the path of least resistance if it leaves operational, security, developer experience, documentation, or testing gaps.
- Solve the underlying problem completely, including failure modes, operator workflows, rollout and maintenance implications, and reviewer expectations.
- If a shortcut would trade away correctness, resilience, or product completeness, do not take it silently. Surface the tradeoff explicitly and treat it as a decision, not an implementation detail.
- Default to closing gaps during implementation rather than leaving follow-up work unless the deferral is explicitly approved.

## Definition of Done

A feature or fix is complete only when all of the following are true. Reviewers should treat missing items as blockers, not follow-up work.

- **Artisan command** — if the feature introduces an operational concern (relay, prune, recover, status), it ships with a corresponding `swarm:*` command with a `--help` description.
- **Config key** — every new tuneable has a key in `config/swarm.php` with an inline comment explaining intent and a safe production default.
- **Prune / retention hook** — any new persistent data has a `swarm:prune` integration or an explicit documented reason it is exempt.
- **Test coverage** — new behavior has deterministic Pest tests; concurrency-sensitive paths have a `test:process-concurrency` lane entry.
- **CHANGELOG.md** — every PR includes a changelog entry. Breaking changes also require an `UPGRADING.md` section.
- **Docs parity** — public surface changes update the relevant file in `docs/` or `README.md` in the same PR, not a follow-up.

If a gap is deferred, it must be recorded as a named follow-up with an owner, not silently dropped.

## Tech Stack

- PHP ^8.5
- Laravel ^13.0
- `laravel/ai` ^0.6
- `orchestra/testbench` ^11
- `pestphp/pest` ^4.4 + `pest-plugin-laravel` ^4.1
- `larastan/larastan` ^3.0
- `laravel/pint` ^1.0
- Optional `laravel/pulse` integration

## Dependency And Upgrade Path

Swarm sits directly on **PHP**, **Laravel**, and **`laravel/ai`** with semver ranges. The package **cannot fully insulate** applications from upstream API, streaming, or provider contract shifts. Treat **Laravel** and **`laravel/ai` bumps** (minor or patch) as **integration-test events**: run your automated suite and any swarm-heavy smoke paths after resolving Composer updates. **This package’s changelog** is the contract for Swarm-owned breaking or behavior changes; it does not replace verifying your app against new Laravel or Laravel AI releases.

## Package Shape

Keep the mental model high-level rather than mirroring every file:

- `src/Attributes` — swarm attributes for topology, timeout, and max agent steps.
- `src/Commands` — `make:swarm` plus history, status, prune, pause, resume, cancel, and recover commands.
- `src/Concerns` / `src/Contracts` — public swarm trait and storage/runtime contracts.
- `src/Events` — lifecycle events for started, step started/completed, completed, failed, paused, resumed, and cancelled.
- `src/Jobs` — queued and durable execution jobs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [builtbyberry/laravel-swarm](https://github.com/builtbyberry/laravel-swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
