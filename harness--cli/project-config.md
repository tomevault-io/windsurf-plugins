---
trigger: always_on
description: A spec-driven Go CLI for Harness. Commands are declared in YAML spec files; the framework wires them into Cobra subcommands at startup. Agents rarely need to touch Go code — most work is adding or editing spec files.
---

# harness/cli — Agent Guide

## What this repo is

A spec-driven Go CLI for Harness. Commands are declared in YAML spec files; the framework wires them into Cobra subcommands at startup. Agents rarely need to touch Go code — most work is adding or editing spec files.

## Using the CLI

**Grammar: `harness <verb> <noun> [id] [flags]`** — verb always comes first.

```sh
harness create pr
harness list pipeline
harness get pr <repo_id>/<pr_number>
harness execute pr:merge <repo_id>/<pr_number>
```

### Discovery

```sh
harness get module <name>      # domain model and noun list for a module (e.g. "code", "pipeline")
harness get noun <noun>        # fields and available verbs for a specific noun
harness list noun --matrix     # all nouns × verbs at a glance
harness <verb> <noun> --help   # flags for a specific command
```

### Common verbs

| Verb | Purpose |
|------|---------|
| `list` | List resources (paginated) |
| `get` | Get a single resource by id |
| `create` | Create a resource |
| `update` | Update a resource |
| `delete` | Delete a resource |
| `execute` | Run/trigger a resource (pipelines, merges, etc.) |

### Qualified nouns (`noun:variant`)

Some commands use a variant suffix to distinguish sub-operations on the same noun:

```sh
harness list pr:mine                        # PRs authored by you
harness execute pr:merge <repo_id>/<pr_number>  # merge a PR
harness execute pr:close <repo_id>/<pr_number>  # close a PR
harness get pipeline:summary <pipeline_id>
```

### Scope flags

Most commands accept `--org` and `--project` to override the profile defaults:

```sh
harness list pipeline --org my-org --project my-project
```

---

## Build & install

```sh
# Build (requires Task: brew install go-task)
task build                    # outputs bin/harness and bin/harness-har

# Faster build when only touching core (not HAR) — HAR pulls in large libraries and is slow
task build:main               # builds bin/harness only, skips HAR

# Install to GOPATH
go install ./cmd/harness/...

# IMPORTANT: active binary lives at ~/.local/bin/harness, not ~/go/bin/harness
cp $(go env GOPATH)/bin/harness ~/.local/bin/harness
```

## Repository layout

```
cmd/harness/          # main entry point
pkg/
  spec/               # ← primary work area: *.spec.yaml + Go struct types
  registry/           # spec → Cobra command wiring (endpoint.go, verb.go, etc.)
  endpoint/           # HTTP execution: BuildRequest, HTTPFetchFn, paging strategies
  exprenv/            # expr-lang evaluation (flags.*, ctx.*, auth.*, it.*)
  client/             # HTTP client (DoRequest, auth injection)
  cmdctx/             # Ctx struct threaded through all handlers
  format/             # Table/field rendering
  tui/                # Interactive UI picker
modules/har/          # External HAR module (separate go.mod)
```

## How specs work

Each `*.spec.yaml` declares:
1. **`nouns`** — entity types with their `fields` (id, expr, label, field_type, mutable_path, width_max, align).
2. **`commands`** — `verb noun[:variant]` pairs wired to `handler_type: endpoint`.

### Noun variant → Cobra subcommand name

```yaml
noun: kg
noun_variant: type      # → cobra command "kg:type"
```

`FullNoun()` returns `"noun:variant"` when variant is set, `"noun"` otherwise.

### Field rendering rules

| Field | When it applies |
|-------|----------------|
| `columns: [...]` | Which fields show in `list` output |
| `fields_subset: [...]` | Which fields show in `get` output (on endpoint, not noun) |
| All noun fields | Available to `get` unless `fields_subset` limits them |

### Mutable fields and update commands

Fields with `mutable_path` are writable via `--set`/`--del` on update commands. `mutable_path` is a dot-path **relative to the `update_body_pick` subtree** — never starts with `it.`:

```yaml
# noun field
- id: name
  expr: it.project.name      # display expression
  mutable_path: name         # relative path within the picked subtree

# update command endpoint
update_strategy: get-then-put
update_body_pick: it.data.project   # evaluated against root GET response
update_body_wrap: project           # re-wraps the mutated object in PUT body
```

Rules:
- `update_body_pick` should match `yaml_pick_expr` on the corresponding `get` command — they describe the same subtree.
- Fields without `mutable_path` are read-only and do not appear in `--list-fields`.
- `mutable_path` must not start with `it.` — the spec validator will reject it.

### Path templating

Paths use Go template syntax evaluated by `exprenv`:

```yaml
path: /code/api/v1/repos/{{ctx.parentId}}/branches
```

Available variables: `ctx.id`, `ctx.idParts[N]`, `ctx.parentId`, `auth.account`, `auth.org`, `auth.project`, `flags.*`.

### id_parts vs requires_parentid

- `id_parts: 2` → user passes `<a>/<b>`; available as `ctx.idParts[0]` and `ctx.idParts[1]`. Works for `get`/`execute`/`delete`.
- `requires_parentid: true` → user passes the parent as a positional arg; available as `ctx.parentId`. Used for `list`/`create` where the sub-resource doesn't have its own id yet. **`id_parts` is NOT supported by `list`.**

### expr-lang tips


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harness/cli](https://github.com/harness/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
