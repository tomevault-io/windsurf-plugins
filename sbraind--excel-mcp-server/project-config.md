---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Excel MCP (Model Context Protocol) Server built with TypeScript and ExcelJS. It provides 34 comprehensive tools for Excel file manipulation through the MCP protocol, enabling Claude Desktop and other MCP clients to read, write, format, and analyze Excel spreadsheets.

**Key Technologies:**
- TypeScript with strict mode enabled
- ExcelJS for Excel file manipulation
- Zod for runtime schema validation
- MCP SDK (@modelcontextprotocol/sdk)

## Build and Development Commands

### Build
```bash
npm install
npm run build
```
Compiles TypeScript to JavaScript in the `dist/` directory.

### Watch Mode (Development)
```bash
npm run watch
```
Continuously rebuilds on file changes.

### Run Server
```bash
npm start
```
Starts the MCP server (used by Claude Desktop via stdio transport).

### Development Mode
```bash
npm run dev
```
Builds and runs the server in one command.

### Create MCP Bundle (MCPB)
```bash
npm run pack:mcpb
```
Creates a distributable `.mcpb` bundle file for one-click installation.

## Architecture

### Entry Point: `src/index.ts`
- Creates MCP server instance with stdio transport
- Registers all 34 tools with their schemas
- Handles tool invocation routing to appropriate handlers
- Manages user configuration (backups, response formats, allowed directories)
- Provides centralized error handling with Zod validation

### Tool Organization: `src/tools/`
Tools are organized by category into separate modules:
- **read.ts**: Reading operations (workbook info, sheet data, cells, formulas)
- **write.ts**: Writing operations (create workbooks, update cells, add rows)
- **format.ts**: Cell formatting (fonts, colors, borders, alignment, column/row sizing)
- **sheets.ts**: Sheet management (create, delete, rename, duplicate)
- **operations.ts**: Data operations (delete rows/columns, copy ranges)
- **analysis.ts**: Analysis tools (search, filter)
- **charts.ts**: Chart creation (line, bar, column, pie, scatter, area)
- **pivots.ts**: Pivot table generation with aggregations
- **tables.ts**: Excel table formatting with styles
- **validation.ts**: Formula and range validation
- **advanced.ts**: Advanced operations (insert rows/columns, merge/unmerge cells)
- **conditional.ts**: Conditional formatting (cell values, color scales, data bars)
- **helpers.ts**: Shared utilities for workbook loading, saving, path validation

### Schema Definitions: `src/schemas/index.ts`
All tool parameters are validated using Zod schemas. Each tool has a corresponding schema that defines:
- Required and optional parameters
- Type constraints (strings, numbers, booleans, arrays, objects)
- Format validation (cell addresses like "A1", ranges like "A1:D10")
- Nested object schemas for complex inputs (formatting, conditional rules)

### Type Safety: `src/types.ts`
Defines TypeScript interfaces for:
- `CellData`: Cell address, value, formula, type
- `SheetInfo`: Sheet metadata (name, row/column counts, state)
- `WorkbookInfo`: Workbook metadata with sheet list
- `CellFormat`: Font, fill, alignment, border, number format definitions
- `ResponseFormat`: "json" or "markdown" output format
- `ToolResponse`: MCP tool response structure

### Constants: `src/constants.ts`
- `TOOL_ANNOTATIONS`: MCP hints (readOnlyHint, destructiveHint, idempotentHint)
- `ERROR_MESSAGES`: Standardized error strings
- `DEFAULT_OPTIONS`: Default values for response format, backups, display limits

## Important Implementation Details

### Security & Path Validation
The `allowedDirectories` configuration restricts file access. All file operations in `helpers.ts` call `ensureFilePathAllowed()` before reading or writing files. When implementing new tools that access files, always use `loadWorkbook()` and `saveWorkbook()` from helpers.ts - never bypass this validation.

### Backup System
Most write operations accept a `createBackup` parameter. When true, `saveWorkbook()` creates a `.backup` file before modifications. The user can configure `createBackupByDefault` in their MCP settings to enable this globally.

### Response Format Flexibility
Read operations support both JSON (structured data) and Markdown (human-readable tables) response formats. Use `formatDataAsTable()` from helpers.ts to generate markdown tables. Always respect the `responseFormat` parameter passed to tools.

### Cell Address Handling
- Cell addresses are strings like "A1", "B2"
- Ranges are strings like "A1:D10"
- Helper functions `columnLetterToNumber()` and `columnNumberToLetter()` convert between letters and numbers
- `parseRange()` validates and parses range strings
- Always validate cell/range formats in schemas

### Error Handling Pattern
All tools follow this pattern:
1. Validate parameters with Zod schema (automatic in index.ts)
2. Load workbook via `loadWorkbook()` (includes path validation)
3. Get sheet via `getSheet()` (throws if sheet not found)
4. Perform operation with try/catch
5. Save workbook via `saveWorkbook()` if modifying
6. Return formatted response (JSON or Markdown)
7. Throw descriptive errors using constants from ERROR_MESSAGES

### ExcelJS Limitations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbraind/excel-mcp-server](https://github.com/sbraind/excel-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
