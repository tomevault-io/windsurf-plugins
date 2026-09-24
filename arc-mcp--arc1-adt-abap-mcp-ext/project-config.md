---
trigger: always_on
description: Guidance for AI assistants (and humans) working on `com.arc1.mcp` (the
---

# CLAUDE.md

Guidance for AI assistants (and humans) working on `com.arc1.mcp` (the
Eclipse plugin in this repo). Read this first.

## Project goal

**Extend SAP's Model Context Protocol (MCP) server inside Eclipse-for-ABAP**
with extra read-only tools, so AI clients (Claude Code, GitHub Copilot,
Cursor, Claude Desktop) can read your ABAP system without any extra process.

Requires **ADT 3.60+**, where SAP ships the MCP server as a supported feature.
The user installs one JAR, turns SAP's server on once (the *ABAP Development →
MCP Server* preference + `-DadtMcpServerPrefEnabled=true` in `eclipse.ini`),
and restarts. From then on, this plugin's tools register on SAP's authenticated
MCP endpoint (default `http://localhost:2234/mcp`) whenever the server runs.

### Why this exists

SAP ships the MCP server inside ADT. In 3.58/3.59 it was dormant with no
activation surface; **as of ADT 3.60 it is a supported feature** with its own
preference page (*ABAP Development → MCP Server*) and a startup flag
(`-DadtMcpServerPrefEnabled=true`) — but it's **off by default** and, on its
own, only carries SAP's own tools. SAP exposes a public Eclipse extension point
`com.sap.adt.mcp.core.adtMcpTools` for contributing extra tools. This plugin
does exactly one thing:

1. **Contribute extra tools** via that documented extension point. SAP's own
   `ToolRegistrationService` registers them whenever the server starts.

(It also optionally pre-warms the ABAP project logon so tools work on the first
call — public API, not a hack.)

Earlier versions (≤ 0.3.x) also reflectively kickstarted the dormant 3.58
server. ADT 3.60 made that both unnecessary (SAP ships activation) and broken
(the `startMCPServer` signature changed to take a `FileSystemMode`), so v0.4.0
removed it — the plugin no longer touches the server lifecycle. See
`docs/decisions.md` D9 (supersedes D3).

### Non-goals

This plugin is intentionally **Eclipse-bound**. It is NOT trying to be:
- A standalone MCP server (that's [ARC-1](https://github.com/marianfoo/arc-1),
  in TypeScript, runs anywhere).
- A managed multi-user service with admin policy ceilings, audit, BTP
  deployment (also ARC-1 territory).
- A write/activate platform — mutating tools belong in SAP's own MCP
  surface (`abap_transport-create`, `abap_generators-generate_objects`),
  which SAP ships and registers itself.
- A headless / programmatic / CI driver for ABAP — that's
  [`adt-ls`](https://github.com/marianfoo/adt-ls) (a TypeScript SDK over SAP's
  headless `adt-ls` language server). We do **not** consume it as a dependency
  (it'd mean a Node process + a second headless ADT inside the real one) — see
  `docs/decisions.md` D10.

If a task is "centralized management", "BTP", or "non-Eclipse" — point the
user at ARC-1 instead. If it's "headless", "programmatic", or "from CI/Node" —
point them at `adt-ls`. Neither belongs inside this plugin.

## Architecture in one screen

```
Eclipse workbench startup
  │
  ├─ OSGi resolves bundles (incl. com.arc1.mcp from dropins/)
  │     com.arc1.mcp requires com.sap.adt.* [3.60.0,4.0.0)
  │
  ├─ SAP's AdtMcpUIStartupHandler.earlyStartup() (via org.eclipse.ui.startup)
  │   └─ starts the server IF -DadtMcpServerPrefEnabled=true AND pref enabled
  │        → AdtMCPCorePlugin.startMCPServer(port, token, FileSystemMode.SFS)
  │
  ├─ Arc1Startup.earlyStartup()  (via org.eclipse.ui.startup)
  │   ├─ log guidance (how to enable the server)
  │   └─ schedule Arc1AutoLogin Job (2s delay), unless -Darc1.mcp.autologin=false
  │
  └─ On server start, SAP's ToolRegistrationService discovers all
       <mcpTool class="..."/> extension contributions (SAP's + ours) and
       addTool()s them on the McpSyncServer (Java MCP SDK).
```

Request flow when a client calls our tool:

```
Client → POST /mcp (Streamable HTTP) with Bearer token
   ↓ DNSRebindingProtectionFilter (Host: localhost?)
   ↓ TokenAuthenticationFilter
   ↓ Java MCP SDK servlet
   ↓ ToolRegistrationService routes by name
   ↓ Arc1Sap<X>Tool.execute(jsonInput)
   ↓ For HTTP-backed tools: AdtHttp.get/post(destinationId, uri, ...)
   ↓ Eclipse's IStatelessSystemSession handles auth/cookies/CSRF
   ↓ SAP ABAP backend
```

## Repo layout

```
arc1-mcp-ext/
├── build.sh                     javac + jar, ~50 lines, no Maven
├── plugin.xml                   extension contributions
├── META-INF/MANIFEST.MF         OSGi bundle headers
├── src/com/arc1/mcp/
│   ├── Arc1McpActivator         OSGi Plugin singleton + log
│   ├── Arc1Startup              IStartup; guidance log + autologin trigger
│   ├── Arc1AutoLogin            Background Job, ensureLoggedOn
│   ├── AdtHttp                  HTTP helper (GET + POST, 256KB cap)
│   ├── Json                     no-dep JSON helpers
│   └── Arc1Sap*Tool             one class per MCP tool
├── scripts/
│   ├── smoke-test.sh            end-to-end test of every tool
│   └── finalize-readme.sh       swap repo URL placeholders
├── docs/
│   ├── architecture.md          deeper than this file
│   ├── decisions.md             non-obvious design choices (D1–D9)
│   ├── plans/                   01–06; one per release
│   ├── research/                bytecode analysis pointers
│   └── release-readiness-review.md
├── .github/workflows/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arc-mcp/arc1-adt-abap-mcp-ext](https://github.com/arc-mcp/arc1-adt-abap-mcp-ext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
