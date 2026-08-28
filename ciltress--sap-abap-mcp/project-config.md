---
trigger: always_on
description: Guidance for AI agents working **on this repository**. If you are an agent _using_ the running server
---

# AGENTS.md

Guidance for AI agents working **on this repository**. If you are an agent _using_ the running server
to work on ABAP, you want [`docs/MCP-Tools.md`](docs/MCP-Tools.md) instead.

## What this is

An MCP server that gives an agent read/write access to an SAP ABAP system over **ADT**, authenticated
with **SPNEGO/Kerberos SSO**, an **X.509 client certificate** or an **OAuth 2.0 bearer token** — none of
which involves a password (there is a password mode too, as a last resort). It wraps
[`abap-adt-api`](https://github.com/marcellourbani/abap-adt-api) and adds one thing ADT cannot do:
**calling RFC-enabled function modules**, over the SAP Gateway JSON-RPC service.

## Read these first

| Document                                                       | When you need it                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **[`docs/Tool-Router.md`](docs/Tool-Router.md)**               | Intent to tool, hand-written: "short dump", "where-used", "SE16". The cheapest useful thing to read, and the right first stop when you know the job but not the tool.                                                                                                                                                                                                                                                                                                                    |
| **[`docs/Tool-Reference.md`](docs/Tool-Reference.md)**         | Every tool with its arguments, **generated** from `getTools()` by `npm run docs:tools`. Never edit it by hand.                                                                                                                                                                                                                                                                                                                                                                           |
| **[`docs/MCP-Tools.md`](docs/MCP-Tools.md)**                   | The rules a generator cannot produce: golden-path workflows, ADT URI and lock semantics, the response/error model, a troubleshooting matrix, and the test-suite map. **Start here** for how the server behaves.                                                                                                                                                                                                                                                                          |
| **[`docs/ABAP-Skills.md`](docs/ABAP-Skills.md)**               | The bundled ABAP skills, which of them use these tools, and the two traps found while wiring them up.                                                                                                                                                                                                                                                                                                                                                                                    |
| **[`docs/Development-Skills.md`](docs/Development-Skills.md)** | The bundled general engineering skills, and which fit this repo.                                                                                                                                                                                                                                                                                                                                                                                                                         |
| **[`docs/JSON-RPC.md`](docs/JSON-RPC.md)**                     | The design and protocol record for the RFC path: the wire protocol read from `/IWBEP/CL_JSRPC_*`, why a batch shares one LUW, and the traps that bite. Read it before touching `JsonRemoteFunctionCallHandlers.ts` or anything that calls a function module.                                                                                                                                                                                                                             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ciltress/sap-abap-mcp](https://github.com/Ciltress/sap-abap-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
