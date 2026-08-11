---
trigger: always_on
description: > Instructions for AI coding agents implementing `aict`.
---

# AGENTS.md — AI-Coreutils Development Guide

> Instructions for AI coding agents implementing `aict`.
> Read this before writing any code.

---

## Project Overview

**What**: Single Go binary (`aict`) reimplementing 33 Unix CLI tools that AI coding agents actually call, outputting structured XML instead of plaintext.

**Why**: Human-readable `ls`, `grep`, `cat` output forces agents to parse column positions and chain follow-up calls (`file`, `stat`) for metadata. aict answers in one call with zero parsing ambiguity — it spends more output tokens per call, but fewer agent turns per task (measured: see `benchmarks/TOKENS.md`).

**Constraints**:
- Go only; all tools and internal packages are stdlib-only. The single permitted external dependency is the official MCP Go SDK, used exclusively in `cmd/mcp`
- Single binary, subcommand model: `aict ls src/`
- Three output modes: XML (default for AI), JSON, plain text (compatibility)
- All paths absolute in output
- All timestamps Unix epoch integers
- All sizes in bytes
- Structured errors, never stderr
- Empty results are valid XML

---

## How to Work on This Project

### 1. Read Before Writing

Before implementing any tool:
1. Read `ROADMAP.md` for the phase and task you're working on
2. Read `ROADMAP.md` for the full XML schema spec of that tool
3. Read existing tools in `tools/` to match patterns
4. Read `internal/` packages for shared utilities

### 2. Implementation Order

Follow the roadmap phases strictly. Do NOT skip ahead:

```
Phase 0: Foundation → ls
Phase 1: cat → grep → find → stat → wc → diff
Phase 2: file → head/tail → du/df → path utils → pwd → sort/uniq → cut/tr → env → system → ps → checksums
Phase 3: Performance, cross-platform, docs, CI
```

Each tool must pass all tests before moving to the next.

### 3. File Structure

```
aict/
├── main.go                       # Subcommand dispatch, global flags
├── cmd/
│   ├── mcp/                      # MCP server (aict mcp subcommand)
│   ├── bench/                    # Speed benchmark harness vs GNU tools
│   ├── tokenbench/               # Token-cost benchmark harness
│   └── gendocs/                  # Docs generation
├── internal/
│   ├── xml/encoder.go            # Shared XML/JSON/plain output
│   ├── detect/language.go        # Extension → language map
│   ├── detect/mime.go            # Magic bytes MIME detection
│   ├── path/resolve.go           # Absolute path resolution
│   ├── format/size.go            # Bytes → human-readable
│   ├── meta/timestamp.go         # Unix time + ago_s helpers
│   ├── tool/                     # Tool registry + schema generation
│   └── version/                  # Build version (ldflags-injected)
└── tools/
    ├── ls/ls.go
    ├── cat/cat.go
    ├── grep/grep.go
    ├── find/find.go
    ├── stat/stat.go
    ├── wc/wc.go
    ├── diff/diff.go
    └── ... (one package per tool)
```

### 4. Tool Implementation Pattern

Every tool follows this structure:

```go
package toolname

import (
    "encoding/xml"
    "flag"
    "fmt"
    "os"
    "time"

    "github.com/synseqack/aict/internal/detect"
    "github.com/synseqack/aict/internal/format"
    "github.com/synseqack/aict/internal/meta"
    "github.com/synseqack/aict/internal/path"
    "github.com/synseqack/aict/internal/xmlout"
)

type Config struct {
    // Tool-specific flags
    XML     bool
    JSON    bool
    Plain   bool
    // ... tool flags
}

func Run(args []string) error {
    cfg := parseFlags(args)
    // ... tool logic
    return output(cfg, result)
}

func parseFlags(args []string) Config {
    // Use stdlib flag package
    // Return Config with all flags set
}

func output(cfg Config, result interface{}) error {
    if cfg.JSON {
        return xmlout.WriteJSON(os.Stdout, result)
    }
    if cfg.Plain {
        return writePlain(os.Stdout, result)
    }
    return xmlout.WriteXML(os.Stdout, result, cfg.XML)
}
```

### 5. XML Output Rules

**MANDATORY** — every tool output must follow these rules:

- Root element named after tool: `<ls>`, `<grep>`, `<find>`, etc.
- Root element always has `timestamp` attribute (Unix epoch integer)
- Root element always has all flags/options as attributes
- Root element always has summary counts where applicable
- Every path has both `path` (as given) and `absolute` (resolved) attributes
- All times are Unix epoch integers with companion `_ago_s` attributes
- All sizes are bytes with companion `_human` attributes
- Booleans are `true`/`false` strings, never `1`/`0`
- Errors are `<error code="" msg=""/>` elements, never stderr
- Empty results are valid: `<grep matched_files="0" total_matches="0"/>`
- Binary files never output as CDATA — omit content or use base64
- Language values: lowercase canonical (`go`, `python`, `typescript`, etc.)

### 6. Testing Requirements

Every tool MUST have:

```go
// toolname_test.go
package toolname

import (
    "bytes"
    "encoding/xml"
    "os"
    "path/filepath"
    "testing"
)

func TestToolName_Basic(t *testing.T) {
    // Create temp dir/files
    dir := t.TempDir()
    // ... setup

    // Run tool
    var buf bytes.Buffer
    err := RunWithOutput(dir, &buf, Config{XML: true})
    if err != nil {
        t.Fatal(err)
    }

    // Validate XML is well-formed
    var result ResultType

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [synseqack/aict](https://github.com/synseqack/aict) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
