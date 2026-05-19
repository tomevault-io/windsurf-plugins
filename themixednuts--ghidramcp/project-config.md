---
trigger: always_on
description: Provides details like program name, path, architecture, and image base.
---

# Creating New Ghidra MCP Tools - Templates & Guidelines

This guide provides templates and key patterns for creating new Ghidra MCP tools, implementing [`IGhidraMcpSpecification`](mdc:src/main/java/com/themixednuts/tools/IGhidraMcpSpecification.java).

**Core Steps:**

1.  **Implement & Annotate:**
    *   Implement [`IGhidraMcpSpecification`](mdc:src/main/java/com/themixednuts/tools/IGhidraMcpSpecification.java).
    *   Add [`@GhidraMcpTool`](mdc:src/main/java/com/themixednuts/annotation/GhidraMcpTool.java) annotation:
        *   `name`: User-facing name in Ghidra Tool Options (e.g., "List Functions").
        *   `description`: Hover description in Ghidra Tool Options.
        *   `category`: Category in Ghidra Tool Options (e.g., [`ToolCategory.FUNCTIONS`](mdc:src/main/java/com/themixednuts/tools/ToolCategory.java)). **Use an existing category or `ToolCategory.UNCATEGORIZED`. Do NOT add new categories to the `ToolCategory` enum.**
        *   `mcpName`: Name for the MCP tool specification (e.g., "list_functions").
        *   `mcpDescription`: Detailed description for the MCP tool specification, guiding its usage by AI agents. See "Writing Effective `mcpDescription`" section below for detailed guidance. Use Java's multi-line string literals (`\"\"\"...\"\"\"`) for readability.
2.  **Register Service:** Add the fully qualified class name to `src/main/resources/META-INF/services/com.themixednuts.tools.IGhidraMcpSpecification`. **This step is mandatory for the tool to be loaded and must be performed automatically when creating a new tool class.**
3.  **Naming Convention:** Follow standard CRUD naming conventions for both the Java class (`Ghidra<Action><Noun>Tool`) and the `mcpName` (`<action>_<noun>`) where possible:
    *   **Create:** Use `Create` for adding new items (e.g., `GhidraCreateBookmarkTool`, `create_bookmark`). Avoid synonyms like `Add`.
    *   **Read:** Use `Get` for single items (e.g., `GhidraGetFunctionByNameTool`), `List` for multiple items (e.g., `GhidraListFunctionsTool`), or `Search` for querying (e.g., `GhidraSearchMemoryTool`).
    *   **Update:** Use `Update` for modifying existing items (e.g., `GhidraUpdateStructMemberTool`). Avoid synonyms like `Edit`, `Set`, or `Change`.
    *   **Delete:** Use `Delete` for removing items (e.g., `GhidraDeleteFunctionTool`). Avoid synonyms like `Remove` or `Clear`.
4.  **Data Models (POJOs):** When a tool needs to return complex data structures (e.g., a list of functions with their details), prefer using or creating Plain Old Java Objects (POJOs) within the [`src/main/java/com/themixednuts/models`](mdc:src/main/java/com/themixednuts/models) directory.
    *   **Reuse:** Check if an existing model (like [`FunctionInfo`](mdc:src/main/java/com/themixednuts/models/FunctionInfo.java), [`SymbolInfo`](mdc:src/main/java/com/themixednuts/models/SymbolInfo.java), `DataInfo`, etc.) already suits the needs.
    *   **Create:** If no suitable model exists, create a new, clearly named POJO in the `models` directory to represent the data being returned. This ensures consistent and well-defined output structures for the client.
    *   Keep POJOs simple, primarily containing fields and a constructor or mapping method to populate them from Ghidra objects.
    *   **Exception:** For grouped operations, use the nested POJOs `IGroupedTool.OperationResult` and `IGroupedTool.GroupedOperationResult` instead of creating separate files.
5.  **`specification` Method:**
    *   **DO NOT override the `specification` method in individual tool classes.**
    *   The default implementation in `IGhidraMcpSpecification` handles retrieving the `@GhidraMcpTool` annotation, generating the schema string from `schema()`, and constructing the final `AsyncToolSpecification`.
    *   This default implementation correctly wraps the `Mono<Object>` from `execute` using `.flatMap(this::createSuccessResult).onErrorResume(this::createErrorResult)` to produce the required `Mono<CallToolResult>`.
5.  **Test Verification:** After implementing one or more tools, run `mvn test` from the project root directory. This verifies that the new tool(s) compile correctly and are properly registered in the service file (via `ServiceRegistrationTest.java`). It's often efficient to create a batch of related tools before running the tests.
6.  **Code Cleanup:** Ensure code is well-formatted and remove any unused imports before finalizing the tool.

---

## Structured Error Handling Guidelines

**CRITICAL:** Always use structured error handling with [`GhidraMcpError.java`](mdc:src/main/java/com/themixednuts/models/GhidraMcpError.java) and [`GhidraMcpException`](mdc:src/main/java/com/themixednuts/exceptions/GhidraMcpException.java) instead of simple `IllegalArgumentException` or other basic exceptions. This provides clients with actionable error information and maintains consistency across all tools.

### Required Imports for Error Handling

```java
import com.themixednuts.exceptions.GhidraMcpException;
import com.themixednuts.models.GhidraMcpError;
import com.themixednuts.utils.GhidraMcpErrorUtils;
import java.util.List;
import java.util.Map;
```

### Constants Usage - NO MAGIC STRINGS

**ALWAYS use constants and annotation values instead of string literals:**

```java

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [themixednuts/GhidraMCP](https://github.com/themixednuts/GhidraMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
