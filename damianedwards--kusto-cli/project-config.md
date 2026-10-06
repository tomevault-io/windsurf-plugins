---
trigger: always_on
description: Use these from the repository root:
---

# Copilot instructions for `kusto`

## Build and test commands

Use these from the repository root:

```powershell
# Build all projects
dotnet build kusto.slnx

# Run the full test suite
dotnet test kusto.slnx

# Run a single test (xUnit filter by fully-qualified name)
dotnet test .\tests\Kusto.Cli.Tests\Kusto.Cli.Tests.csproj --filter "FullyQualifiedName~ParserTests.Parse_DatabaseList_AcceptsFilterAndTake"
```

There is no separate lint command configured in this repository.

## High-level architecture

### 1) CLI composition and command wiring
- `src/Kusto.Cli/Program.cs` is the minimal entry point: it builds the root command via `CommandFactory.CreateRootCommand()` and invokes parsing/execution.
- `src/Kusto.Cli/CommandFactory.cs` defines the entire command surface (`cluster`, `database`, `table`, `query`) and binds each action through `CliRunner.RunAsync(...)`.

### 2) Runtime composition and execution flow
- `src/Kusto.Cli/CliRunner.cs` is the command execution wrapper: it parses common options (`--format`, `--log-level`), constructs runtime dependencies, formats output, and centralizes error handling.
- `CliRunner.CreateRuntime(...)` wires `FileConfigStore`, `KustoConnectionResolver`, `AzureTokenProvider`, `KustoHttpService`, `OfflineTableDataStore`, `TableSchemaProvider`, `TableOfflineDataManager`, the confirmation prompt, and `OutputFormatter` into a `CliRuntime`.
- `src/Kusto.Cli/Contracts.cs` defines the core interfaces (`IConfigStore`, `IKustoService`, `IKustoConnectionResolver`, `ITableSchemaProvider`, `ITableOfflineDataManager`, `IConfirmationPrompt`, `IOutputFormatter`) and the `CliRuntime` container.

### 3) Data/config and connection resolution
- `src/Kusto.Cli/FileConfigStore.cs` persists config to `%USERPROFILE%\.kusto\config.json` (or `KUSTO_CONFIG_PATH` override).
- `src/Kusto.Cli/ClusterUtilities.cs` normalizes cluster URLs and config values; defaults and lookups rely on normalized URLs.
- `src/Kusto.Cli/KustoConnectionResolver.cs` resolves effective cluster/database from explicit options or configured defaults.
- `src/Kusto.Cli/OfflineTableDataStore.cs` persists offline table data per normalized cluster/database in the schema-cache directory; each entry can hold cached schema JSON plus per-table notes.
- `src/Kusto.Cli/TableSchemaProvider.cs` reads/refreshes cached schema data for `table show`, while `src/Kusto.Cli/TableOfflineDataManager.cs` owns notes and offline-data import/export/purge/clear flows.

### 4) Kusto transport layer
- `src/Kusto.Cli/KustoHttpService.cs` calls Kusto REST endpoints:
  - management commands: `/v1/rest/mgmt`
  - queries: `/v2/rest/query`
- Response parsing handles both object payloads (`Tables`) and frame-array payloads (`FrameType == DataTable`), then selects `PrimaryResult` when available.
- Request payloads and output JSON use source-generated serializers (`src/Kusto.Cli/KustoJsonSerializerContext.cs`) for AOT-safe serialization.

### 5) Output and logging pipeline
- `src/Kusto.Cli/OutputFormatter.cs` supports `human`, `json`, and `markdown` output; human output now uses Hex1b-backed rendering plus text-table formatting for static terminal output.
- Query result tables are flagged with `CliOutput.IsQueryResultTable` for query-specific table styling/alignment.
- `src/Kusto.Cli/Logging.cs` configures always-on file logging (`%TEMP%\kusto\kusto.log`) and optional stderr logging when `--log-level` is explicitly set.
- `src/Kusto.Cli/KustoConsoleFormatter.cs` and `src/Kusto.Cli/ConsoleStyling.cs` apply console styling (light-gray logs, red errors, ANSI-aware behavior).

### 6) Chart rendering pipeline
- `src/Kusto.Cli/KustoVisualizationExtractor.cs` parses Kusto `render` annotations from query responses into `QueryVisualization`.
- `src/Kusto.Cli/KustoChartCompatibilityAnalyzer.cs` is the compatibility gate for chart rendering. It maps Kusto render kinds to `QueryChartKind`, validates columns/layouts, and produces either a `HumanChart`, a `MarkdownChart`, or explicit reasons why rendering is not supported.
- `src/Kusto.Cli/Models.cs` defines the chart model surface: `QueryChartKind` (`Column`, `Bar`, `Line`, `Pie`) and `QueryChartLayout` (`Simple`, `Grouped`, `Stacked`, `Stacked100`).
- `src/Kusto.Cli/Hex1bChartRenderer.cs` is the terminal renderer used for human output. It renders `Column`, `Bar`, `Line`, and `Pie`.
- `src/Kusto.Cli/MermaidChartRenderer.cs` is the markdown renderer. It renders Mermaid `xychart` output for cartesian charts and Mermaid `pie` output for pie charts.
- In `src/Kusto.Cli/CommandFactory.cs`, `query --chart` chooses Hex1b for `human` output and Mermaid for `markdown`/`md`; JSON output does not support chart rendering.

## Key repository conventions

1. **User-facing errors must be actionable and implementation-agnostic**
   - Use `UserFacingException` for expected CLI/user errors.
   - Let `ErrorMapper`/`CliRunner` surface concise messages; do not expose stack traces or raw HTTP status-code wording to users.

2. **Authentication is routed per cluster**
   - Token acquisition is routed by `RoutingTokenProvider` based on each `ResolvedCluster`'s authentication mode.
   - Clusters with no `authentication` (or `mode: default`) use `DefaultAzureCredential` (`AzureTokenProvider`); for those, credential/access failures guide the user to run `az login`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DamianEdwards/kusto-cli](https://github.com/DamianEdwards/kusto-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
