---
trigger: always_on
description: You are in the ovstorage repository. This is the canonical agent-neutral entry
---

# ovstorage Agent Entry

You are in the ovstorage repository. This is the canonical agent-neutral entry
point; [`CLAUDE.md`](CLAUDE.md) is only a compatibility shim.

`ovstorage` is a portable storage abstraction with a stable plugin ABI. The
core Rust library, CLI, MCP server, result-envelope contract, C/C++/Python
bindings, and first-party `file://` and HTTP(S) plugins live in
[`ovstorage-core/`](ovstorage-core/). The remote-deployment surface — gRPC
broker daemon, REST gateway, authz SPI, and the in-tree TOML authz plugin —
lives in [`ovstorage-remote/`](ovstorage-remote/). Service/API contracts,
conformance material, Kubernetes deployment guidance, and service-stack
operator skills live in [`ovstorage-services/`](ovstorage-services/) as
vendored source of truth.

## Route by Task

| You want to... | Load |
|---|---|
| Use or modify the Rust library surface | [`docs/public/library-rust/AGENTS.md`](docs/public/library-rust/AGENTS.md) |
| Use or modify the Python binding | [`docs/public/library-python/AGENTS.md`](docs/public/library-python/AGENTS.md) |
| Run or extend Python examples | [`docs/public/library-python/AGENTS.md`](docs/public/library-python/AGENTS.md) and [`ovstorage-core/examples/python/README.md`](ovstorage-core/examples/python/README.md) |
| Use or modify the C++ binding | [`docs/public/library-cpp/AGENTS.md`](docs/public/library-cpp/AGENTS.md) |
| Call ovstorage over HTTP/REST | [`docs/public/library-web/AGENTS.md`](docs/public/library-web/AGENTS.md) |
| Work on storage plugin behavior | [`docs/public/plugin-storage/AGENTS.md`](docs/public/plugin-storage/AGENTS.md) and [`docs/public/plugin-development/AGENTS.md`](docs/public/plugin-development/AGENTS.md) |
| Work on the Omniverse Storage Service client plugin | [`docs/public/plugin-storage/plugin-services-client.md`](docs/public/plugin-storage/plugin-services-client.md) |
| Work on the broker-client plugin | [`docs/public/plugin-storage/plugin-broker.md`](docs/public/plugin-storage/plugin-broker.md) plus [`docs/public/plugin-storage/AGENTS.md`](docs/public/plugin-storage/AGENTS.md) |
| Author or operate an authz plugin | [`docs/public/plugin-authz/AGENTS.md`](docs/public/plugin-authz/AGENTS.md) |
| Operate the broker daemon | [`docs/public/broker-operator/AGENTS.md`](docs/public/broker-operator/AGENTS.md) |
| Operate the REST gateway | [`docs/public/library-web/AGENTS.md`](docs/public/library-web/AGENTS.md) |
| Work on repository maintenance or CI | Source-developer skills under [`skills/`](skills/) |
| Use the MCP tools or result-envelope contract | [`docs/public/agent/README.md`](docs/public/agent/README.md) |
| Work on service/API specs, service-stack deployment, or service operations | [`ovstorage-services/AGENTS.md`](ovstorage-services/AGENTS.md) |

## Personas

The five user-facing personas (one persona doc per audience):

| Persona | Audience |
|---|---|
| [`library-rust`](docs/public/library-rust/README.md) | Rust applications linking the `ovstorage` crate |
| [`library-python`](docs/public/library-python/README.md) | Python applications using the abi3-py310 wheel |
| [`library-cpp`](docs/public/library-cpp/README.md) | C / C++ applications using the C ABI cdylib + C++20 header |
| [`library-web`](docs/public/library-web/README.md) | HTTP callers using `ovstorage-rest` |
| [`agent`](docs/public/agent/README.md) | MCP / result-envelope consumers |

Plus three plugin / operator personas:

| Persona | Audience |
|---|---|
| [`plugin-storage`](docs/public/plugin-storage/README.md) | Storage backend plugin authors |
| [`plugin-authz`](docs/public/plugin-authz/README.md) | Authz plugin authors |
| [`broker-operator`](docs/public/broker-operator/README.md) | `ovstorage-broker` deployment + policy management |

## Invocable Skills

Root [`skills/`](skills/) covers the ovstorage client library, MCP surface,
plugins, broker, and repo workflows. It does not deploy or operate the heavier
`ovstorage-services` Kubernetes stack; use
[`ovstorage-services/AGENTS.md`](ovstorage-services/AGENTS.md) for those
service-stack routes.

User-facing skills:

| Skill | Use when |
|---|---|
| [`ovstorage-user-connect-via-mcp`](skills/ovstorage-user-connect-via-mcp/SKILL.md) | Connecting an agent client to the MCP stdio server |
| [`ovstorage-user-getting-started`](skills/ovstorage-user-getting-started/SKILL.md) | Discovering configured backends and routes |
| [`ovstorage-user-read-bytes`](skills/ovstorage-user-read-bytes/SKILL.md) | Reading one bounded object into memory |
| [`ovstorage-user-list-and-paginate`](skills/ovstorage-user-list-and-paginate/SKILL.md) | Listing a prefix with pagination |
| [`ovstorage-user-write-safely`](skills/ovstorage-user-write-safely/SKILL.md) | Writing without accidental clobbering |
| [`ovstorage-user-delete-safely`](skills/ovstorage-user-delete-safely/SKILL.md) | Deleting objects or directories safely |
| [`ovstorage-user-materialize`](skills/ovstorage-user-materialize/SKILL.md) | Getting a stable local path for large/random-access reads |
| [`ovstorage-user-handle-errors`](skills/ovstorage-user-handle-errors/SKILL.md) | Interpreting `ok: false` envelope errors |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/ovstorage](https://github.com/NVIDIA-Omniverse/ovstorage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
