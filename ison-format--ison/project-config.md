---
trigger: always_on
description: This file provides comprehensive guidance for AI coding agents (Claude Code, Cursor, GitHub Copilot, etc.) working on the ISON project.
---

# AGENTS.md - AI Coding Agent Guidelines

This file provides comprehensive guidance for AI coding agents (Claude Code, Cursor, GitHub Copilot, etc.) working on the ISON project.

## Project Overview

**ISON (Interchange Simple Object Notation)** is a token-efficient data format optimized for LLMs and Agentic AI workflows. The project is a monorepo containing parser implementations across 5 languages plus tooling.

- **Website:** https://www.ison.dev
- **Documentation:** https://www.getison.com
- **Author:** Mahesh Vaikri
- **License:** MIT

## Repository Structure

```
ison/
├── ison-js/           # JavaScript parser (NPM: ison-parser)
├── ison-ts/           # TypeScript parser + validation (NPM: ison-ts)
├── ison-py/           # Python parser + validation (PyPI: ison-py)
├── ison-rust/         # Rust parser (Crates.io: ison-rs)
├── ison-cpp/          # C++ header-only parser
├── ison-go/           # Go parser + validation
├── ison-cli/          # Python CLI tool (PyPI: ison-cli)
├── ison-vscode/       # VS Code extension (Marketplace: ison-lang)
├── n8n-nodes-ison/    # n8n community node
├── isonantic-ts/      # DEPRECATED - use ison-ts/validation
├── isonantic/         # DEPRECATED - use ison_parser.validation
├── isonantic-go/      # DEPRECATED - use ison-go/validation
├── isonantic-rust/    # Rust validation (NOT deprecated - separate crate)
├── isonantic-cpp/     # C++ validation (NOT deprecated - separate header)
├── benchmark/         # Token efficiency benchmarks
├── images/            # Logo and assets
└── README.md          # Main documentation
```

## Package Registry Mapping

| Directory | Package Name | Registry | Current Version |
|-----------|--------------|----------|-----------------|
| ison-js | `ison-parser` | NPM | 1.0.2 |
| ison-ts | `ison-ts` | NPM | 1.0.2 |
| ison-py | `ison-py` | PyPI | 1.0.3 |
| ison-rust | `ison-rs` | Crates.io | 1.0.2 |
| ison-go | `github.com/ISON-format/ison/ison-go` | Go Modules | - |
| ison-cli | `ison-cli` | PyPI | 1.0.0 |
| ison-vscode | `ison-lang` | VS Code Marketplace | 1.0.2 |
| n8n-nodes-ison | `n8n-nodes-ison` | NPM | 1.0.1 |
| isonantic-ts | `isonantic-ts` | NPM | 1.0.0 (deprecated) |
| isonantic | `isonantic` | PyPI | 1.0.1 (deprecated) |
| isonantic-rust | `isonantic-rs` | Crates.io | 1.0.0 |

## ISON Format Quick Reference

```ison
# Comments start with #

table.users                          # Block: kind.name
id:int name:string email active:bool # Fields with optional types
1 Alice alice@example.com true       # Data rows (space-separated)
2 "Bob Smith" bob@example.com false  # Quoted strings for spaces
3 ~ ~ true                           # ~ or null for null values

table.orders
id user_id product
1 :1 Widget                          # :1 = reference to id 1
2 :user:42 Gadget                    # :user:42 = namespaced reference
3 :OWNS:5 Gizmo                      # :OWNS:5 = relationship reference (UPPERCASE)

object.config                        # Single-row object block
key value
debug true
---                                  # Summary separator
count 100                            # Summary row
```

### ISONL (Streaming Format)
```
table.users|id name email|1 Alice alice@example.com
table.users|id name email|2 Bob bob@example.com
```

## Development Commands

### JavaScript (ison-js)
```bash
cd ison-js
npm install
npm test                    # Run tests
npm run build              # Build dist/
npm pack                   # Create tarball
npm publish                # Publish to NPM
```

### TypeScript (ison-ts)
```bash
cd ison-ts
npm install
npm run build              # tsup build
npm test                   # vitest
npm publish
```

### Python (ison-py, isonantic, ison-cli)
```bash
cd ison-py
pip install -e .           # Editable install
pytest                     # Run tests
python -m build            # Build wheel
twine upload dist/*        # Publish to PyPI
```

### Rust (ison-rust, isonantic-rust)
```bash
cd ison-rust
cargo test                 # Run tests
cargo build --release      # Build
cargo publish              # Publish to Crates.io
```

### C++ (ison-cpp, isonantic-cpp)
```bash
cd ison-cpp
mkdir build && cd build
cmake ..
cmake --build .
ctest                      # Run tests
```

### Go (ison-go, isonantic-go)
```bash
cd ison-go
go test -v ./...           # Run tests
go mod tidy                # Tidy dependencies
# Publishing: tag and push (proxy.golang.org auto-indexes)
```

### VS Code Extension (ison-vscode)
```bash
cd ison-vscode
npm install
npm run compile            # TypeScript compile
npm run package            # Create .vsix
vsce publish               # Publish to Marketplace
```

## Testing Requirements

All packages must maintain test coverage. Current test counts:

| Package | Tests | Command |
|---------|-------|---------|
| ison-js | 80 (33 parser + 47 validation) | `npm test` |
| ison-ts | 23 | `npm test` |
| ison-py | 212+ (31 parser + validation) | `pytest` |
| ison-rust | 10 (9 + doctests) | `cargo test` |
| ison-cpp | 30 | `ctest` |
| ison-go | 40+ | `go test -v ./...` |

**Total: 395+ tests across all packages**

## Coding Conventions

### General
- MIT License header not required in source files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ISON-format/ison](https://github.com/ISON-format/ison) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
