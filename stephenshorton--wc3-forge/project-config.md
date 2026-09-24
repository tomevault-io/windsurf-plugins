---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Build the desktop app (this is the canonical build — `go build` alone produces a runnable binary but skips the embedded frontend and the postbuild library copy, leaving you with something that loads neither UI nor CASC):

```
wails build                               # production build
wails dev                                 # dev build with Vite HMR
```

Output paths differ per OS:
- Windows: `build/bin/wc3-forge.exe` (with `wc3-forge-dev.exe` for dev)
- macOS: `build/bin/wc3-forge.app/Contents/MacOS/wc3-forge`

**macOS only — first-time setup.** Run `./scripts/build-casclib-macos.sh` once after cloning. It downloads CascLib, builds `libcasc.dylib`, and drops it in `scripts/casclib/` (where the `darwin/*` postBuildHook expects it). The dylib is `.gitignore`d.

Run the existing binary directly (useful when you don't need a rebuild) — replace `<bin>` with the per-OS path above:

```
<bin>                                     # GUI + bridge
<bin> --open <path>                       # auto-load a map folder or .w3x on startup
<bin> --headless                          # MCP bridge only, no window
<bin> --no-bridge                         # GUI only, no MCP bridge
<bin> --reforged                          # start in HD asset mode (default is SD/Classic)
<bin> --camera x,y,z,distance             # pin startup camera for verification
```

Tests + checks (mirror the CI flow in `.github/workflows/ci.yml`):

```
go vet ./...
go test ./...
go test ./internal/forge/...             # single package
go test ./internal/forge -run TestSessionSaveRoundTrip   # single test
cd frontend && npm ci                    # first-time + after package.json changes
cd frontend && npm run check             # svelte-check (typecheck)
cd frontend && npm run build             # vite build (also runs as part of `wails build`)
```

The `cmd/dumpw3e` parser-replay tool exists for cross-checking renderer divergences against HiveWE — `go run ./cmd/dumpw3e <map-path>`.

## Process etiquette for parallel agents

Multiple wc3-forge instances coexist by design — the MCP bridge auto-picks an unused port and each process writes its own lockfile at `~/.wc3-forge/mcp/<pid>.lock`. **Do not run process-wide kills like `Stop-Process wc3-forge` (Windows) or `pkill wc3-forge` (macOS)** — that kills sibling agents' verification windows and the user's interactive session.

When launching your own instance for testing, capture the PID and kill only that PID on cleanup:

```powershell
# Windows
$proc = Start-Process -FilePath 'build\bin\wc3-forge.exe' -PassThru
# ... do work ...
Stop-Process -Id $proc.Id -Force -ErrorAction SilentlyContinue
```

```bash
# macOS
build/bin/wc3-forge.app/Contents/MacOS/wc3-forge &
PID=$!
# ... do work ...
kill -TERM "$PID" 2>/dev/null
```

`wails dev` parents the dev binary under a `wails` process — when cleaning up after a dev build, filter the process-name prefix (`wc3-forge`) so the child doesn't outlive the parent and hold the CASC library locked.

If you'd otherwise launch only to drive an MCP call, read an existing PID's lockfile (`{port, token}`) and connect over JSON-RPC instead.

## Architecture

Single Wails v2 executable. Go owns map parsing/I/O and the MCP bridge; TypeScript + Svelte owns the 3D viewport and panels. Two control surfaces (the GUI and external MCP clients) converge on the same `forge.Session` singleton, so a JSON-RPC `units.move` and a viewport drag end up in the same mutator and emit the same `entity-changed` / `dirty-changed` events.

```
                ┌────────────── Go process ──────────────┐
   MCP client ──┤ bridge (NDJSON / JSON-RPC over TCP) ───┤
                │                                        │
   GUI (Wails)──┤ App (bindings) ──┐                     │
                │                  ├─► forge.Session ◄───┤
                │ assetHandler ────┘   (map state +      │
                │ /asset/<path>        history + emits)  │
                │   1) map archive                       │
                │   2) CASC mount  ◄── WC3FORGE_WC3_PATH │
                └────────────────────────────────────────┘
```

### Go side (`internal/`)

- `internal/forge` — the editor core. `Session` is the singleton; `RegisterAll(b)` in `handlers.go` is the **single registration point** for every MCP method. Adding a new MCP tool means: write `handle<Foo>`, add `reg("foo.bar", handleFoo)` to `RegisterAll`, and add the matching mutator to `session.go` so the command flows through history. All session mutations go through `recordCommand` so undo/redo + `entity-changed` events stay coherent across both control surfaces. **Also expose the tool to clients:** add it in `mcp/src/tools.ts` (the catalog source of truth), then run `cd mcp && npm run gen:tools` to regenerate `internal/mcpserver/tools.json` and commit it — that JSON is what the in-binary `--mcp` server serves via tools/list.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StephenSHorton/wc3-forge](https://github.com/StephenSHorton/wc3-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
