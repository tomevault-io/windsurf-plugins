---
trigger: always_on
description: Microkernel Python agent runtime with MCP-native tools, a local model
---

# AgentOS — Agent Instructions

Microkernel Python agent runtime with MCP-native tools, a local model
router, durable memory, a layered sandbox, and multi-channel messaging
(Web UI, CLI, chat). One shared turn loop drives every client.

- **Language / runtime:** Python **3.12+** only.
- **Package manager:** **uv** (not pip/poetry). Never edit lockfiles by hand.
- **License:** Apache-2.0 (see `LICENSE` + `NOTICE`; core derived from
  OpenSquilla — attribution in `THIRD_PARTY_NOTICES.md`). Default LLM
  provider: **OpenRouter** (Bankr selectable).
- **PRs target `main`.** Keep them small, focused, and test-covered.

## Setup

```sh
uv sync --extra dev --extra recommended                        # dev deps + local embeddings
uv sync --extra dev --extra recommended --extra mcp --frozen   # match CI exactly
```

## Quality gate — run before every commit / PR

Run these in order; all must pass. This mirrors CI (`.github/workflows/ci.yml`):

```sh
uv run ruff check src tests                    # lint (E, F, I, N, W, UP; line-length 100)
uv run mypy src/agentos --show-error-codes     # type check
uv run pytest -q                               # full test suite (~490 test files)
uv build --wheel                               # packaging must not break
```

Fast inner loop while iterating:

```sh
uv run pytest tests/test_engine -q                 # one subsystem
uv run pytest tests/path/test_x.py::test_name -q   # one test
uv run ruff check src/agentos/<changed>            # lint just what you touched
uv run ruff format src tests                       # autofix formatting
```

### Test policy

- Default tests **must be offline, deterministic, credential-free, and fork-safe.**
  Do **not** add network, provider, browser, or channel dependencies to the
  default `pytest` path.
- Live/costly tests are opt-in via markers and excluded from the default run:
  `llm`, `llm_smoke`, `llm_costly`, `llm_tools`, `llm_gateway`, `llm_image`,
  `webui_browser`, `live_channel`, etc. Run a marker group explicitly, e.g.
  `uv run pytest -m llm_smoke`. Skip them all with `-m "not llm"`.
- Add or update a **public regression test** for every behavior change or bugfix.
- Private/maintainer-only fixtures live under `tests/_private/` (excluded from
  collection and from the public tree). Never commit real transcripts,
  credentials, channel IDs, or local paths.

> A small set of pre-existing failures (some `onboard_cmd` ANSI-width tests and
> Docker-migration tests) fail independently of your change — verify against a
> clean tree before attributing a failure to your work.

## Source layout — `src/agentos/`

Every client hits one local **gateway**; the gateway runs turns through the
shared **engine** loop, which calls the **router** to pick a model and executes
**tools** inside the **sandbox**.

| Package | Responsibility |
|---|---|
| `engine/` | Agent core state machine / turn loop (lazy public surface — import `agentos.engine.types` without dragging in the world). |
| `gateway/` | ASGI gateway: WebSocket, middleware, RPC, sessions, approvals, task runtime. |
| `agentos_router/` | Local model router — picks the cheapest capable model per turn. Default strategy `v4_phase3` (on-device ML, BGE+LightGBM, no LLM call); alternative `llm_judge` strategy tiers each turn via a small LLM call. |
| `provider/` | Unified LLM provider abstraction (OpenRouter, Bankr, OpenAI, Anthropic, Ollama, DeepSeek, Gemini, Qwen, …). |
| `tools/` | Tool Registry + built-in tools (`ToolContext`, `RegisteredTool`, `ToolError`). |
| `sandbox/` | Layered sandbox & security grading. Safe to import at startup (no subprocess/FS probing). |
| `memory/` | Durable persistent memory + on-device embeddings. `memory/models/` holds committed ONNX weights — **do not delete**. |
| `skills/` | Six-layer skill system; bundled skills in `skills/bundled/`, experimental in `skills/exp/`. |
| `channels/` | Channel adapters: Terminal, WebSocket, Slack, Discord, Telegram, and others. |
| `cli/` | Typer CLI. Entry points: `agentos` → `cli.main:app`, `gateway` → `cli.main:gateway_app`. |
| `mcp/`, `mcp_server/` | MCP client + AgentOS-as-MCP-server. |
| `onboarding/` | First-run setup / provider selection. |
| `scheduler/` | Cron / scheduled agent work. |
| `search/` | Built-in web search + content extraction. |
| `session/`, `persistence/` | Session state and SQLite/SQLModel storage (migrations under top-level `migrations/`). |
| `safety/`, `identity/`, `health/`, `observability/` | Guardrails, identity, health checks, structured logging/metrics. |

Docs live in `docs/` (`docs/README.md` is the index); helper scripts in `scripts/`.

## Conventions

- **Style is enforced by ruff** — target `py312`, line length **100**, rule sets
  `E,F,I,N,W,UP`. Run `ruff format` rather than hand-aligning. Match the
  surrounding file's idiom, naming, and comment density; don't restyle
  untouched code.
- **Typing:** new/changed code should pass `mypy` (`warn_return_any`,
  `warn_unused_configs`). Don't silence errors with a blanket `# type: ignore` —
  narrow the type, or add a scoped override in `pyproject.toml` only when a
  module is genuinely unstubbed.
- **Config / env:** runtime settings come from `AGENTOS_*` env vars

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [use-agent-os/agent-os](https://github.com/use-agent-os/agent-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
