---
trigger: always_on
description: A shell completion registry powered by [carapace](https://github.com/carapace-sh/carapace). This is **not** the carapace library itself — it aggregates completers from multiple sources, resolves which variant to use per command, exposes shared actions as macros, and provides a runtime for spec-based completions.
---

# carapace-bin

A shell completion registry powered by [carapace](https://github.com/carapace-sh/carapace). This is **not** the carapace library itself — it aggregates completers from multiple sources, resolves which variant to use per command, exposes shared actions as macros, and provides a runtime for spec-based completions.

For carapace library concepts (actions, macros, specs, integration, scraping), see the **carapace** skill in `skills/carapace/`.

## Commands

| Command | Notes |
|---------|-------|
| `go generate ./cmd/...` | Regenerate all code (completer lists, macros, conditions). **Required after adding/modifying public actions or completers** |
| `go install -v -tags force_all ./cmd/carapace` | Build with all platform completers |
| `go install -v ./cmd/carapace` | Build for current platform only |
| `go test -v ./cmd/...` | Run tests |
| `gofmt -d -s .` | Format check (must produce empty diff) |
| `staticcheck ./...` | Static analysis |
| `go run ./cmd/carapace-lint completers/*/*/cmd/*.go` | Alphabetical ordering linter |
| `go run ./cmd/carapace-lint --fix-flags-order completers/*/*/cmd/*.go` | Auto-fix ordering |

No Makefile — orchestration is in `.github/workflows/go.yml`.

## MCP & Skills

This project provides a carapace MCP server. When working here, use it:

- **`list_macros`** / **`carapace_carapace_list_macros`** — Look up available macros, their signatures, and descriptions before referencing them in code or specs
- **`complete_command`** / **`carapace_carapace_complete_command`** — Test completion output for any completer
- **`complete_macro`** / **`carapace_carapace_complete_macro`** — Test macro completion output

The `skills/carapace/` directory contains the composite carapace skill with detailed guides. **Use it** when working on the corresponding aspects. The SKILL.md entry point routes to the right sub-resource:

| Sub-resource | When to use |
|-------------|-------------|
| `references/action.md` | Creating/modifying shared actions, naming, opts, Uid/QueryF, modifiers |
| `references/spec.md` | Writing YAML completion spec files |
| `references/macro.md` | Macro types, formatting, signature lookup, cross-executive macros |
| `references/env.md` | Environment variable completion (Go definitions, user YAML overrides) |
| `references/scrape.md` | Generating specs from CLI source code (patch-and-container) |
| `references/integrate.md` | Integrating carapace into cobra CLIs (PreRun, PreInvoke, bridge) |
| `references/setup.md` | Shell integration, environment variables, overlays, extensions |
| `references/choice.md` | Choices, variants, bridges (implicit shell + explicit framework), completer resolution |
| `references/mcp.md` | MCP server tools (complete_command, complete_macro, list_macros, codegen), client setup, protocol |
| `references/man.md` | Man page documentation (carapace-man), YAML man pages, UID-to-file mapping, known concepts vs live entities |
| `references/convert.md` | Converting YAML specs to native Go completers |

## Project Structure

```
cmd/
  carapace/              # Main binary: spec loader, macro dispatcher, completer invoker
  carapace-generate/     # Code generator (macros, completer lists, conditions, specs)
  carapace-lint/         # Custom linter: alphabetical flag/flag-action ordering
  carapace-parse/        # Spec parser
  carapace-shim/         # Shell shim manager
completers/              # Individual completers
  common/                # Cross-platform (some need networking)
  unix/                  # Unix-only
  linux/                 # Linux-only (~100+)
  darwin/                # macOS-only (3)
  bsd/                   # BSD variants
  windows/               # Windows-only
  android/               # Android/Termux
  bash/                  # Bash-only
pkg/
  actions/               # Shared actions exposed as macros
    actions_generated.go  # Macro map (GENERATED — do not edit)
    net/ os/ fs/ color/   # Domain categories
    tools/                # Tool-specific (one subpackage per tool)
  styles/                 # Project-specific style definitions
  conditions/             # Condition helpers (conditions_generated.go is GENERATED)
  env/                    # Environment variable helpers
  util/                   # Utility functions
  completer/              # Completer registry helpers
internal/condition/       # Internal condition logic
skills/
  carapace/               Composite skill for carapace library concepts
    SKILL.md                Entry point with routing table
    references/             Sub-resources loaded on demand
docs/                     # mdBook documentation
.docker/                  # Docker Compose services for distro testing
```

## Completer Registry and Resolution


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carapace-sh/carapace-bin](https://github.com/carapace-sh/carapace-bin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
