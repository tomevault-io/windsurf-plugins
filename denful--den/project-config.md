---
trigger: always_on
description: Den is a Nix flake framework for declarative multi-entity system configuration. It sits on top of NixOS, nix-darwin, and home-manager, providing a pipeline-driven layer for organizing configuration into reusable, composable units called **aspects**.
---

# Den

Den is a Nix flake framework for declarative multi-entity system configuration. It sits on top of NixOS, nix-darwin, and home-manager, providing a pipeline-driven layer for organizing configuration into reusable, composable units called **aspects**.

## Repository layout

```
flake.nix / default.nix    — entry points (both delegate to nix/)
nix/                       — all library and flake-module code
  default.nix              — true root: exports flakeModule, lib, templates, etc.
  flakeModule.nix           — flake-parts module, imports all of ../modules/
  denTest.nix              — test harness (denTest helper, fixtures: igloo, tuxHm, etc.)
  lib/
    aspects/               — aspect engine: types, resolve, fx pipeline
      fx/                  — effect handlers, key classification, content utils
        handlers/          — compile-static, compile-parametric, emit-classes, bind, gate, etc.
        aspect/            — children, normalize, provide
        policy/            — policy dispatch and effects
    entities/              — host.nix, home.nix entity kind definitions
    diag/capture.nix       — trace capture (graph/rendering moved to denful/den-diagram)
  nixModule/               — den.aspects, den.policies, den.lib option declarations
modules/                   — NixOS-module-style option declarations and batteries
  options.nix              — den.hosts, den.homes, den.schema, den.classes, den.quirks
  aspects/batteries/       — built-in batteries (define-user, home-manager, hostname, etc.)
  policies/                — core and flake-level policy declarations
templates/                 — example flakes + CI test suite
  ci/                      — 133+ test files in modules/features/, deadbugs/ for regressions
  minimal/, default/, example/, noflake/, microvm/, nvf-standalone/
```

## Core concepts

- **Entities** — structural units: `host`, `user`, `home`. Declared via `den.hosts`, `den.homes`. Each has an `.aspect` entry point resolved through the pipeline.
- **Aspects** — the main content unit, declared under `den.aspects.<name>`. Attrsets whose keys are classified as class keys, nested keys, or pipe keys.
- **Classes** — output buckets (`nixos`, `darwin`, `homeManager`). Aspect keys matching registered classes emit modules into that class. Registered via `den.classes`.
- **Provides / `_`** — sub-aspect namespace on every aspect. Used for selectable includes, self-provide, and cross-entity delivery. `_` is an alias for `provides`.
- **Policies** — context-driven functions that emit typed effects (routes, includes, provides). Fire when their argument signature is satisfied by scope context.
- **Pipes / Quirks** — registered via `den.quirks`. Pipe keys in aspects register pipe effects assembled post-pipeline.
- **Scope** — context-derived identity (`"host=igloo,user=tux"`) isolating emissions per entity level. Scope tree emerges from policy-driven context expansion.

## FX pipeline

The pipeline is an algebraic effects trampoline. Every state change is an effect; pure data transforms stay as functions.

Key stages: `resolve` → `compile` (shape router) → `gate` (dedup + constraints) → `compile-static` / `compile-parametric` / `compile-forward` / `compile-conditional` → `classify` → `emit-classes` → `emitNestedAspect` → `resolve-children` → policy iteration → drain deferred.

Four aspect shapes, detected by the compiler router:

- **Static** — no special fields → `compile-static`
- **Parametric** — has `__args` → `compile-parametric` (binds scope args, re-resolves)
- **Forward** — has `meta.__forward` → `compile-forward`
- **Conditional** — has `meta.guard` → `compile-conditional`

## Development commands

```bash
# formatting (required before commits, CI rejects unformatted code)
nix develop -c just fmt

# run full CI suite
nix develop -c just ci

# run full CI suite with full nix-unit output (slow)
nix develop -c just ci-deep

# run a specific test suite
nix develop -c just ci nested-aspects

# run a specific test with traces
nix develop -c just ci nested-aspects.test-direct-nesting-basic

# run tests directly via nix-unit (more control)
nix-unit --override-input den . --flake ./templates/ci#.tests.<suite>

# check a template
nix flake check --override-input den . ./templates/<template>

# interactive repl with den loaded
just repl
```

## Testing

Tests live in `templates/ci/modules/features/`. Bug regressions go in `deadbugs/`.

Test files export `flake.tests.<suite>.<test-name>` using the `denTest` helper:

```nix
{ denTest, ... }:
{
  flake.tests.my-suite = {
    test-something = denTest (
      { den, igloo, ... }:
      {
        den.hosts.x86_64-linux.igloo.users.tux = { };
        den.aspects.igloo.nixos.networking.hostName = "test";

        expr = igloo.networking.hostName;
        expected = "test";
      }
    );
  };
}
```

Key `denTest` args: `den`, `igloo` (nixosConfigurations.igloo.config), `tuxHm` (igloo.home-manager.users.tux), `ns` (when a namespace is imported).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denful/den](https://github.com/denful/den) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
