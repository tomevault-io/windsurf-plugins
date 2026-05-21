---
trigger: always_on
description: **BigConfig** is a Clojure/Babashka workflow and template engine for infrastructure-as-code (IaC) automation. It provides a "zero-cost build step" by using Clojure to generate configuration files (JSON, YAML, HCL) and orchestrate CLI tools (Terraform/OpenTofu, Ansible, Kubectl, etc.).
---

# BigConfig — AI Assistant Guide

## Project Overview

**BigConfig** is a Clojure/Babashka workflow and template engine for infrastructure-as-code (IaC) automation. It provides a "zero-cost build step" by using Clojure to generate configuration files (JSON, YAML, HCL) and orchestrate CLI tools (Terraform/OpenTofu, Ansible, Kubectl, etc.).

The library is published as a Clojure tool and consumed via `clojure -Ttools` or Babashka (`bb`). Versioning follows `0.3.<git-commit-count>`.

---

## Repository Layout

```
big-config/
├── src/clj/            # Production source (Clojure)
│   ├── big_config/     # Core library namespaces
│   └── big_tofu/       # OpenTofu/Terraform HCL constructs
├── test/clj/           # Test files (mirrors src layout)
├── test/resources/     # Shared test data
├── test/fixtures/      # Pre-built template fixtures
├── resources/
│   ├── big-config/     # Template library (devenv, ansible, multi, etc.)
│   └── quickdoc/       # API doc generation config
├── env/
│   ├── dev/clj/        # Dev-environment source (REPL helpers)
│   └── test/resources/ # Test-env resources
├── .big-config/        # Internal project self-bootstrapping config
├── .github/workflows/  # CI/CD (ci.yml)
├── .clj-kondo/         # Linter config (lint-as mappings, Specter macros)
├── .lsp/               # LSP config (clojure-lsp)
├── deps.edn            # Clojure deps and aliases
├── bb.edn              # Babashka tasks
├── devenv.nix          # Nix dev environment definition
└── devenv.yaml         # devenv packages list
```

---

## Core Namespaces

| Namespace | Purpose |
|---|---|
| `big-config.core` | Foundational primitives: `ok`, `choice`, `->workflow`, `->step-fn` |
| `big-config.workflow` | High-level orchestration: `->workflow*`, `run-steps`, `parse-args` |
| `big-config.pluggable` | Multimethod extensibility: `handle-step`, `->workflow*` |
| `big-config.render` | Selmer-based template engine |
| `big-config.lock` | Git-tag pessimistic locking (client-side Atlantis) |
| `big-config.unlock` | Force-release locks |
| `big-config.run` | Shell command execution (`generic-cmd`, `run-cmds`) |
| `big-config.store` | Redis-backed event-sourcing store (`write!`, `restore!`) |
| `big-config.system` | Lifecycle management (alternative to Integrant) |
| `big-config.git` | Git helpers (check, push) |
| `big-config.step-fns` | Step function wrappers (`->step-fn`, `log-step-fn`) |
| `big-config.utils` | Helpers: `deep-merge`, `debug` macro, `keyword->path`, Specter walkers |
| `big-config.selmer-filters` | Custom Selmer template filters |
| `big-config.toml` | TOML parsing/generation |
| `big-config.tools` | Template scaffolding CLI (`package`, `devenv`, `action`) |
| `big-config.build` | Build orchestration |
| `big-tofu.core` | OpenTofu `To` protocol, `Construct` record, AWS ARN helpers |
| `big-tofu.create` | BigTofu stdlib for common constructs |

---

## Key Concepts

### Workflow Engine

Workflows thread an `opts` map through a sequence of steps. Every step must return `opts` with `::big-config/exit` set to a non-negative integer (0 = success, non-zero = failure).

```clojure
;; Minimal step
(defn my-step [opts]
  (core/ok opts))   ; sets ::big-config/exit 0

;; Step that fails
(defn failing-step [opts]
  (merge opts {:big-config/exit 1 :big-config/err "Something went wrong"}))
```

**Workflow types:**
- `tool-workflow` — renders templates and executes one CLI tool
- `comp-workflow` — sequences multiple `tool-workflows` into a lifecycle (`create`, `delete`) and can expose workflow-level `validate` / `describe` hooks
- `system-workflow` — manages start/stop of background system components

#### Composition layer (subworkflow isolation)

`run-steps` and `workflow/->workflow*` are a *composition layer* — a "workflow
of workflows" — **not** workflow steps. The pure-step / single-threaded-`opts`
contract holds *within one workflow*. Across the composition layer the
invariant is instead **subworkflow isolation**:

- Each subworkflow (`::create`/`::delete`, or each `:pipeline` step) runs on a
  purpose-built `opts` — `create-opts`/`delete-opts`, or
  `(merge step-args globals-opts <step>-opts)` — seeded from the shared globals,
  **never** the parent's running `opts`.
- `::validate` and `::describe` are workflow-level hooks dispatched by
  `run-steps` through `::workflow/validate-fn` and `::workflow/describe-fn`.
  They run only when explicitly requested in `::workflow/steps` and should
  return the normal `opts` map with `::bc/exit` / `::bc/err`.
- Each subworkflow's terminal `opts` is accumulated under its step key (a
  *vector* in `run-steps`, so repeated `create`/`delete` runs are kept
  side-by-side as history); only `::bc/exit` / `::bc/err` propagate upward to
  drive the parent's branching and short-circuit.

The closed-over atoms in these two functions *implement* this isolation; they
are deliberate, not a purity leak (see *What to Avoid*).

### Pluggable Steps (Multimethods)

Override or add steps via `big-config.pluggable/handle-step`:

```clojure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bigconfig-ai/big-config](https://github.com/bigconfig-ai/big-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
