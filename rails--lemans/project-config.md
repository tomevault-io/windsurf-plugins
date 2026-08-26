---
trigger: always_on
description: This file holds only lasting guidelines. Temporary or local instructions (work-in-progress notes, current refactoring state) go to `TODO.local.md` (gitignored, imported via CLAUDE.md); prune entries there as the work lands.
---

# Lemans development guidelines

This file holds only lasting guidelines. Temporary or local instructions (work-in-progress notes, current refactoring state) go to `TODO.local.md` (gitignored, imported via CLAUDE.md); prune entries there as the work lands.

## Architecture layers and boundaries

Each layer may look down, never up:

1. **CLI** (commands, reporters, `CLI::Report`) — builds Config, Store, Runner; renders. Nothing below knows a terminal exists.
2. **Runner** (`Executor`, `Runner::Task`) — scheduling, concurrency, resume; the only place run status, reporter events, and result persistence (`store.save`) meet.
3. **Trial** (+ `Trial::Setup`/`Snapshot`/`Patch`/`Verifier`) — executes one attempt, fills the Result, saves artifacts through the store. Standalone-runnable: `Trial.new(task).run` from a console needs no runner machinery — keep it that way.
4. **Agents / Environments** — adapters doing the work; they report through return values.
5. **Config / TaskDefinition** — passive data, parsed once; knows nothing above it.

`Result` and `Store` ride across the top: constructed at CLI/Runner level, filled/used down to Trial and its components — and never lower.

Boundary rules:

- Never use `Store` or the `Result` record below Trial (and its components). Agents hand evidence back via `Agent::Response` (outcome, usage, trajectory, error, raw result); the Trial persists it. Result's value parts (`Outcome`, `Usage`, `CostSource`) are the shared vocabulary any layer may construct.
- An agent sees exactly three things: its `profile` (the `Config::Agent` section, at construction), the `task`, and the `environment`. Never the whole Config, never paths.
- Environments know no lemans domain: they receive resolved config values (`ImageSpec`, `Resources`, `NetworkPolicy`, opaque labels) and expose the narrow exec contract. No Task, Config, Result, or Store.
- Phase machinery (Trial and its components) parameterizes from `task.<section>` (the per-task projection), never from global config sections. Run-level knobs (agent profile, backend, models, attempts, digest, revision) are read from `config` at Runner/Trial level only.
- Only the Store knows persistence: layout, paths, and reading/writing result.json. `CLI::Report` and `Clobber` consume the store API (`query`/`fetch`/`unreadable`/`delete`); nothing globs the filesystem past the store. `Result` alone owns the serialization (`as_json`/`from_json`).
- `Executor` is generic (anything with `#run`) and reporter-agnostic. Reporters are CLI-side consumers of exactly two payload contracts: `Runner::Task` readers on `:started`, `Result` readers on `:finished`.
- Errors are a boundary too: `ConfigError` is the bench author's bug and raises through every layer; infrastructure/verifier failures become Result statuses at the Trial; a `StandardError` escaping a `Runner::Task` is a harness bug and poisons the run at the Executor.

## Code style

- Ruby 3.4: use modern syntax — `it` block parameter, shorthand hash/kwarg notation (`config_path:`), endless methods for one-liners.
- No explanatory comments. A comment is only warranted for a non-obvious constraint the code cannot express. Never modify or delete comments written by a human.
- Adapter pattern: the abstract base class lives at the root as `Lemans::<Role>` (`store.rb`, `agent.rb`, `environment.rb`); implementations live in the plural directory and inherit the root class (`Stores::FS < Store`, `Agents::Miniswen < Agent`); name-to-class registries live on the plural module (`Agents.build`, `Environments.build`).
- Avoid the `#call` interface — use meaningful verb methods: `Agent#run`, `Setup#execute!`, `Verifier#verify!`.
- `#initialize` organization (see `Result`): required instance vars — blank line — containers (required initial empty values) — blank line — inferred state. Only attributes required at construction go into the signature; the rest is set through writers or lifecycle methods.
- A long-lived process reads better as one procedural method (`Trial#run`) than as extracted steps that make no sense on their own; peripheral extractions (`save_trajectory`, `check_cost_limit!`) are fine.
- Don't extract constants used once in a constructor — inline them as keyword defaults (`timeout: 300`), not `TIMEOUT`.
- Keep legacy files until the very end of a refactoring, even unused ones; don't touch functionality outside the current focus.
- Not defensive: assume user-provided values have sane types (strings where strings are expected) and good faith. Validate semantics (unknown modes, mutually exclusive keys), never types — and never paths: no existence, confinement, or relative/absolute checks on user-declared files.
- No single-purpose helper modules (a module wrapping one validation is an anti-pattern) — make such helpers private methods of the class that uses them. Same for shared conversion modules: only generic value parsing belongs in `Config::Conversion`; feature-specific validation (restore paths) lives on the section that owns the feature.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rails/lemans](https://github.com/rails/lemans) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
