---
trigger: always_on
description: This file gives Claude (and humans) what they need to operate this repo.
---

# CLAUDE.md — okto-pulse

This file gives Claude (and humans) what they need to operate this repo.
Project overview lives in `README.md`; this file is for procedures.

This repo is the deliverable: CLI, embedded frontend, Dockerfile, and the
release pipeline. It depends on `okto-pulse-core` (a sibling repo) for the
actual engine.

---

## Docker architecture

The deployable artifact is a single image: `ghcr.io/oktolabsai/okto-pulse:<tag>`.
Everything else (compose files, wheels, attestations) is produced from this repo
plus the `okto-pulse-core` sibling.

### Multi-stage Dockerfile

```
python:3.12-slim
    │
    ├─ base                  apt deps + python env
    │
    ├─ wheel-builder         COPY okto-pulse/ + okto-pulse-core/ (siblings)
    │                        → python -m build → /wheels/*.whl
    │   ↓
    ├─ local-install         COPY pyprojects + uv.lock → uv pip install
    │                        --frozen → /wheels/*.whl on top
    │   ↓
    └─ local-runtime         pre-download all-MiniLM-L6-v2,
                             verify model cache presence, EXPOSE 8100/8101,
                             HEALTHCHECK, CMD ["okto-pulse", "serve"]
    │
    ├─ pypi-install          uv pip install okto-pulse==${OKTO_PULSE_VERSION}
    │   ↓
    └─ pypi-runtime          same finalizer as local-runtime
```

Two **independent finalizer paths** (`local-runtime`, `pypi-runtime`) so dev and
prod don't share an install layer that could mask a publishing bug. They emit
the same runtime contract: ports 8100/8101, `okto-pulse serve` as PID 1, healthcheck on
`/api/v1/kg/settings`, model cache at `/opt/hf-cache`.

### Compose files

| File | Target | Build context | When to use |
|------|--------|----------------|-------------|
| `docker-compose.yml` | `local-runtime` | `..` (parent of `okto-pulse/`) | Hacking on `okto-pulse-core/` and `okto-pulse/` together. Wheels built from local source. |
| `docker-compose.prod.yml` | `pypi-runtime` | `.` | Pulling a pinned `okto-pulse==X.Y.Z` from PyPI. No sibling repo needed. |

Both bind host ports to `127.0.0.1` only and set `HOST=0.0.0.0` /
`MCP_HOST=0.0.0.0` inside the container so port-mapping actually reaches the
listeners. (Without those env vars, uvicorn binds to the container's loopback
interface and Docker's port mapping silently produces "Connection reset by peer"
on the host side.)

### Why the sibling-checkout pattern

The `local-runtime` target needs both repos at the same commit/tag so an image
built in CI is reproducible from source. The release pipeline does sibling
checkouts of `okto-pulse-core@vX.Y.Z` and `okto-pulse@vX.Y.Z`, sets the build
context to the parent directory, and the Dockerfile's `wheel-builder` stage
`COPY`s both into `/src/`. The `okto-pulse/pyproject.toml` dep pin
(`okto-pulse-core>=X.Y.Z,<1.0.0`) is a floor for the PyPI install path; it does
NOT control which core source is built into the local-runtime image.

### Env vars the runtime reads

| Variable | Default | Purpose |
|----------|---------|---------|
| `HOST` | `127.0.0.1` | API/UI uvicorn bind host. Read by `CommunitySettings.host` (pydantic-settings, no prefix). |
| `MCP_HOST` | `127.0.0.1` | MCP uvicorn bind host. Read by `community/main.py` (since v0.1.12) AND `core/mcp/server.py:run_mcp_server()` standalone. |
| `DATA_DIR` | `~/.okto-pulse` | SQLite + KG root. Set to `/data` in compose. |
| `KG_BASE_DIR` | derived from `DATA_DIR` | Per-board graph storage. |
| `HF_HOME` | `~/.cache/huggingface` | Pre-warmed to `/opt/hf-cache` in the image. |
| `MCP_PORT` / API port | from CLI flags or `settings.mcp_port` / `settings.port` | Override port numbers without remapping in compose. |
| `MCP_TRACE_ENABLED` | unset | `=1` records every MCP call to `${MCP_TRACE_DIR}/session_*.jsonl`. |
| `MCP_TRACE_DIR` | `${KG_BASE_DIR}/mcp_traces` | Trace output directory when tracing is enabled; falls back to `./mcp_traces` when `KG_BASE_DIR` is unset. |

**MCP_HOST runtime path gotcha:** there are TWO uvicorn callers in the codebase.
`okto-pulse-core/.../mcp/server.py:run_mcp_server()` is only used when running
the MCP server standalone (`python -m okto_pulse.core.mcp.server`). The
deployed container runs `okto-pulse serve`, which uses the dual-port runner in
`okto-pulse/src/okto_pulse/community/main.py`. Both honor `MCP_HOST` since
v0.1.12 — but pre-v0.1.12 only the standalone path did. If anyone backports
older releases, watch for the regression: setting `MCP_HOST=0.0.0.0` would do
nothing inside the container.

### Healthcheck

```
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
    CMD curl -fsS http://127.0.0.1:8100/api/v1/kg/settings || exit 1
```

The healthcheck curls **inside the container** so it always uses 127.0.0.1
even when the public binding is 0.0.0.0. A container can be `healthy` but
unreachable from the host if the listener bound to 127.0.0.1 (uvicorn loopback
inside container ≠ Docker's NAT loopback). Always check the uvicorn startup
line in `docker logs` when "container healthy + curl fails from host":

```
INFO:     Uvicorn running on http://0.0.0.0:8100   ← reachable
INFO:     Uvicorn running on http://127.0.0.1:8100 ← NOT reachable from host
```

### Local Docker dry-run


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OktoLabsAI/okto-pulse](https://github.com/OktoLabsAI/okto-pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
