---
trigger: always_on
description: `idac` is a CLI for IDA Pro with two execution paths:
---

# AGENTS

## Repo Overview

`idac` is a CLI for IDA Pro with two execution paths:

- `gui`: talks to a live IDA desktop session through the bridge plugin
- `idalib`: opens `.i64` / `.idb` files in a short-lived headless worker

Most implementation lives under `src/idac`:

- `src/idac/cli.py`: command registration and argument parsing
- `src/idac/ops/`: typed operation families, manifest, dispatch, preview execution, runtime helpers, and shared helper modules
- `src/idac/cli2/renderers/`: text rendering
- `src/idac/transport/schema.py`: wire request/response schema
- `src/idac/transport/`: GUI bridge transport and `idalib` worker transport
- `plugin/`: IDA GUI bridge plugin code
- `tests/`: CLI and backend coverage
- `fixtures/`: committed binaries, databases, logs, and source used by tests
- `docs/` and `src/idac/skills/idac/`: user-facing command docs and agent-oriented usage guidance

## Working Style

- Prefer `uv run ...` for repo-local commands.
- Prefer targeted tests first, then broader validation if the change touches shared behavior.
- Treat committed fixture artifacts as part of the product surface. If you change fixture symbols, fixture source, or docs/examples that depend on them, regenerate the fixture outputs too.
- Do not revert unrelated worktree changes. This repo may contain user-owned untracked recovery artifacts and local editor files.
- In `src/idac/cli2`, keep `argparse.Namespace` at the parser boundary. Use direct `args.foo` access for fields guaranteed by that subcommand, and reserve `vars(args).get(...)` for wrapper or `argparse.SUPPRESS` cases.
- For command-local argument normalization in `src/idac/cli2/commands/`, prefer `_foo_request(args) -> FooRequest` plus `FooRequest.to_params()` rather than spreading selector/default coercion through handlers.
- When request-building logic becomes nontrivial, add a focused unit test for the builder itself in addition to end-to-end CLI coverage.

## Reverse-Engineering Defaults

- Work from the binary first. Do not search the web or external source trees unless the user explicitly asks for that or the task is specifically about external correlation.
- During type or prototype recovery, always use `idac decompile --f5` or `idac decompilemany --f5` so readback reflects the latest imported types and signatures. `--f5` is the same as `--no-cache`.
- Before `function prototype set`, run `function prototype show` to read the current signature and confirm what is changing.
- After meaningful type or prototype mutations, run `idac misc reanalyze ...` before local rename-heavy cleanup, then reread pseudocode or locals instead of assuming propagation.
- Before batch local renames, capture `idac function locals list <func> --json` and prefer `--local-id` or `--index` selectors once prototypes or reanalysis may have shifted the local set.
- Stop a rename batch on the first miss. Reread locals, recalibrate selectors, and only then continue.
- Declare support types before dependent prototypes. If a prototype references a missing type, create the placeholder type first and retry.
- Prefer minimal `struct` declarations first. Start with the vtable pointer and directly observed fields, keep uncertain names provisional, and use blob padding for unknown regions instead of guessed scalars.

## Common Commands

Initial setup:

```bash
uv sync
```

Useful local commands:

```bash
uv run idac --help
uv run idac --full-help
make format
make lint
make test
make check
uv run pytest -q tests/test_idalib_classes.py
IDAC_RUN_LIVE_GUI_TESTS=1 uv run pytest -q -m gui_live tests/test_gui_transport_live.py
```

Prefer targeted `idac <command> --help` when you already know the likely command family. Use `idac --full-help` when you need the full command tree in one pass.

## Codebase Notes

When changing commands or request/response shapes:

- update CLI wiring in `src/idac/cli.py`
- update the operation implementation in `src/idac/ops/`
- update renderers/schema if output shape changed
- update tests and any affected docs under `README.md`, `docs/`, or `src/idac/skills/idac/`

When changing the operation layer, keep these boundaries in mind:

- `src/idac/ops/runtime.py` is the shared toolkit for reusable IDA-facing helpers. Prefer adding cross-operation lookup, normalization, and readback helpers there instead of duplicating them across op modules.
- keep `src/idac/ops/families/` focused on command-family orchestration, typed request/result models, and user-facing error messages
- `src/idac/ops/manifest.py` is the source of truth for supported ops, mutation flags, and preview metadata
- `src/idac/ops/dispatch.py` should derive handler registration from the manifest and registry, not maintain a parallel operation list
- `src/idac/ops/preview.py` should stay thin. If preview behavior changes, prefer encoding defaults and policy in `PreviewSpec` rather than branching in wrappers.
- `src/idac/cli2/renderers/__init__.py` owns text rendering. Before adding another formatter, look for an existing helper or adjacent renderer that can absorb the behavior.
- for `type declare`, keep `DeclarationChunk` as the internal representation through parse / diagnose / bisect flows and only convert to plain dicts at the API boundary when needed by tests or wire output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trailofbits/idac](https://github.com/trailofbits/idac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
