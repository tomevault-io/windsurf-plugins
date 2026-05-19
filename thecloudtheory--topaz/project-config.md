---
trigger: always_on
description: Project-specific knowledge for AI coding agents working in this repository.
---

# CLAUDE.md — Topaz

Project-specific knowledge for AI coding agents working in this repository.

## Quick commands

```bash
dotnet build Topaz.sln
dotnet test Topaz.sln
dotnet run --project Topaz.Host   # starts the emulator host
dotnet run --project Topaz.CLI    # interacts with a running host
```

## Architecture

Topaz emulates Azure services in a single .NET 8 solution. The host process (`Topaz.Host/Host.cs`) composes services and exposes HTTP/AMQP endpoints.

Services live under `Services/Topaz.Service.*`. Each service has:
- Control plane (`*ServiceControlPlane`) — CRUD, resource listing
- Data plane (e.g. `AcrDataPlane`) — runtime/protocol behaviour
- `Endpoints/` — one file per HTTP operation
- `Models/` — resource models (`ArmResource<T>` + `*ResourceProperties`)

## Key paths

| Purpose | Path |
|---|---|
| Host composition | `Topaz.Host/Host.cs` |
| Shared settings / ports | `Topaz.Shared/GlobalSettings.cs` |
| ARM resource base | `Topaz.ResourceManager/ArmResource.cs` |
| Container Registry service | `Services/Topaz.Service.ContainerRegistry/` |
| E2E tests (Azure SDK) | `Topaz.Tests/E2E/` |
| E2E tests (Azure CLI) | `Topaz.Tests.AzureCLI/` |
| Portal tests | `Topaz.Tests.Portal/` |
| API coverage docs | `website/docs/api-coverage/` |
| Backlog | `BACKLOG.md` (root) + `website/docs/roadmap.md` |
| MCP server | `Topaz.MCP/` |

## Where to look first

- [Topaz.Host/Host.cs](Topaz.Host/Host.cs) — host composition, service list, endpoint wiring.
- [Topaz.MCP/](Topaz.MCP/) — MCP server exposing Topaz management tools to AI assistants (GitHub Copilot, Claude, etc.).
- [Topaz.CLI/Program.cs](Topaz.CLI/Program.cs) and [Topaz.CLI/Commands/StartCommand.cs](Topaz.CLI/Commands/StartCommand.cs) — how commands bootstrap the host.
- [Topaz.ResourceManager/ArmResource.cs](Topaz.ResourceManager/ArmResource.cs) — resource model base and ID parsing.
- [Topaz.Shared/GlobalSettings.cs](Topaz.Shared/GlobalSettings.cs) — JSON and default ports.
- Example service: [Services/Topaz.Service.KeyVault/](Services/Topaz.Service.KeyVault/) and its endpoints/control plane.

## Build, run and env notes

- Ports and emulator directory: defaults are in `Topaz.Shared/GlobalSettings.cs` (e.g., `MainEmulatorDirectory = .topaz`). The host will create `.topaz` and `global-dns.json` on first run.
- TLS and certificates: host expects PEM files `topaz.crt`/`topaz.key` or accepts `--certificate-file`/`--certificate-key` CLI options (`Topaz.CLI/Commands/StartCommand.cs`).
- Containerization: `Topaz.CLI/Dockerfile` and `Topaz.MCP/Dockerfile` exist; CI scripts and `publish/` contain packaging helpers. See `scripts/` and `install/` for platform-specific helpers.

## Mandatory steps

### Every endpoint change

1. **One file per HTTP operation** in `Endpoints/` — never combine multiple operations in one `IEndpointDefinition`.
2. Register in `*Service.cs` → `Endpoints` property.
3. Update `website/docs/api-coverage/<service>.md` — flip ❌ → ✅ for implemented operations.
4. Add tests in **both** suites (see Tests section).
5. For ARM-manageable nested objects (for example network rule sets), model them as `ArmSubresource<T>` and persist them via `CreateOrUpdateSubresource` / `GetSubresourceAs` instead of building ad-hoc response DTOs in endpoints.

### Every new service (control plane)

1. Implement `Deploy()` — never `throw new NotImplementedException()`. Follow the KeyVault pattern: cast `GenericResource` → `resource.As<TResource, TProperties>()`, map fields, delegate to `CreateOrUpdate`.
2. Register in `TemplateDeploymentOrchestrator.RouteDeployment()` with `case "Microsoft.X/y":`.
3. Add `<ProjectReference>` in `Topaz.Service.ResourceManager.csproj`.
4. Add new tool in `Topaz.MCP` project. Follow the conventions found in `Tools` folder.
5. Add Terraform and PowerShell tests in `Topaz.Tests.Terraform` and `Topaz.Tests.AzurePowerShell`

### Backlog / Roadmap

- New work → add `<!-- TODO: ... -->` to `BACKLOG.md` **and** a row to `website/src/pages/roadmap.md`.
- Completed work → remove the task from backlog so it will be removed from GitHub issues.
- Badges: `<span class="badge--stable">Stable</span>` or `<span class="badge--preview">Preview</span>` — use CSS classes, never inline styles.
- Badge CSS lives in `website/src/css/custom.css` (`.badge--stable` and `.badge--preview`). Do not add inline styles; always use these classes.
- For simple tasks (CRUD, scafollding, boilerplate) always add `good first issue` label.

## API Coverage docs

- The `website/docs/api-coverage/` directory contains one Markdown file per service. Each file tracks which Azure REST API operations are implemented in Topaz, mapped to the official Microsoft REST API reference.
- **Always consult** the relevant `api-coverage/<service>.md` file before adding or removing endpoints for a service so you know what is already tracked.
- **Always update** the relevant `api-coverage/<service>.md` file after adding or removing endpoint implementations: flip ❌ → ✅ (or vice-versa) for the affected operations. If the service page is still a stub, fill in the full operation table (use the Azure REST API reference link in the file header as a guide).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheCloudTheory/Topaz](https://github.com/TheCloudTheory/Topaz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
