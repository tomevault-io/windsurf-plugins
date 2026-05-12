---
trigger: always_on
description: Stringer is a codebase archaeology tool that mines existing repositories to produce [Beads](https://github.com/steveyegge/beads)-formatted issues. It solves the cold-start problem: when you adopt Beads on a mature codebase, agents wake up with zero context. Stringer gives them instant situational awareness by extracting actionable work items from signals already present in the repo.
---

# AGENTS.md — Stringer

## What is Stringer?

Stringer is a codebase archaeology tool that mines existing repositories to produce [Beads](https://github.com/steveyegge/beads)-formatted issues. It solves the cold-start problem: when you adopt Beads on a mature codebase, agents wake up with zero context. Stringer gives them instant situational awareness by extracting actionable work items from signals already present in the repo.

## Architecture

```
stringer/
├── cmd/stringer/           # CLI entrypoint
│   ├── main.go                 # cobra root setup
│   ├── root.go                 # root command, global flags
│   ├── scan.go                 # scan subcommand and flags
│   ├── report.go               # report subcommand
│   ├── context.go              # context subcommand
│   ├── docs.go                 # docs subcommand
│   ├── init.go                 # init subcommand (bootstrap stringer in a repo)
│   ├── config.go               # config get/set/list subcommands
│   ├── collectors.go           # collectors list/info subcommands (info shows thresholds, supports --json)
│   ├── baseline.go             # baseline create/suppress/list/remove/status subcommands
│   ├── mcp.go                  # mcp serve subcommand (MCP server)
│   ├── validate.go             # validate subcommand (JSONL validation)
│   ├── version.go              # version subcommand
│   ├── configwiring.go         # shared flag-to-config wiring
│   ├── exitcodes.go            # exit code constants
│   └── fs.go                   # filesystem helpers
├── internal/
│   ├── beads/              # Beads integration
│   │   ├── conventions.go      # Beads naming and format conventions
│   │   ├── dedup.go            # Beads-aware signal deduplication
│   │   └── reader.go           # Read existing beads from .beads/ directory
│   ├── bootstrap/          # stringer init bootstrapping
│   │   ├── bootstrap.go        # Bootstrap orchestration
│   │   ├── detect.go           # Project detection (language, framework, CI)
│   │   ├── config.go           # Generate .stringer.yaml defaults
│   │   ├── agentsmd.go         # Append stringer section to AGENTS.md
│   │   └── mcpjson.go          # Generate .mcp.json for Claude Code
│   ├── collector/          # Collector registry and interface
│   │   └── collector.go        # Register(), List(), Get(), Collector interface
│   ├── collectors/         # Signal extraction modules (one file per collector)
│   │   ├── todos.go            # TODO/FIXME/HACK/XXX/BUG/OPTIMIZE scanner
│   │   ├── gitlog.go           # Reverts, high-churn files, stale branches
│   │   ├── patterns.go         # Large files, missing tests, low test coverage ratios (Go, JS/TS, Python, Ruby, Java, Kotlin, Rust, C#, PHP, Swift)
│   │   ├── lotteryrisk*.go     # Lottery risk: core, ownership math, review analysis
│   │   ├── github.go           # GitHub issues, PRs, and review comments
│   │   ├── dephealth*.go       # Dependency health: 10 ecosystems (Go, npm, Cargo, Maven, NuGet, PyPI, Packagist, SwiftPM, sbt, Hex)
│   │   ├── vuln*.go            # Vuln scanner: 11 ecosystems via OSV.dev (+ PHP, Swift, Scala, Elixir parsers)
│   │   ├── configdrift.go       # Config drift: env var drift, dead keys, inconsistent defaults
│   │   ├── apidrift.go         # API drift: undocumented routes, unimplemented spec paths, stale versions
│   │   ├── docstale.go         # Doc staleness: stale docs, co-change drift, broken links
│   │   ├── duplication*.go     # Code duplication: exact clones (Type 1) and near-clones (Type 2) via FNV-64a sliding window
│   │   ├── coupling*.go        # Coupling: circular dependencies (Tarjan's SCC) and high fan-out modules via import graph
│   │   ├── complexity.go       # Complexity: AST-based for Go (cyclomatic/cognitive/nesting), regex-based for other languages
│   │   ├── complexity_go.go    # Go AST analysis: cyclomatic, cognitive, nesting depth via go/parser
│   │   ├── githygiene.go       # Git hygiene: large binaries, merge conflicts, committed secrets, mixed line endings
│   │   ├── secrets.go          # Secret detection: 24+ built-in patterns, custom patterns, allowlist, entropy detection
│   │   └── duration.go         # Duration parsing helpers
│   ├── analysis/           # LLM-powered analysis
│   │   ├── cluster.go          # Signal clustering via LLM
│   │   ├── priority.go         # Priority inference via LLM
│   │   └── dependency.go       # Dependency detection via LLM
│   ├── config/             # .stringer.yaml config file support
│   │   ├── config.go           # Config and CollectorConfig structs
│   │   ├── yaml.go             # Load(), Write(), LoadRaw(), WriteFile()
│   │   ├── validate.go         # Validate() — multi-error validation
│   │   ├── merge.go            # Merge() — file config + CLI merge
│   │   ├── keypath.go          # Dot-notation key path navigation
│   │   └── global.go           # Global config (~/.config/stringer/)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davetashner/stringer](https://github.com/davetashner/stringer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
