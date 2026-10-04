---
trigger: always_on
description: Read this before touching code. Architecture and the dependency rule live in
---

# Agent Context — frankenst-ai

Read this before touching code. Architecture and the dependency rule live in
`docs/architecture.md`, methodology in `docs/ways-of-working.md`, release mechanics in
`docs/release.md`. This file carries only what a change can break.

## Where things live (and what breaks)

| Path | Watch out for |
| --- | --- |
| `src/frankstate/` | The published wheel and the only public API. Any change is consumer-facing, and **the PR title's type is the version bump**: `feat` → minor, `fix` → patch, `!` → minor while 0.x. The root exports only `WorkflowBuilder`; `entity` and `managers` export nothing. It is an assembler: LangGraph options travel as `**kwargs` (`WorkflowBuilder(...)` → `StateGraph`, `compile(...)` → `StateGraph.compile`), never as parameters of its own. Handler kwargs are declared by class annotations on the subclass; frankstate raises only for what LangGraph cannot see |
| `pyproject.toml` extras | `databricks` and `mcp` are declared conflicting (`mcp<2` vs `mcp>=2`): sync one, never both. `make install-dev` is the quality env, `make install-mcp` the MCP env; `uv export --all-extras` fails by design |
| `src/config/settings.py` | The one door to configuration paths **and secrets**. `resolve_secret(name)` is how any code reaches a secret (env, then `.env`, then Key Vault only when `AZURE_KEY_VAULT_NAME` names one; otherwise a `LookupError` naming the variable); flagged `AzureSettings` fields fall back the same way. It imports `utils.secrets` and nothing else imports it |
| `src/config/config_llms.yaml` | Data, never code: kwargs and `{secret: NAME}` names. One runtime per `launch` key; a key without its `<provider>.<kind>` section fails at launch, on purpose. `use_responses_api` is declared here per section, not defaulted in Python. `launch()` builds every key at once, so the shipped file keeps every key on `ollama`: `databricks` only works in an env synced with that extra |
| `src/config/config_nodes.yaml` | `metadata.tools` / `metadata.sensitive_tools` are the binding between layouts and tools, by the tools' own `name`s (set in each `*Property`). A name no tool carries fails `build_runtime()`. `SIMPLE_OAKTOOLS_NODE` never lists a sensitive tool: `simple_oak` has no review node |
| `src/core_ai_examples/models/interrupt/human_review.py` | `HumanReviewRequest` is what the pause shows, `HumanReviewDecision` what `Command(resume=...)` must satisfy. The notebook's resume payloads and `server_oaklang_agent.py` read these two |
| `src/services/llm/llm_services.py` | Homologous to `channel-planning`'s copy. This one adds the `azure_ai` provider and resolves secrets through `settings.resolve_secret` (env, `.env`, then Key Vault when named); that one has `ollama` + `databricks` and resolves through `utils.secrets` with a Databricks scope. A shared fix goes to both |
| `src/config/graph_layout/` | The one sanctioned upward import: a layout wires `core_ai_examples` components to `services.llm`. Nothing under `core_ai_examples` imports it back; `test_layer_imports.py` fails the cycle |
| `src/core_ai_examples/`, `src/services/` | Reference and integration layers, not in the wheel. `tests/integration_test/test_wheel_contents.py` fails if they leak. Structural changes here are `chore(<layer>)`: any releasing type publishes an unchanged wheel |
| `src/core_ai_examples/components/runnables/structured_grade_document/` | Branches on `getattr(model, "use_responses_api", False)`: `/v1/responses` rejects `response_format`, so it binds `text.format` and parses. Do not collapse the two routes |
| `CHANGELOG.md`, `[project].version`, `uv.lock`'s `frankstate` entry | Written by semantic-release only. Anything added by hand to the changelog sinks to the bottom |
| `.github/workflows/release.yaml`, environment `release` | Both names are bound to the PyPI trusted publisher. Renaming either breaks publishing without an error in this repo. The release commit is a direct push to `main`: any required status check on `main` rejects it (seen on 0.3.0) |
| `.github/scripts/comment_ratio.py` | The `make comment-ratio` gate: `#` lines ≤ 15% of non-blank lines per file, docstrings excluded, `src/frankstate` report-only |

## Commands (the Makefile is the single interface)

```bash
make help                      # every target
make install-dev | install-mcp # the quality env (examples + databricks) or the mcp env (examples + mcp)
make ci                        # everything the quality job runs, in CI order — run before pushing
make type test-mcp             # in the mcp env: what the mcp job runs
make lint | format | format-check | type | test | test-frankstate | cov | cov-frankstate
make comment-ratio | yaml-check | audit | pre-commit | build
make release-prepare VERSION=x # what semantic-release calls; never by hand
python main.py --layout simple_oak
```

`ci.yaml` and `release.yaml` call these same targets, so they cannot drift.

## Guardrails (DO NOT)

- Do not edit `CHANGELOG.md` or bump the version; change the commit message instead.
- Do not commit or push without the maintainer's explicit request.
- Do not create `.yml` files — always `.yaml` (`make yaml-check`).
- Do not hardcode endpoints, model names or credentials; `config_llms.yaml` declares

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aamaragones/frankenst-ai](https://github.com/aamaragones/frankenst-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
