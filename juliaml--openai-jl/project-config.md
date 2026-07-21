---
trigger: always_on
description: Guidance for AI agents working on this repository.
---

# AGENTS.md

Guidance for AI agents working on this repository.

## Project Overview

This repository is `OpenAI.jl`, an unofficial Julia wrapper for the OpenAI API.
It currently has two API layers:

- A handwritten convenience wrapper in `src/OpenAI.jl` and `src/assistants.jl`.
- A generated OpenAPI client under `src/generated/`, exposed as
  `OpenAI.OpenAIClient`.

The handwritten wrapper is still the stable, ergonomic package API. Do not
remove or rewrite it just because the generated client covers overlapping
endpoints. Existing users call functions such as `create_chat`,
`create_embeddings`, `create_completion`, `create_responses`, `list_models`,
and `retrieve_model`.

The generated client is the broad low-level API surface. It provides typed
models and API groups generated from the OpenAI OpenAPI specification snapshot
in `openapi/openapi.yaml`.

## Repository Layout

- `Project.toml`: package metadata, dependencies, compat bounds, and test
  targets. The package currently requires Julia `1.9` or newer because
  `StreamCallbacks 0.6.x` requires Julia `1.9+`.
- `src/OpenAI.jl`: main module, providers, auth, request helpers, handwritten
  endpoint wrappers, streaming callback integration, generated client include,
  and exports.
- `src/assistants.jl`: handwritten Assistants/Threads/Messages/Runs helpers.
  Some assistant exports are intentionally disabled in `src/OpenAI.jl`.
- `src/generated/`: generated Julia OpenAPI client source. Treat this as
  generated code.
- `openapi/openapi.yaml`: committed snapshot of the upstream OpenAI OpenAPI
  specification used to generate `src/generated/`.
- `scripts/generate_openapi_client.sh`: reproducible generation script using
  the Docker image `openapitools/openapi-generator-cli`.
- `test/`: live and smoke tests. Most tests expect `OPENAI_API_KEY`.
- `docs/`: Documenter.jl source and build script.
- `.gitattributes`: marks `src/generated/**` as generated for GitHub/Linguist.
- `.gitignore`: ignores `Manifest.toml`, editor files, `.env`, and
  `docs/build/`.

## Dependencies And Compatibility

Runtime dependencies are declared in `Project.toml`.

Important current dependencies:

- `HTTP`: handwritten HTTP request layer.
- `JSON3`: handwritten request/response JSON handling.
- `StreamCallbacks`: streaming support for handwritten APIs.
- `OpenAPI` and `TimeZones`: required by generated OpenAPI client code.

Do not add a dependency only for convenience. If a new dependency is required,
add it to `[deps]`, add a conservative `[compat]` entry, and verify docs/tests
still resolve on the supported Julia version.

`Manifest.toml` is ignored and should not be committed for this package unless
the project policy changes explicitly.

## Handwritten API Layer

The handwritten request path is centered on:

- `OpenAIProvider` and `AzureProvider`.
- `auth_header`.
- `build_url`.
- `build_params`.
- `_request` and `openai_request`.
- `OpenAIResponse`.

The handwritten wrappers accept loose Julia-native inputs such as `Dict`,
vectors, strings, and keyword arguments. They return `OpenAIResponse(status,
response)` where `response` is parsed with `JSON3.read`.

Streaming is integrated through `StreamCallbacks`:

- Public callers should pass `streamcallback=...` rather than manually setting
  `stream=true` for `create_chat`.
- `configure_callback!` sets `stream=true` and
  `stream_options=(include_usage=true,)`.

When modifying handwritten API behavior, preserve existing call signatures
unless intentionally making a breaking change. Update README/docs/tests for any
public behavior change.

## Generated OpenAPI Client

The generated client lives in `src/generated/` and is included by
`src/OpenAI.jl`:

```julia
include("generated/OpenAIClient.jl")
```

The generated module is exported as `OpenAIClient`, and the helper
`openai_client(...)` creates an authenticated `OpenAPI.Clients.Client`.

Typical generated-client usage:

```julia
using OpenAI

client = openai_client(ENV["OPENAI_API_KEY"])
models_api = OpenAIClient.ModelsApi(client)
models, http_response = OpenAIClient.list_models(models_api)
```

Generated files should usually be changed by regenerating from
`openapi/openapi.yaml`, not by manual edits. If a generated file must be patched
by hand as a temporary unblocker, document why in the PR and prefer following up
with a generator/spec/template fix.

To regenerate:

```bash
scripts/generate_openapi_client.sh
```

The script:

- Requires Docker.
- Uses `openapitools/openapi-generator-cli` unless
  `OPENAPI_GENERATOR_IMAGE` is set.
- Generates Julia client source with `julia-client`.
- Suppresses generated Markdown docs with
  `--global-property apiDocs=false,modelDocs=false`.
- Replaces `src/generated/`.

The generator may emit warnings for OpenAPI 3.1 support, inline schemas,
free-form objects, aliases, multipart form models, or unknown formats such as
`unixtime`. Those warnings are not automatically failures. Always verify that
the package loads and the generated smoke test passes after regeneration.

## What To Commit

Commit these when they change intentionally:

- `openapi/openapi.yaml`
- `scripts/generate_openapi_client.sh`
- `src/generated/**`
- `.gitattributes`
- README/docs/tests that explain or exercise generated APIs

Do not commit these unless project policy changes:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JuliaML/OpenAI.jl](https://github.com/JuliaML/OpenAI.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
