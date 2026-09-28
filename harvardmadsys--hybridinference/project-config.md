---
trigger: always_on
description: This is an onboarding guide for any AI coding agent (Claude Code, Cursor, Codex,
---

# AGENTS.md

This is an onboarding guide for any AI coding agent (Claude Code, Cursor, Codex,
etc.) working in this repository. Read this first.

## 1. About this project

HybridInference is a FastAPI gateway that routes LLM requests across local
inference servers (vLLM, SGLang, Ollama) and remote OpenAI-compatible providers
(DeepSeek, Zhipu, OpenRouter, Anthropic, Gemini, etc.).

- **Repo README:** [README.md](README.md)
- **Developer guide:** [docs/developer/](docs/developer/)

If you are working on a particular deployment, its hosts, accounts and
operational notes live in that deployment's overlay — see
`distributions/<name>/AGENTS.md`. They are deliberately not here: this file
ships with the source, and a URL or credential written into it is published to
everyone who clones the repository.

## 2. Repo map

The layout is in [docs/developer/contributing.md](docs/developer/contributing.md),
under "Repository layout". Two things it does not tell you:

- Deployment overlays under `distributions/` move out of this repository before
  publication. `example/` is the one that remains.
- `distributions/example/` is marked `EXAMPLE_OVERLAY`, and that marker is what
  excludes it from automatic distribution discovery. Select it explicitly
  with `make up DISTRIBUTION=example` to run the local tutorial.

## 3. Getting set up

Prerequisites: Python 3.10–3.13 (3.12 recommended) and [uv](https://github.com/astral-sh/uv).

```bash
make setup-dev
```

The Makefile honors a `UV_RUN` override: `make lint UV_RUN="uv run --active"`.

## 4. Quality gates

`make format`, `make lint`, `make test`, `make all`. Run `make help` for the
full target list.

[docs/developer/contributing.md](docs/developer/contributing.md) is the
canonical reference for the gates and the test tiers.

Pre-commit hooks are installed by `make setup-dev`.

**Before opening a PR:** run `make format` and ensure `make test` passes.

## 5. Workflow

- **Branch off `dev`**, never `main`.
- **Branch naming:** `<user>/<scope>/<feature-name>` (e.g. `jason/claude/add-x`).
- **Use a git worktree** rather than working in the main checkout. Worktree should be put in /tmp/claude/worktree/<feature-name>.
- **PRs target `dev`.**
- **Verify against a running deployment** before claiming done. If you are
  working on one, its staging host and test account are in its overlay guide
  (`distributions/<name>/AGENTS.md`) — never write credentials into this file.

## 6. Project-specific knowledge

### 6.1 Architecture in one diagram

```text
client → FastAPI gateway (serving/) → routing engine (routing/) → adapter → provider
```

For the full diagram (network layer, observability, storage), see
[docs/developer/architecture.md](docs/developer/architecture.md).

### 6.2 Key abstractions

- **Adapter** — provider-specific client. Lives in `apps/backend/serving/adapters/`.
  Dedicated adapters: `openai_compat` (generic OpenAI-compatible APIs — also
  serves local vLLM/SGLang/Ollama), `claude` (Claude via Google Vertex),
  `anthropic` (direct api.anthropic.com), `gemini`, `openrouter`. Local
  inference servers have no dedicated adapter — they route through
  `openai_compat`.
- **`provider` vs `endpoint_id`** — `provider` is a string label on `ModelConfig`
  identifying the API service (used in metrics labels, e.g. `"openai"`,
  `"anthropic"`). `endpoint_id` is the unique per-endpoint key
  (format `{model_id}:{location}`, minted by `registry._make_provider_id` —
  `local-<port>` / `local` for a host in `_LOCAL_HOSTS`, otherwise
  `<service>-api`, e.g. `glm-4.6:local-12003`, `glm-4.6:zai-api`) used for
  latency profiling and availability tracking. The suffix is not a reliable
  ownership signal: a gateway-owned server on a LAN address is stamped
  `<octet>-api`, and an admin-supplied `route_id` becomes the `endpoint_id`
  verbatim. The word "upstream" appears informally in code
  comments meaning "the remote API" but isn't a formal type.
- **Router / HybridRouter abstraction** — the agreed design uses one common
  router contract with independent strategy implementations. Reuse
  [RouterProtocol](apps/backend/routing/protocols.py) as that contract; do not
  create a duplicate interface merely to name it `HybridRouter`. `FixedRouter`
  in [routers.py](apps/backend/routing/routers.py) and `RouteWiseRouter` are
  peer implementations, both able to select across local and cloud candidates.
  Future Greedy/Nimbus routers belong at the same level. Each implementation
  owns its routing, retry and feedback flow; shared helpers need not impose one
  execution loop on every strategy. A separate `RouteWisePolicy` is not required.
  **Naming distinction:** the existing concrete
  [routing.hybrid.HybridRouter](apps/backend/routing/hybrid.py) composes a policy
  with backends; it is not the common interface in the design. Its `FixedPolicy`
  / `BackendSelection` can remain internal or compatibility implementation
  details. Preserve public imports if names change.
  The current bootstrap uses that concrete composition only for mixed
  `router: fixed` models with `router_params.hybrid_composition: true`.
  Other Fixed models retain the shared FixedRouter. The opt-in composition
  still differs in affinity, rejected primary claim re-selection, and fallback

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HarvardMadSys/hybridInference](https://github.com/HarvardMadSys/hybridInference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
