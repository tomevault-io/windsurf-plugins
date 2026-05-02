---
trigger: always_on
description: > **For AI coding agents helping users adopt and use Den.**
---

# AGENTS.md — Den AI Agent Guide

> **For AI coding agents helping users adopt and use Den.**
> Den is an aspect-oriented, context-driven Dendritic Nix configuration framework.
> Read this document fully before generating any Den configuration.

______________________________________________________________________

## 1. Mandatory Source Consultation

**Always read the source on an as-needed basis.** Do not guess at API shapes or option names. Instead, look them up directly in the repository. The following directories are your primary references:

| Directory | What it contains |
|-----------|-----------------|
| `./docs/src/content/docs/` | Full user-facing documentation (explanation, guides, reference, tutorials) |
| `./nix/` | Core Den library (`parametric`, `canTake`, `take`, `__findFile`, context types, etc.) |
| `./modules/` | OS framework (schema options, aspect definition, batteries, context wiring, output) |
| `./templates/ci/modules/` | **Every Den feature as a fully isolated, executable nix-unit test** |

The CI test suite at `./templates/ci/modules/features/` is the **most authoritative** working-code reference. Every test is a self-contained, evaluated Nix expression that demonstrates exactly how a feature behaves. Read the relevant test file whenever you need a code example.

**Key source files to consult on demand:**

- `docs/src/content/docs/explanation/core-principles.mdx` — Design philosophy
- `docs/src/content/docs/explanation/aspects.mdx` — Aspect & functor pattern
- `docs/src/content/docs/explanation/parametric.mdx` — Parametric dispatch mechanics
- `docs/src/content/docs/explanation/context-pipeline.mdx` — Host → user → home pipeline
- `docs/src/content/docs/explanation/context-system.mdx` — `den.ctx` architecture
- `docs/src/content/docs/explanation/library-vs-framework.mdx` — Using Den without NixOS
- `docs/src/content/docs/reference/lib.mdx` — Full `den.lib` API
- `docs/src/content/docs/reference/ctx.mdx` — Full `den.ctx` API
- `docs/src/content/docs/reference/schema.mdx` — `den.hosts`, `den.homes`, `den.schema`
- `docs/src/content/docs/reference/aspects.mdx` — `den.aspects`, `den.provides`, `den.ful`
- `docs/src/content/docs/reference/batteries.mdx` — All `den.provides.*` batteries
- `docs/src/content/docs/reference/output.mdx` — Flake output generation
- `docs/src/content/docs/guides/` — Practical cookbooks (batteries, custom classes, namespaces, migration, debugging)
- `nix/lib/default.nix` — Library entrypoint (all sub-modules listed)
- `modules/options.nix` — `den.hosts`, `den.homes`, `den.schema` option declarations
- `modules/config.nix` — How hosts and homes are instantiated into flake outputs
- `modules/aspects/provides/` — All built-in battery implementations

______________________________________________________________________

## 2. Den in One Paragraph

Den is built on [`flake-aspects`](https://github.com/vic/flake-aspects). It inverts the traditional host-centric Nix model: **aspects** (features) are the primary unit of organization. Each aspect declares its behavior per Nix *class* (`nixos`, `darwin`, `homeManager`, `hjem`, `maid`, `user`, custom…). Hosts simply include the aspects they need. A **context pipeline** (`den.ctx`) transforms host/user/home declarations into fully resolved Nix module system inputs. Parametric dispatch (via `builtins.functionArgs` introspection) means a function requiring `{ host, user }` is silently skipped in a `{ host }`-only context — no `mkIf`, no `enable` flags needed to gate context-sensitive configuration.

______________________________________________________________________

## 3. Core Concepts

### 3.1 Aspects

An aspect is an attrset containing:

- **Owned configs**: keys named after Nix classes (`nixos`, `darwin`, `homeManager`, `hjem`, `maid`, `user`, `os`, or any custom class). Values are plain attrset modules or function modules (`{ pkgs, ... }: { }`).
- **`includes`**: a list of other aspects, static attrsets, or parametric functions to include as dependencies.
- **`provides`**: named sub-aspects scoped to this aspect, accessible via `den.aspects.foo.provides.bar` or the shorthand `den.aspects.foo._.bar`.

Read: `docs/src/content/docs/explanation/aspects.mdx`, `docs/src/content/docs/guides/configure-aspects.mdx`\
CI examples: `templates/ci/modules/features/parametric.nix`, `templates/ci/modules/features/top-level-parametric.nix`, `templates/ci/modules/features/auto-parametric.nix`

### 3.2 Parametric Dispatch

Den uses `builtins.functionArgs` to inspect a function's declared arguments. A function is included only when all its **required** (non-default) arguments are present in the current context:

- `{ host, ... }` → matches `{ host }`, `{ host, user }`, etc. (atLeast)
- `{ host, user }` → matches only when both are present
- `{ home }` → matches only standalone home contexts
- `{ class, aspect-chain }` → static aspect (evaluated during resolution, not per-context)

`den.lib.parametric` wraps an aspect with this dispatch logic. `den.lib.canTake`, `den.lib.take.atLeast`, `den.lib.take.exactly` are the underlying primitives.

Read: `docs/src/content/docs/explanation/parametric.mdx`, `docs/src/content/docs/reference/lib.mdx`\
CI examples: `templates/ci/modules/features/parametric.nix`

**Parametric variants:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greysilly7/nixos-config](https://github.com/greysilly7/nixos-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
