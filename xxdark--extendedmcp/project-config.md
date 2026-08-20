---
trigger: always_on
description: IntelliJ plugin that extends the built-in JetBrains MCP server with additional tools.
---

# ij-mcp — IntelliJ MCP Server Extension Plugin

## What This Is
IntelliJ plugin that extends the built-in JetBrains MCP server with additional tools.
Registers `McpToolset` implementations via the `com.intellij.mcpServer.mcpToolset` extension point — the built-in server auto-discovers them.

## Project Structure
```
src/main/kotlin/dev/xdark/ijmcp/
  util/PsiUtil.kt              — resolveFile, formatLocation, findSymbolByName, Java/Kotlin class lookup
  util/FilePatternResolver.kt  — Glob/literal file pattern resolution, PSI batch resolution
  settings/Setting.kt          — Setting<T> delegate, setting() factory, SettingsHolder base
  settings/ConfigurableToolset.kt — Opt-in interface: settingsTitle + settingsHolder
  settings/ToolSettingsService.kt — App-level persistence (ijMcpToolSettings.xml) + SettingCodec
  settings/SettingsUiGenerator.kt — editorFor(): reflective Setting → Swing control by KType
  settings/McpToolSettingsConfigurable.kt — Settings > Tools > MCP Toolset Settings UI

  tools/nav/                   — read-only lookup and navigation
    FindUsagesToolset.kt         — find_usages
    FindClassToolset.kt          — find_class
    FindInFilesToolset.kt        — find_in_files
    ImplementationsToolset.kt    — get_implementations
    GoToDeclarationToolset.kt    — go_to_declaration
    TypeInfoToolset.kt           — get_type_info
    FileStructureToolset.kt      — get_file_outline
    ListPackageClassesToolset.kt — list_package_classes
    LibrarySourcesToolset.kt     — check_library_sources

  tools/refactor/              — PSI-level code modification
    MoveClassToolset.kt          — move_class
    RenameMemberToolset.kt       — rename_member
    SafeDeleteToolset.kt         — safe_delete
    ChangeSignatureToolset.kt    — change_method_signature
    InlineToolset.kt             — inline_method
    ExtractMethodToolset.kt      — extract_method
    AddMethodToolset.kt          — add_method
    AddFieldToolset.kt           — add_field
    AddMemberSupport.kt          — AddMemberResult + findJavaClass/findKotlinClass, shared by the two above
    AddImportToolset.kt          — add_imports
    ReplaceMethodBodyToolset.kt  — replace_method_body
    ExtendedRefactoringToolset.kt — optimize_imports
    ShortenReferencesToolset.kt  — shorten_references
    ReformatToolset.kt           — reformat_files
    QuickFixToolset.kt           — apply_quick_fix

  tools/files/                 — file content read/write
    BatchFileTextToolset.kt      — read_files
    BatchProblemsToolset.kt      — get_problems_in_files
    CreateFileToolset.kt         — create_file
    ReplaceLinesToolset.kt       — replace_lines
    BatchReplaceLinesToolset.kt  — batch_replace_lines
    BatchReplaceTextToolset.kt   — batch_replace_text_in_file

  tools/docs/                  — documentation tools
    AddDocumentationToolset.kt   — add_documentation
    GetDocumentationToolset.kt   — get_documentation
    MissingDocumentationToolset.kt — missing_documentation

  tools/build/                 — Gradle / dependency operations
    GradleToolset.kt             — run_gradle_task
    GradleSyncToolset.kt         — gradle_sync
    DownloadSourcesToolset.kt    — download_sources
    GradleSettings.kt            — Gradle settings bean (noisyPatterns) — first ConfigurableToolset consumer

  infra/                       — tool filtering, arg normalization, metrics, menu actions
    ToolFilterToolset.kt         — list_tools_filter
    ToolListService.kt           — App service: enumerates registered tools + built-in tool names (classloader check)
    ArgNormalizingFilterProvider.kt — McpToolFilterProvider: disables filtered tools + wraps tools for arg normalization
    ArgNormalizingMcpTool.kt     — McpTool wrapper that runs ArgumentNormalizers before delegating to the real tool
    ArgumentNormalizer.kt        — fun interface: normalize(args, propertiesSchema)
    UnknownParameterNormalizer.kt — Fails fast (mcpFail) on unknown parameter names
    StringEncodedJsonNormalizer.kt — Parses a JSON-string arg into a real array/object when the schema expects one; mcpFails on a string that opens like a container but is malformed
    SingleElementArrayNormalizer.kt — Wraps a lone value into a single-element array when the schema expects an array and the value validates as one element
    JsonSchemaValidator.kt       — Minimal recursive validator for the MCP JSON-Schema subset (type/items/properties/required/enum); used by SingleElementArrayNormalizer
    ToolFilterState.kt           — Persists disabled tool names (mcpToolFilter.xml)
    ToolFilterAction.kt          — Tools menu UI for toggling tools
    McpMetricsService.kt         — Tool call counting + persistence (also declares McpMetricsListener)
    McpMetricsAction.kt          — Tools menu UI for viewing metrics
src/main/resources/META-INF/plugin.xml  — Toolset registrations
```

## Build
- **Build command**: Use run config `ij-mcp [buildPlugin]` (or `./gradlew buildPlugin`)
- After building, user reloads the plugin in the IDE, then reconnects MCP (`/mcp` in Claude Code)
- `platformVersion` in `gradle.properties` MUST match the running IDE version — mismatch causes `NoClassDefFoundError` at runtime

## Writing New Tools

### Skeleton
```kotlin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xxDark/extendedmcp](https://github.com/xxDark/extendedmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
