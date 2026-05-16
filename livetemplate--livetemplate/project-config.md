---
trigger: always_on
description: LiveTemplate is a high-performance Go library for building reactive web applications. It provides an API similar to `html/template` but with the additional capability of generating minimal, tree-based updates that can be efficiently transmitted to clients.
---

# LiveTemplate - Development Guidelines

## Project Overview

LiveTemplate is a high-performance Go library for building reactive web applications. It provides an API similar to `html/template` but with the additional capability of generating minimal, tree-based updates that can be efficiently transmitted to clients.

**Related repositories:**
- **CLI Tool (lvt)**: Code generator and development server — maintained at `github.com/livetemplate/lvt`
- **TypeScript Client**: Browser-side update handler — maintained at `github.com/livetemplate/client`

## Controller+State Pattern

Controllers hold dependencies (singleton, never cloned). State holds pure data (cloned per session). Action methods: `func (c *Controller) Action(state State, ctx *Context) (State, error)`. Mount runs on every HTTP request and WebSocket connect.

**Key caveats:**
- **Mount() guard pattern:** Mount runs on POST too, so guard side effects with the connect-kind helpers: `if ctx.IsInitialMount() { trackPageView() }` (initial HTTP GET only) or `if ctx.IsReconnect() { ... }` (WS reconnect with restored state). The older `if ctx.Action() == ""` idiom still works but conflates GETs with WS connects/reconnects and internal navigate POSTs.
- **BroadcastAction ordering:** `ctx.With*()` creates shallow copies. Call `ctx.BroadcastAction()` AFTER all `With*()` calls, or broadcasts queued before the copy won't propagate.
- **AssertPureState[T]():** Use in tests to catch dependency types accidentally in state structs.

See `docs/references/controller-pattern.md` for the full pattern guide with examples.

## Progressive Complexity

Standard HTML (forms, buttons, dialogs) handles Tier 1. `lvt-*` attributes (`lvt-on:`, `lvt-el:`, `lvt-fx:`, `lvt-mod:`, `lvt-form:`, `lvt-nav:`) are reserved for Tier 2 behaviors HTML cannot express.

See `docs/guides/progressive-complexity.md` for the full walkthrough.

## Architecture

5 operational phases: **Parse -> Build -> Diff -> Render -> Send**, each in `internal/{phase}/`. Supporting packages: `internal/keys/`, `internal/session/`, `internal/observe/`, `internal/upload/`, `internal/fuzz/`.

See `docs/design/ARCHITECTURE.md` for design decisions and `docs/design/CODE_STRUCTURE.md` for the complete file-by-file map.

## Key Data Structures

### TreeNode
```go
type TreeNode struct {
    Statics     []string                // Static HTML parts (key: "s")
    Dynamics    []interface{}           // Dynamic content indexed by position (keys: "0", "1", etc.)
    AutoKey     string                  // Range item key, replaces previous "_k" map key
    Fingerprint string                  // Structure hash (key: "f")
    Range       *RangeData              // Range operation data (key: "d")
    Metadata    *TreeMetadata           // Additional metadata (key: "m")
}
```
- Core structure for representing static/dynamic content
- Custom JSON marshaling maintains wire format compatibility (numeric keys for dynamics)
- Can be nested for complex templates

### Key Constructs
- `FieldConstruct`: Simple field replacement `{{.Field}}`
- `ConditionalConstruct`: If/else branches `{{if .Cond}}...{{else}}...{{end}}`
- `RangeConstruct`: Iteration `{{range .Items}}...{{end}}`
- `WithConstruct`: Context switching `{{with .Item}}...{{end}}`
- `TemplateInvokeConstruct`: Template invocation `{{template "name" .}}`

## Testing Strategy

### Test Files Structure
- `template_test.go`: Core template functionality tests (includes key injection tests)
- `tree_test.go`: Tree structure invariant validation
- `e2e_update_spec_test.go`: Tree update specification compliance tests
- Internal package tests: `internal/*/`

**Browser-based E2E Tests:**
Browser-based chromedp E2E tests are maintained in the lvt repository:
- Location: `github.com/livetemplate/lvt/e2e/livetemplate_core_test.go`
- These tests validate the library from a black-box perspective using real browser automation
- Tests include: complete rendering sequences, loading indicators, focus preservation, etc.

### Test Data
- `testdata/fixtures/`: Template fixtures for unit tests
- `testdata/golden/`: Golden files for snapshot testing
- `testdata/fuzz/`: Fuzz test corpus

### Running Tests
```bash
# Run all tests
go test -v ./...

# Run specific test categories
go test -run TestTemplate -v          # Template engine tests
go test -run TestTreeInvariant -v     # Tree invariant tests
go test -run TestKeyInjection -v      # Key injection tests
go test -run TestE2EUpdateSpec -v     # Update spec compliance tests

# Run with timeout
go test -v ./... -timeout=30s
```

## Development Conventions

### Release Process
- **Never create git tags manually** - Always use `release.sh` script for releases

### Code Style
1. **No unnecessary comments** - Code should be self-documenting
2. **Follow existing patterns** - Check neighboring code for conventions
3. **Use existing utilities** - Don't reinvent the wheel
4. **Maintain idiomatic Go** - Follow Go best practices

### Template Processing Flow
1. **Parse**: Template string -> Compiled template structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [livetemplate/livetemplate](https://github.com/livetemplate/livetemplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
