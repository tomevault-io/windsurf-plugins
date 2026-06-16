---
trigger: always_on
description: Agentix targets **agent eval**, **RL rollouts**, and **rollout data
---

# Project Conventions

## Product context

Agentix targets **agent eval**, **RL rollouts**, and **rollout data
collection** (via `agentix.utils.trace` + `abridge`). Positioning: a friendlier
alternative to HTTP rollout servers such as
[ProRL-Agent-Server](https://github.com/NVIDIA-NeMo/ProRL-Agent-Server)—
integrate with importable callables and `client.remote(fn, ...)`, not
custom `AgentHandler` services. Public slogan: *The universal bridge
between agents and environments.*

## Two Concepts

Agentix has exactly two ideas:

1. **Remote calls** — `c.remote(fn, *args, **kwargs)` calls an
   importable Python function inside a sandbox worker. The target is
   `fn.__module__ + "::" + fn.__qualname__`; args/kwargs travel as a
   single pickle blob and the return value is unpickled host-side.
2. **Bundle** — `agentix build [path]` packages a Python project and
   its declared dependencies into a deploy-ready Docker image. The
   project's `[project].dependencies` defines what modules are
   installed into the runtime venv.

The primary user model is:

```python
from app import run

result = await client.remote(run, input="hello")
```

`import app; await client.remote(app.run, ...)` also works because it
passes the same function object.

## Three Built-In Systems

agentix-core ships **three** independent systems, mapped to three
reserved Socket.IO namespaces:

| Namespace | System  | Public API                                      |
|-----------|---------|-------------------------------------------------|
| `/rpc`    | RPC     | `client.remote(fn, ...)`                        |
| `/trace`  | tracing | `agentix.utils.trace.span(...)` / `trace.Processor` |
| `/log`    | logging | stdlib `logging` (auto-bridged sandbox → host) |

Plugins (`abridge`, future LLM tools, ...) MUST live on their own
namespace `/<package-name>`. Two plugins can never collide because
PyPI package names are globally unique.

## Composition Over Inheritance

Use inheritance only for genuine lifecycle interfaces:
- `SandboxProvider` Protocol for provider backends
- `agentix.Namespace` / `agentix.AsyncClientNamespace` for plugin SIO
  handlers (mirrors `socketio.AsyncClientNamespace`)
- `trace.Processor` for trace sinks

Everywhere else, prefer normal functions, Protocols, composition
objects, or callbacks. A remote target is just a Python callable
serialized by stdlib pickle — there is no base class for user code to
inherit from.

## No Backward Compatibility Shims

This repo is in active design. Breaking changes are fine.

- Rename by deleting the old name, not by accepting both.
- Do not add deprecation warnings.
- Do not leave comments explaining removed behavior.
- Update tests to the current shape; do not preserve tests for removed
  behavior.

## Monorepo Layout

Everything lives in this one repo, wired as a **uv workspace** — the
core, the plugins, and the cookbook examples. Edit any file and it's
live in the shared venv; there is no commit → push → publish cycle for
day-to-day iteration.

```text
Agentix/                       — repo root = workspace root
├── pyproject.toml             — `agentixx` core package + [tool.uv.workspace]
├── uv.lock                    — one lock for the whole workspace
├── agentix/                   — core source (see Systems Map below)
├── tests/                     — core tests
├── plugins/
│   ├── abridge/               — `agentix-bridge` (import `agentix.bridge`)
│   │   ├── pyproject.toml
│   │   ├── agentix/bridge/
│   │   └── tests/
│   ├── providers/
│   │   ├── docker/            — `agentix-provider-docker` → `docker` + `podman`
│   │   ├── daytona/           — `agentix-provider-daytona` → `daytona`
│   │   ├── e2b/               — `agentix-provider-e2b` → `e2b`
│   │   └── apptainer/         — `agentix-provider-apptainer` → `apptainer`
│   └── runtime-basic/         — `agentix-runtime-basic` → `bash` + `files`
└── examples/
    └── eval-cc-swe/           — `eval-cc-swe` cookbook example
```

`[tool.uv.workspace] members = ["plugins/*"]` — drop a plugin dir under
`plugins/` and it is a workspace member; `uv sync --all-packages`
installs it editable.

Each provider-backend member is a single module that contributes a
sibling into the core `agentix/provider/` namespace (e.g.
`agentix/provider/docker.py`). The dirs carry no `__init__.py` — that
file belongs to the core. The backend is wired in by its
`[project.entry-points."agentix.provider"]`, which the `Registry`
discovers via `importlib.metadata` — so an editable workspace install
makes `from agentix.provider.docker import DockerProvider` work and
`providers().get("docker")` / `providers().get("podman")` resolve (the
string registry powers the CLI; typed code imports the class directly).

Dependency separation is preserved: each member has its own
`pyproject.toml` + dependency list. The core never pulls a plugin's
deps — `openai` belongs to `agentix-bridge`; the E2B/Daytona SDKs
belong to their backend members, not `agentixx`. Members reference each
other with `[tool.uv.sources] <dep> = { workspace = true }` (editable,
no fetch).

`runtime-basic` is a *sandbox-side* member — it ships the `bash` +
`files` namespaces and their `default.nix` data files into the
`agentix build` bundle. The provider members are *host-side*. Both

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agentix-Project/Agentix](https://github.com/Agentix-Project/Agentix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
