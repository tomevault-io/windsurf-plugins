---
trigger: always_on
description: aspire publish --apphost src/AgentForge.AppHost/AgentForge.AppHost.csproj -o artifacts/aspire-output
---

# Copilot Instructions — AgentForge Multi-Vertical Platform

## Build & Run

```bash
# Build entire solution (zero warnings expected)
dotnet build

# Start all services (Aspire orchestrates WAHA container, McpHost, WebApi, DevTunnel)
aspire start

# Generate Docker Compose deployment artifacts
aspire publish --apphost src/AgentForge.AppHost/AgentForge.AppHost.csproj -o artifacts/aspire-output

# Run a single project
dotnet run --project src/AgentForge.McpHost
dotnet run --project src/AgentForge.WebApi
```

There are no automated test projects in this repository. Verification is done by running the app and exercising the webhook manually (see `src/AgentForge.WebApi/AgentForge.WebApi.http`).

## Architecture Overview

This is a **.NET Aspire** solution with eight projects. The repository should now be treated as **AgentForge**, a reusable WhatsApp AI platform with a travel reference vertical currently shipped in-tree.

| Project | Role |
|---|---|
| `AgentForge.AppHost` | Aspire orchestrator — wires up all services, secrets, and the DevTunnel |
| `AgentForge.Hosting` | Custom Aspire integration (`AddWaha`) that encapsulates the WAHA Docker container |
| `AgentForge.McpHost` | Generic MCP host — loads tools/resources from the active vertical plugin and exposes them over `StreamableHttp` at `/mcp` |
| `AgentForge.WebApi` | Generic AI gateway — receives WhatsApp webhooks, runs the active vertical agent, sends replies |
| `AgentForge.ServiceDefaults` | Shared defaults — OpenTelemetry, health checks, HTTP resilience, service discovery |
| `AgentForge.Verticals.Abstractions` | Shared contracts for vertical metadata, scheduled actions, and host/plugin messaging |
| `AgentForge.Verticals.Hosting` | Shared loader layer that resolves the active in-process vertical for both hosts |
| `AgentForge.Verticals.Travel` | Current in-tree travel vertical implementation: agent metadata, prompt, tools, resources, data, and scheduled action behavior |

### Message flow

```
WhatsApp → WAHA container → DevTunnel → /webhook (WebApi)
    → WhatsAppMessageQueue (Channel<T>)
    → AgentChatService → VerticalAgentFactory (active vertical agent / ChatClientAgent)
        → AgentForge.McpHost (tools/resources from active vertical plugin over StreamableHttp)
        → Azure AI Foundry (GPT-5.4 mini)
    → WahaApiClient → WAHA → WhatsApp
```

**Key design decisions:**
- The `WhatsAppMessageQueue` is a bounded `Channel<T>` (capacity 200, `DropOldest`). The webhook returns `200 OK` immediately and processing happens asynchronously in a `BackgroundService`.
- Conversation history is **client-managed** (`AgentSessionStore`, keyed by phone number). The Azure AI Foundry chat completions API does not support server-managed history — do not use the `conversationId` overload of `CreateSessionAsync`.
- **Retries are intentionally disabled** on all HTTP clients (via `ServiceDefaults`). Retrying `WahaApiClient.SendTextAsync` would deliver the same WhatsApp message multiple times.
- `VerticalAgentFactory` uses double-checked locking (`SemaphoreSlim`) to lazily initialise the `ChatClientAgent` for the active vertical exactly once — MCP tool discovery is async and cannot happen in a constructor.
- Runtime vertical selection is controlled by `AgentForge.Verticals.Hosting`: `VERTICAL_PLUGIN_PATH`, or `VERTICAL_PLUGIN_ROOT` + `VERTICAL_ID`, or fallback to the in-tree travel plugin.

## Key Conventions

### C# style
- **Primary constructors** on all services (no `private readonly` field boilerplate for injected deps).
- **`ConfigureAwait(false)`** on every `await` call inside services and library code.
- **C# 14 features** where appropriate: `field`-backed properties, extension members, `System.Threading.Lock`, collection expressions (`[.. items]`), etc.
- No `#pragma warning disable` — fix the root cause.

### DI lifetime rules
- Stateful classes → `Singleton`
- Per-request / per-message work (e.g., `AgentChatService`) → `Scoped` (resolved per `IServiceScope` inside the queue's `BackgroundService`)
- Never `Transient` for stateful classes

### HTTP clients
- Always registered via `IHttpClientFactory` — never `new HttpClient()`.
- Service-discovery names match Aspire resource names: `"http://waha"`, `"http://mcpserver"`.

### Vertical plugin authoring
- New industries should be implemented as separate vertical libraries under `src/Verticals/AgentForge.Verticals.<Vertical>/`.
- The plugin contract lives in `src/AgentForge.Verticals.Abstractions/VerticalContracts.cs`.
- A vertical should implement `IVerticalPlugin`, expose an `IVerticalMcpRegistrar`, configure any plugin-specific configuration sources/options, create the runtime `IVerticalDescriptor`, and register any WebApi-specific services it needs.
- The current travel plugin entry point is `src/Verticals/AgentForge.Verticals.Travel/TravelVerticalPlugin.cs`.

### MCP tool authoring (current travel vertical example: `src/Verticals/AgentForge.Verticals.Travel/Tools/`)
- Decorate the class with `[McpServerToolType]` and each method with `[McpServerTool]`.
- All parameters and descriptions use `[Description("...")]` attributes.
- Tools are auto-registered via `WithToolsFromAssembly` — no registration needed in `AgentForge.WebApi`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goldytech/whatsapp-ai-travel-agent](https://github.com/goldytech/whatsapp-ai-travel-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
