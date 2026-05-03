---
trigger: always_on
description: - **No code samples in documentation.** Documentation files should describe concepts in prose and tables, not paste code. The code lives in the code.
---

# Spectral — Project Specification

## Style preferences

- **No code samples in documentation.** Documentation files should describe concepts in prose and tables, not paste code. The code lives in the code.
- **Import convention.** All imports go at the top of the file (standard Python style). Internal CLI code (`cli/…`) is always imported at the top level — no lazy imports. Heavy external dependencies (e.g., `mitmproxy`, `yaml`, `pydantic_ai`) may be lazy-imported inside helper functions to keep `spectral --help` fast. Optional dependencies may use a lazy try/except with a user-friendly error. For mutable module-level state (e.g., `_model` in `llm/_client.py`), import the *module* at the top and access the attribute at call time to avoid stale references.
- **Private function convention.** Every module should clearly separate its public API from internal helpers. Prefix internal functions with `_` (e.g. `_build_timeline_text`, `_resolve_url`). The public API of a module is the set of unprefixed functions and classes — only those should be imported by other packages. Tests may import private functions for unit testing, but production code in other packages should only depend on the public API.

## Development environment

- Package manager is **uv**. Use `uv run` to execute commands (no need to activate the venv):
  - `uv run pytest tests/` — run tests
  - `uv run spectral mcp analyze ...` — run the CLI
  - `uv add <package>` — add a dependency (updates `pyproject.toml` + `uv.lock`)
  - `uv add --dev <package>` — add a dev dependency
- The LLM provider and API key are resolved from stored config at `~/.local/share/spectral/config.json` or created interactively on first use via `spectral config`. Supports Anthropic, OpenRouter, OpenAI, and Ollama. No `.env` file needed.
- **Before finishing any code change**, run the full verification suite and fix any new errors:
  - `uv run pytest tests/ -x -q` — all tests must pass
  - `uv run ruff check` — zero lint errors (use `--fix` for auto-fixable import sorting)
  - `uv run pyright` — zero new type errors (pre-existing errors in `proxy.py`, `test_proxy.py` are known)
- **Shell completion scripts** (`cli/completions/spectral.bash` and `spectral.zsh`) are static — they must be updated manually whenever a CLI command, subcommand, or option is added, removed, or renamed.
- **Conventional Commits** are mandatory. Every commit message must follow the format `type(scope): description` (e.g. `fix:`, `feat:`, `chore:`, `docs:`, `refactor:`, `test:`, `ci:`). This drives python-semantic-release: `fix:` bumps patch, `feat:` bumps minor, `BREAKING CHANGE:` bumps major. Use `chore:`/`docs:`/`refactor:`/`test:`/`ci:` for changes that should not trigger a release.

## What this project is

A four-stage pipeline that automatically discovers, documents, and exposes web application APIs:

1. **Capture** — A Chrome Extension or MITM proxy records network traffic + UI actions while the user browses normally
2. **Analyze** — A CLI tool correlates UI actions with API calls using an LLM. REST traces produce an OpenAPI 3.1 spec; GraphQL traces produce a typed SDL schema. Both are enriched with business semantics
3. **Authenticate** — The CLI detects the app's auth flow and generates a login script. Run it once to obtain a session; the MCP server refreshes it automatically
4. **Use** — Generated MCP tools let AI agents call the discovered API directly

The key innovation is the **correlation of UI actions with network traffic** to understand the *business meaning* of each API call, not just its technical shape.

## Project structure

```
spectral/
├── extension/              # Chrome Extension (Manifest V3)
│   ├── background/         # Service worker modules (background.js, network.js, websocket.js, graphql.js, capture.js, native.js)
│   ├── content/            # UI context capture (content.js)
│   └── popup/              # Extension popup UI
├── cli/                    # Python CLI tool
│   ├── main.py             # Entry point: wires command groups
│   ├── commands/
│   │   ├── openapi/        # REST analysis → OpenAPI 3.1 YAML
│   │   ├── graphql/        # GraphQL analysis → SDL schema
│   │   ├── mcp/            # MCP tool generation and stdio server
│   │   ├── auth/           # Authentication management
│   │   ├── capture/        # Bundle parsing, inspect, MITM proxy
│   │   ├── extension/      # Chrome Native Messaging host (listen, install)
│   │   ├── analyze/        # Shared analysis engine (pipeline, steps, correlator, protocol, schemas)
│   │   └── android/        # Android APK tools (list, pull, patch, install, cert)
│   ├── formats/            # Pydantic models (capture_bundle, mcp_tool, app_meta)
│   └── helpers/            # Shared utilities (llm, storage, naming, console, http, auth_framework)
├── tests/                  # Mirrors cli/ structure
├── pyproject.toml
└── README.md
```

## Data model convention

| Pattern | Contents | Python construct |
|---------|----------|-----------------|
| `cli/formats/<name>.py` | Serialization models (external formats: capture bundle, API spec) | Pydantic `BaseModel` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spectral-mcp/spectral](https://github.com/spectral-mcp/spectral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
