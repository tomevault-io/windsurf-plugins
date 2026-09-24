---
trigger: always_on
description: > **`AGENTS.md` is the real file.** `CLAUDE.md` is a committed symlink to it.
---

# AGENTS.md — Wally

> **`AGENTS.md` is the real file.** `CLAUDE.md` is a committed symlink to it.
> Editing either name edits the same bytes. `bash scripts/ci/check-agents-sync.sh --fix`
> recreates the link and mirrors `.claude/skills/` → `.agents/skills/`. CI fails
> if the pair drifts.

This repository is the official `wally` product CLI. It consumes a **packaged
C++ desktop kit** via `find_package(RunAnywhere)`. It does not `add_subdirectory`
or FetchContent the SDK, and it does not compile llama.cpp / Sherpa / ONNX / MLX
from source.

Pin: `cmake/sdk-pin.cmake`. Prefix: `-DCMAKE_PREFIX_PATH=` or `-DWALLY_SDK_KIT=`.
Pointing `WALLY_SDK_DIR` at SDK **source** is a configure error.

`BEST_PRACTISES.md` / `BEST_PRACTICES.md` (if present) is a local playbook and
must stay gitignored. The rules below are the subset that applies to this CLI.

## Ownership model

```text
argv / flags / env
  -> src/commands/cmd_*.cpp     thin: parse → bootstrap() → one rac_* → render
  -> C++ desktop kit            catalog, download, lifecycle, generate, serve
  -> engines (in the kit)       llama.cpp, Sherpa, ONNX, MLX (Apple host)
```

The kit owns truth: models, backends, proto contracts, download, inference.
The CLI renders and interacts. If a command is composing a multi-step bootstrap,
hardcoding an engine name, or post-processing model output, that is a bug in the
SDK — fix it there, then consume a new kit.

## Layering

- Command TUs stay thin. Business rules do not live in CLI11 callbacks, Swift,
  or the REPL.
- **Proto is the SOT.** Include kit `include/runanywhere/proto/*.pb.h` (same
  protoc that built commons). Do not run `protoc` here. Do not compile `*.pb.cc`
  (those objects are already inside `librac_commons.a`). Parse `rac_*` byte
  buffers with `src/io/proto.h` into `runanywhere::v1::*`.
- Typed contracts at every boundary: CLI11 → proto messages → `rac_*` → stdout.
  No parallel hand-written enums for values that exist in `idl/*.proto`.
- Structured errors. Machine-readable codes from the ABI; human text on stderr.
  Results on stdout. `--json` prints exactly one document on stdout.
- Never log API keys, tokens, or Authorization headers. Status/progress go to
  stderr.

### P0 contract-first network rule

Protobuf remains the source of truth for the packaged SDK/ABI boundary. For a
direct HTTP API owned by RunAnywhere—especially Wally auth, account, billing,
catalog, usage, and inference—the source of truth is the service's pinned
OpenAPI 3.1 artifact. Add the operation to that contract first, then consume a
generated C++ binding (or a mechanically verified thin adapter) from the exact
contract hash.

- Every operation has a stable `operationId` and named request, response,
  error, parameter, and streaming-event schemas.
- No command may invent JSON with string-built paths, ad-hoc objects,
  untyped arrays, duplicated enums, or unchecked response parsing. Closed
  values are generated enums; variants are discriminated unions; owned objects
  are closed; IDs and scalar domains carry their contract constraints.
- The transport may move opaque bytes for the SDK, but business code must never
  treat opaque JSON as a typed result. SSE/OpenAI streaming adapters use
  generated event/chunk types plus conformance tests.
- Pin the OpenAPI artifact hash beside `SCHEMA_LOCK`. Contract, binding, CLI
  adapter, fixtures, and drift/conformance tests land atomically; CI fails when
  any one is stale.

Do not wrap protobuf in OpenAPI merely to change protocol names. When Wally only
transports SDK-owned bytes, protobuf generation and `SCHEMA_LOCK` satisfy this
rule. When Wally directly owns an HTTP call, the OpenAPI requirement applies.

The console's nine CLI calls (`/auth/cli/{start,poll,refresh,revoke}`, `/v1/me`,
`/v1/cli/usage`, `/v1/models`, `/v1/models/catalog`,
`/v1/requests/{request_id}/cancel`) follow this.
`contracts/wally-cli-v1.openapi.json` is the pinned artifact, extracted from
InferenceInfra's `control-plane-v1.openapi.json` by
`contracts/extract-cli-contract.py`. `contracts/generate_console_binding.py`
turns it into `src/account/console_contract.h` (typed requests and responses,
DO NOT EDIT), which `console.cpp` uses instead of hand-built JSON. Requests
serialize strictly; responses read tolerantly (a missing field defaults, a wrong
type or unknown enum value still fails) so the CLI survives a server that lags
the contract. `test_wally_contract` and
`python3 contracts/sync_from_inferenceinfra.py --check` fail the build if the
header, the pin, and the artifact drift.

To re-vendor from an InferenceInfra checkout (records the source commit on the
extract):

```bash
python3 contracts/sync_from_inferenceinfra.py --from /path/to/InferenceInfra
```

`--check` without `--from` is hermetic. Freshness against InferenceInfra HEAD
is enforced on the InferenceInfra PR (`consumer-impact`); this repo cannot
read that private source from CI.

A sync rewrites both generated files, so it refuses to start when either one
has uncommitted changes rather than destroying them; commit or stash first, or
pass `--force`. The same all-or-nothing rule applies to provenance: the
extractor takes `--source-commit` and `--source-branch` together or not at all,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RunanywhereAI/wally](https://github.com/RunanywhereAI/wally) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
