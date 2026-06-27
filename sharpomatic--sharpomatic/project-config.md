---
trigger: always_on
description: ﻿# Repository Project
---

﻿# Repository Project

SharpOMatic is an open-source project on GitHub that allows a user to build and execute workflows with an emphasis on AI-related tasks. It has deep integration with .NET by allowing users to add C# code snippets and call directly into backend code.

The frontend is an Angular and TypeScript browser-based application.
The backend is .NET/C#-based and consists of Engine, Editor, AGUI, DemoServer, and provider packages for SQLite and SQL Server.
The 'docs' directory defines the static website and uses Docusaurus.

## Project Structure & Module Organization
- `src/SharpOMatic.sln` is the .NET solution used by Visual Studio and CLI builds to load the backend projects (Engine, Editor, AGUI, DemoServer, SQLite/SQL Server providers, tests) plus solution items like `DEV.md` and `TODO.md`. The Angular frontend is not included as a project in the solution; instead, the Editor project runs the frontend build and embeds the output during its own build.

- `src/SharpOMatic.FrontEnd/` is the Angular + TypeScript SPA for the workflow designer/editor UI, including pages, components, and services that call the backend APIs and SignalR endpoints. Its production build output (`dist/SharpOMatic-Editor/browser`) is consumed by the Editor project and embedded into the .NET package, so UI changes here affect the hosted editor experience.

- `src/SharpOMatic.Editor/` is a .NET 10 class library that hosts the editor UI and supporting endpoints; it embeds the built Angular assets as resources and exposes ASP.NET controllers and SignalR hubs for the editor and transfer flows. It depends on the Engine project and provides extension methods to plug the editor into an ASP.NET Core app, so changes here affect server-side hosting, routing, and API contracts for the UI.

- `src/SharpOMatic.Engine/` is the .NET 10 core workflow engine that defines the runtime model (nodes, contexts, metadata), persistence (EF Core models/migrations), and services used to execute workflows and manage assets, runs, and repository state. It is the main backend library consumed by both the Editor package and the Server host, so changes here typically impact execution behavior and API DTOs.

- `src/SharpOMatic.AGUI/` is a .NET 10 library that exposes AG-UI compatible endpoints and services for stateful agent/chat integrations. It depends on the Engine project and shares workflow run, conversation, stream event, and history-building behavior with the core runtime, so changes here should be kept aligned with AG-UI docs and tests.

- `src/SharpOMatic.Engine.Sqlite/` and `src/SharpOMatic.Engine.SqlServer/` are provider packages for EF Core database configuration and migrations. Schema-affecting repository/entity changes usually need matching migrations and model snapshots in both provider projects.

- `src/SharpOMatic.DemoServer/` is a .NET 10 ASP.NET Core host application that wires Engine and Editor together for local running, testing, and sample configuration (routing, database setup, asset storage). It serves as the integration harness where you can validate end-to-end editor + engine behavior before packaging or deploying elsewhere. This is used during development to test changes, but end users are expected to create their own server or integrate the Editor and Engine into their own existing project.

- `src/SharpOMatic.Tests/` contains xUnit tests for engine workflow behavior, editor/AG-UI behavior, and service-level behavior such as transfer import/export. It is the primary regression suite for backend changes.

### Project Structure Engine

- `SharpOMatic.Engine/bin`
    Build outputs produced by local or CI builds (compiled assemblies, temporary artifacts). These files are generated and should not be edited; delete the folder if you need a clean rebuild.

- `SharpOMatic.Engine/Contexts`
    Runtime context data structures (ContextObject/ContextList) plus RunContext/ThreadContext that carry state through workflow execution. Values must be JSON-serializable to persist runs, and custom converters are registered through the JSON conversion services.

- `SharpOMatic.Engine/DTO`
    Request/response payload models used by editor and transfer APIs (typically Request/Result naming). These DTOs are used for API-specific contracts that are not represented by core entities or metadata.

- `SharpOMatic.Engine/Entities`
    Domain entities for workflows, nodes, and supporting data, including versioned configuration that is persisted to the repository. Editing these models usually affects serialization, migrations, and runtime behavior.

- `SharpOMatic.Engine/Enumerations`
    Shared enums used across multiple engine areas (runs, nodes, metadata, services), keeping cross-cutting state flags centralized.

- `SharpOMatic.Engine/Exceptions`
    Engine-specific exception types that provide domain context for validation, parsing, and execution failures; these surface to API responses and logs.

- `SharpOMatic.Engine/FastSerializer`
    Custom JSON tokenizer/deserializer used for fast, location-aware parsing of context data and other engine JSON, which is critical for precise error reporting in user-authored inputs.

- `SharpOMatic.Engine/Helpers`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sharpomatic/SharpOMatic](https://github.com/sharpomatic/SharpOMatic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
