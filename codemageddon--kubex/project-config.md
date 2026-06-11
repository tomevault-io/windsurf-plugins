---
trigger: always_on
description: Kubex is an async-first Kubernetes client library for Python, inspired by [kube.rs](https://kube.rs/). It is built on Pydantic v2 and is async-runtime agnostic (supports asyncio and trio). The project is in **beta** (v0.1.0-beta.1) — backward compatibility may still break between releases.
---

# CLAUDE.md

## Project Overview

Kubex is an async-first Kubernetes client library for Python, inspired by [kube.rs](https://kube.rs/). It is built on Pydantic v2 and is async-runtime agnostic (supports asyncio and trio). The project is in **beta** (v0.1.0-beta.1) — backward compatibility may still break between releases.

**Documentation site:** https://kubex.codemageddon.me/

**Implementation plans** live at `.ralphex/plans/` (not `docs/`).

## Quick Reference

```bash
# Install dependencies (requires uv)
uv lock --python 3.13 && uv sync --python 3.13 --all-extras

# Run tests (requires Docker for testcontainers/K3S)
uv run pytest .

# Lint
uv run ruff check .

# Format check
uv run ruff format --check .

# Auto-fix lint + format
uv run ruff check --fix . && uv run ruff format .

# Type check (strict mode)
uv run mypy .

# Run pre-commit hooks
pre-commit run --all-files

# Serve docs locally with live reload
mise run docs:serve

# Build docs in strict mode (--strict turns warnings into errors)
mise run docs:build

# Regenerate all K8s model packages (downloads specs + runs codegen + verifies)
mise run regenerate-models
```

## Repository Structure

```
kubex/                          # Main package — PEP 420 namespace package (no __init__.py) so the
                                #   workspace `kubex-k8s-*` packages can contribute `kubex.k8s.*` submodules.
                                #   Public API is imported from explicit submodules:
                                #   `from kubex.api import Api, create_api`,
                                #   `from kubex.client import BaseClient, create_client, ClientOptions`,
                                #   `from kubex.configuration import ClientConfiguration`
├── __version__.py              # Version string (0.1.0-beta.1)
├── py.typed                    # PEP 561 type hint marker
├── api/                        # High-level API layer
│   ├── api.py                  # Api[ResourceType] generic class + create_api() factory
│   ├── _logs.py                # LogsAccessor + _LogsDescriptor — api.logs.get() and api.logs.stream()
│   ├── _scale.py               # ScaleAccessor + _ScaleDescriptor — api.scale.get(), replace(), patch()
│   ├── _status.py              # StatusAccessor + _StatusDescriptor — api.status.get(), replace(), patch()
│   ├── _eviction.py            # EvictionAccessor + _EvictionDescriptor — api.eviction.create()
│   ├── _ephemeral_containers.py # EphemeralContainersAccessor + _EphemeralContainersDescriptor — api.ephemeral_containers.get(), replace(), patch()
│   ├── _resize.py              # ResizeAccessor + _ResizeDescriptor — api.resize.get(), replace(), patch()
│   ├── _exec.py                # ExecAccessor + _ExecDescriptor + ExecResult — api.exec.run(), api.exec.stream()
│   ├── _attach.py              # AttachAccessor + _AttachDescriptor — api.attach.stream() (no run(); attaches to existing container process)
│   ├── _stream_session.py      # _BaseChannelSession (shared lifecycle base) + StreamSession — multiplexes Kubernetes channel-protocol streams over WebSocketConnection (used by exec and attach)
│   ├── _portforward.py         # PortforwardAccessor + _PortforwardDescriptor + PortForwarder + PortForwardStream (ByteStream) — api.portforward.forward(), api.portforward.listen()
│   ├── _portforward_session.py # PortForwardSession(_BaseChannelSession) — port-aware v4 channel multiplexer with per-port ByteStream + error iterator (kubelet portforward registers v4 only)
│   ├── _metadata.py            # MetadataAccessor — api.metadata.get(), list(), patch(), watch()
│   └── _protocol.py            # ApiProtocol[ResourceType], type aliases, SubresourceNotAvailable, namespace helpers
├── client/                     # HTTP client implementations
│   ├── client.py               # BaseClient ABC, create_client() factory, ClientChoise enum
│   ├── options.py              # ClientOptions pydantic model — timeout, log_api_warnings, proxy, keep_alive, keep_alive_timeout, buffer_size, ws_max_message_size, pool_size, pool_size_per_host, trust_env
│   ├── websocket.py            # WebSocketConnection ABC — abstraction used by exec and attach subresources
│   ├── httpx.py                # HttpxClient implementation (exec via httpx-ws)
│   └── aiohttp.py              # AioHttpClient implementation (exec via aiohttp ws_connect)
├── configuration/              # Auth and cluster config
│   ├── configuration.py        # ClientConfiguration, KubeConfig pydantic models
│   ├── file_config.py          # configure_from_kubeconfig() — kubeconfig file parsing
│   ├── incluster_config.py     # configure_from_pod_env() — in-cluster service account auth
│   └── auth/                   # Authentication mechanisms
│       ├── exec.py             # Exec provider authentication
│       ├── oidc.py             # OIDC provider (in progress)
│       └── refreshable_token.py # Token refresh logic
└── core/                       # Request/response primitives

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codemageddon/kubex](https://github.com/codemageddon/kubex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
