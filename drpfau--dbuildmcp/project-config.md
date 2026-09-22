---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**dbuildmcp** is an MCP (Model Context Protocol) server for building Delphi Applications with MSBuild. This project enables AI assistants to invoke MSBuild using the correct Delphi build environment through the standardized MCP protocol. The server is written in Delphi.

## Repository Structure

- `Source/` - Main source code
  - `dbuildmcp.dpr` - Main program
  - `dbuildmcp.Tool.MSBuild.pas` - MSBuild tool implementation
- `sample code/` - Reference code (not used at runtime)
  - `Delphi-MCP-Server-Reference/` - MCP Library reference implementation
  - `DOSCommand-reference/` - DOSCommand Library reference

## Development Workflow

### Manual workflow

1. Open `Source/dbuildmcp.dproj` in RAD Studio
2. Compile and run (F9)
3. Server starts on `http://localhost:3001/mcp`
4. Claude can then interact via curl or MCP client

### Agent workflow

1. Build/Compile via `dbuildmcp`
2. run the process
3. Server starts on `http://localhost:3001/mcp`
4. Claude can then interact via curl or MCP client

## MCP Tools

- `dbuildmcp` to compile/build the project

### msbuild

Build Delphi projects using MSBuild with the correct RAD Studio environment.

**IMPORTANT:** Use forward slashes (`/`) in file paths, not backslashes. The tool converts them internally.

**Parameters:** (JSON keys are lowercase)

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `projectfile` | Yes | - | Full path to .dproj file. **Use forward slashes!** |
| `buildtype` | No | `Make` | `Build` (full rebuild) or `Make` (incremental) |
| `platform` | No | `Win64` | Target platform: `Win32` or `Win64` |
| `config` | No | `Debug` | Build configuration: `Debug` or `Release` |
| `verbosity` | No | `quiet` | MSBuild verbosity: `quiet`, `normal`, or `detailed` |
| `showhintsandwarnings` | No | from settings.ini | Show hints and warnings in output. Defaults to the `DefaultShowHintsAndWarnings` ini value when omitted; an explicit `true`/`false` overrides it. |
| `graphviz` | No | `false` | Generate a GraphViz `.gv` unit-dependency file. Passes `--graphviz` (and `--graphviz-exclude`) to dcc. |
| `graphvizexclude` | No | `System.*;Vcl.*;Winapi.*;Data.*;Soap.*;Xml.*` | Semicolon-separated unit-name wildcards to exclude from the graph. Only used when `graphviz=true`. |
| `graphvizoutdir` | No | next to project | Directory to collect the generated `.gv` file. **Use forward slashes!** Only used when `graphviz=true`. |
| `maxerrors` | No | from settings.ini (`10`) | Max number of error lines shown on a failed build. `1` = just the first error, `0` = all. Overrides `DefaultMaxErrors`. |

### Output size / token limits

To keep tool results small (they are sent back to the model and count against context), the output is bounded after filtering:

- **`maxerrors`** (per-call) / **`DefaultMaxErrors`** (ini, default `10`) — on a failed build, only the first N error lines are shown; the rest are collapsed into a `(+K more error(s) …)` note. MSBuild's own `N Error(s)` summary is always kept, so the true total is still visible. Set `maxerrors=1` for just the first (root-cause) error, or `0` for all.
- **`DefaultMaxHintsWarnings`** (ini, default `30`) — when hints/warnings are shown (`showhintsandwarnings=true`), they are capped at this many, with a `(+K more … suppressed)` note.
- **`MaxOutputLines`** (ini, default `200`) — a hard ceiling on total output lines after all filtering; the head and tail are kept (so the trailing summary survives) with a `… (K lines truncated) …` marker in the middle.

Every limit treats `0` as unlimited. Error detection only ever *caps* detected error lines — an undetected error is treated as ordinary text and kept, so no error is hidden by the cap.

### GraphViz unit-dependency graphs

When `graphviz=true`, the build forwards the dcc switches `--graphviz` and `--graphviz-exclude=<patterns>` to the Delphi compiler through the project's `DCC_AdditionalSwitches` MSBuild property (these are raw compiler switches, not native MSBuild flags). The compiler emits a `<ProjectName>.gv` GraphViz DOT file describing the unit `uses` graph (implementation-section uses are drawn `style=dashed`). Process the `.gv` with [GraphViz](https://graphviz.org/) to render a diagram.

- Semicolons in the exclude list are escaped to `%3B` internally so MSBuild does not mistake them for `/p:` property separators.
- `System`, `SysInit`, and `System.Variants` are always excluded by the compiler regardless of the exclude list.
- dcc only emits the `.gv` when it actually compiles. An incremental `Make` build that recompiles nothing produces no file — use `buildtype=Build` to force it.
- The result header gains a `GraphViz:` line reporting the final `.gv` path (or a note if none was generated).

**Example curl call:**
```bash
curl -X POST http://localhost:3001/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"msbuild","arguments":{"projectfile":"C:/projects/MyApp/MyApp.dproj","platform":"Win64","config":"Release"}}}'
```

**Response Format:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drpfau/dbuildmcp](https://github.com/drpfau/dbuildmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
