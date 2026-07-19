---
trigger: always_on
description: Public CLI and Python package for Softmax v2 tournaments ("Coworlds"). This package owns the user-facing `coworld`
---

# AGENTS.md - coworld

Public CLI and Python package for Softmax v2 tournaments ("Coworlds"). This package owns the user-facing `coworld`
entrypoint, local episode/play tooling, Coworld uploads, policy uploads/submission helpers, the Paint Arena reference
Coworld, and the public Coworld docs shipped with the package. It depends on `softmax-cli` for auth-backed commands.

## ⚠️ Building a player that calls an LLM / Bedrock? Read [`src/coworld/docs/BEDROCK.md`](src/coworld/docs/BEDROCK.md) FIRST.

The one rule: in a hosted episode, **send every Bedrock call to the `AWS_ENDPOINT_URL_BEDROCK_RUNTIME` endpoint** (the
per-pod sidecar that signs with the runner identity), using **`InvokeModel`, not `Converse`**. Hitting the real AWS host
instead → HTTP 403 with the injected placeholder creds, and a silent fall back to a non-LLM baseline. Standard SDKs
(boto3, `AnthropicBedrock`, AWS SDK for JS, `@cogweb/llm`) honor that env var automatically; hand-rolled HTTP must read
it. Full contract, copy-paste examples, and the 403 troubleshooting table: [`BEDROCK.md`](src/coworld/docs/BEDROCK.md).

## Coworlds Expert Agent

A distributable Claude Code agent for coworld developers is available at
[`docs/coworlds-expert-agent/`](docs/coworlds-expert-agent/). It knows coworld design principles
(the derivation chain, grader philosophy, player policy design, schema contracts) and can be
installed into any coworld project's `.claude/agents/` directory. See its
[README](docs/coworlds-expert-agent/README.md) for install instructions.

## Before Editing

- Read this file, `LESSONS.md`, the package [README](README.md), and the [Coworld docs map](src/coworld/docs/README.md)
  before changing user-facing Coworld behavior or docs.
- Keep package docs public-package-facing. Avoid private Metta backend paths unless the document is intentionally
  explaining a platform integration boundary.
- Treat Paint Arena under `src/coworld/examples/paintarena/` as the canonical in-tree example.

## CLI

The package installs the Typer app at `coworld.cli:app`:

```bash
uv run coworld --help
uv run coworld leagues
uv run coworld download <coworld-name-or-id> --output-dir ./coworld
uv run coworld run-episode <manifest-or-id> [request.json|image...] [-n N]
uv run coworld scrimmage <manifest-or-id> <policy-image>
uv run coworld play <manifest.json> [image|request.json]
uv run coworld xp-request create <body.json|-> / list / get / episodes
uv run coworld reporters list [-q TEXT] [--type T] [--mode hosted|external] / search <text> / show <rptr_...>
uv run coworld build / certify / upload-coworld
uv run coworld upload-policy / submit
uv run coworld player list / use <player-id> / unset
```

Auth-backed commands require `uv run softmax login` first. The `auth` extra pulls in `softmax-cli`.

`coworld player` is softmax-cli's player subapp mounted for discoverability (player identity is a
Softmax-platform concept; `softmax player ...` is the same thing, implemented in
`packages/softmax-cli/src/softmax/players.py`). `player use <player-id>` mints (or reuses) a 24h player
session and stores it as the active player in `~/.softmax/credentials.yaml` (`player_sessions`). Every
identity-bearing command then acts as that player, because they all resolve their token through
`softmax.auth.load_current_token`. `player unset` clears the active pointer, reverting to your main user
credential.

## Validation

Use the narrowest check that covers the touched surface, then broaden when changing shared contracts:

```bash
uv run metta pytest packages/coworld/tests/test_types.py -v
uv run metta pytest packages/coworld/tests -v
uv run metta pytest --changed
./bazel/fix_lint.sh
```

When changing manifest Pydantic models or generated schema files, update `types.py` first and regenerate the checked-in
schema JSON:

```bash
uv run --project packages/coworld python packages/coworld/scripts/generate_coworld_schemas.py
uv run metta pytest packages/coworld/tests/test_types.py -v
```

Do not hand-edit `src/coworld/coworld_manifest_schema.json` or `src/coworld/runner/episode_request_schema.json` as the
source of truth. They are generated docs and `$schema` targets; `test_types.py` checks that they match `types.py`.

## Source Layout

- `src/coworld/cli.py`, `tournament_cli.py` - Typer command surface for local episodes, uploads, leagues, and hosted
  tournament inspection.
- `src/coworld/cli_support.py`, `api_client.py` - shared CLI helpers and the Softmax/Coworld API client.
- `src/coworld/certifier.py` - `coworld certify` smoke-test pipeline (episode + declared-reporter certification).
- `src/coworld/manifest_validation.py`, `schema_validation.py`, `manifest_uri.py` - manifest and schema validation.
- `src/coworld/report.py` - the safe-render-profile checker `coworld certify` enforces on commissioner round reports.
  (The reporter report-zip role was retired by spec 0061; the wasm reporter's emit-time render check lives in
  `app_backend/src/metta/app_backend/reporters/render_check.py`.)
- `src/coworld/bundle.py` - `coworld build`: hydrates a manifest template from a Docker Compose build.
- `src/coworld/play.py`, `src/coworld/runner/` - local play, local episode runner, and hosted-runner contracts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Metta-AI/coworld](https://github.com/Metta-AI/coworld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
