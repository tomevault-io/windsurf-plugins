---
trigger: always_on
description: > Model Context Protocol server for Unreal Engine 5.7. Single Go binary.
---

# CLAUDE.md — mcp-unreal

> Model Context Protocol server for Unreal Engine 5.7. Single Go binary.
> Repository: https://github.com/remiphilippe/mcp-unreal
> License: Apache-2.0

---

## Project Overview

This is an MCP (Model Context Protocol) server written in Go that gives AI coding agents (Claude Code, Cursor, etc.) complete autonomous control over an Unreal Engine 5.7 project — builds, tests, editor manipulation, Blueprint editing, procedural mesh generation, and documentation lookup.

**Read `IMPLEMENTATION.md` before writing any code.** It contains the full architecture, tool inventory, communication paths, and design decisions.

### Key Facts

- **Language**: Go (primary), C++ (UE editor plugin only)
- **MCP SDK**: `github.com/modelcontextprotocol/go-sdk/mcp` (official, co-maintained with Google)
- **Transport**: stdio (JSON-RPC 2.0) — Claude Code launches the binary as a subprocess
- **Doc search**: `github.com/blevesearch/bleve/v2` (compiled into the binary, no external service)
- **Target UE version**: 5.7 (macOS primary, Windows and Linux secondary)
- **Go version**: 1.25+

---

## Repository Structure

```
mcp-unreal/
├── cmd/
│   └── mcp-unreal/
│       └── main.go                     # Entry point, CLI flags, tool registration
├── internal/
│   ├── config/
│   │   └── config.go                   # Environment config, project detection, path resolution
│   ├── headless/
│   │   ├── build.go                    # build_project, cook_project, generate_project_files
│   │   ├── build_test.go
│   │   ├── test.go                     # run_tests, run_visual_tests, list_tests
│   │   ├── test_test.go
│   │   └── log.go                      # get_test_log
│   ├── editor/
│   │   ├── client.go                   # HTTP client for RC API + plugin
│   │   ├── client_test.go
│   │   ├── actors.go                   # Actor CRUD tools
│   │   ├── actors_test.go
│   │   ├── properties.go              # set_property, get_property, call_function
│   │   ├── blueprints.go             # blueprint_query, blueprint_modify
│   │   ├── anim_blueprints.go        # anim_blueprint_query, anim_blueprint_modify
│   │   ├── assets.go                  # search_assets, get_asset_info
│   │   ├── materials.go              # material_ops
│   │   ├── characters.go             # character_config
│   │   ├── input.go                   # input_ops
│   │   ├── mesh.go                    # procedural_mesh, realtime_mesh
│   │   ├── levels.go                  # level_ops
│   │   └── utilities.go              # console cmd, output log, viewport, scripts
│   ├── docs/
│   │   ├── index.go                   # Bleve index open/create/query
│   │   ├── index_test.go
│   │   ├── lookup.go                  # lookup_docs, lookup_class tool handlers
│   │   ├── ingest.go                  # Markdown → bleve doc entries
│   │   └── class_parser.go           # Parse UE class reference markdown
│   └── status/
│       └── status.go                  # status tool, connectivity checks
├── docs/                               # Documentation source files for the index
│   ├── ue5.7/
│   ├── realtimemesh/
│   └── README.md                      # How to add/update doc entries
├── plugin/                             # UE 5.7 C++ editor plugin
│   ├── MCPUnreal.uplugin
│   ├── Source/
│   │   └── MCPUnreal/
│   │       ├── MCPUnreal.Build.cs
│   │       └── ...
│   └── README.md                      # Plugin build/install instructions
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                     # Go lint + test + build
│   │   └── release.yml                # GoReleaser on tag push
│   └── ISSUE_TEMPLATE/
│       ├── bug_report.md
│       └── feature_request.md
├── IMPLEMENTATION.md                   # Full architecture document
├── CLAUDE.md                          # This file
├── CONTRIBUTING.md
├── LICENSE                            # Apache-2.0
├── README.md                          # User-facing: install, configure, use
├── SECURITY.md
├── go.mod
├── go.sum
├── Makefile
└── .goreleaser.yml
```

---

## Code Standards

### Go

- **Format**: `gofmt` / `goimports` on all files. No exceptions.
- **Lint**: `golangci-lint` with the config below. CI must pass.
- **Naming**: Follow Effective Go. Exported types use `PascalCase`. Unexported use `camelCase`. Acronyms keep case (`HTTP`, `URL`, `API`, not `Http`).
- **Errors**: Always wrap with context using `fmt.Errorf("doing X: %w", err)`. Never discard errors silently. Use sentinel errors in `internal/` packages only when callers need to match on them.
- **Logging**: Use `log/slog` (structured logging). Never `fmt.Println` for operational output. The MCP transport uses stdout — any stray prints corrupt the JSON-RPC stream.
- **Context**: All tool handlers receive `context.Context`. Propagate it to subprocess calls and HTTP requests. Respect cancellation.
- **Testing**: Every package under `internal/` must have `_test.go` files. Use table-driven tests. Use `testify/assert` only if the team agrees — standard library `testing` is fine.
- **Dependencies**: Minimize. Currently allowed:
  - `github.com/modelcontextprotocol/go-sdk` — MCP SDK
  - `github.com/blevesearch/bleve/v2` — Full-text search

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remiphilippe/mcp-unreal](https://github.com/remiphilippe/mcp-unreal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
