---
trigger: always_on
description: This file provides guidance to Codex when working with this repository.
---

# AGENTS.md

This file provides guidance to Codex when working with this repository.

## Project Overview

**AgentShield** is a Rust-based, offline-first security scanner for AI agent extensions
(MCP servers, OpenClaw skills, LangChain tools). It produces SARIF output compatible
with GitHub Code Scanning.

## Repository Structure

```
agentshield/
├── src/
│   ├── lib.rs                    # Public API: scan(), render_report()
│   ├── error.rs                  # ShieldError (thiserror)
│   ├── bin/cli.rs                # Clap CLI: scan, list-rules, init
│   ├── ir/                       # Intermediate Representation (ScanTarget)
│   │   ├── mod.rs                # ScanTarget, Framework, SourceFile, ArgumentSource
│   │   ├── tool_surface.rs       # Tool definitions, permissions
│   │   ├── execution_surface.rs  # Commands, file IO, network IO
│   │   ├── data_surface.rs       # Sources, sinks, taint paths
│   │   ├── dependency_surface.rs # Dependencies, lockfiles
│   │   └── provenance_surface.rs # Author, repo, license
│   ├── adapter/                  # Framework → IR (3-phase pipeline)
│   │   ├── mod.rs                # Adapter trait, auto_detect_and_load(root, ignore_tests)
│   │   ├── mcp.rs                # MCP server adapter + is_test_file() + shared helpers
│   │   ├── openclaw.rs           # OpenClaw SKILL.md adapter
│   │   ├── crewai.rs             # CrewAI adapter (BaseTool, @tool)
│   │   └── langchain.rs          # LangChain adapter (@tool, BaseTool, langgraph)
│   ├── parser/                   # Language parsers
│   │   ├── mod.rs                # Parser trait, ParsedFile, FunctionDef, CallSite
│   │   ├── python.rs             # tree-sitter Python + regex patterns
│   │   ├── typescript.rs         # tree-sitter TypeScript/TSX + regex fallback
│   │   ├── shell.rs              # Regex-based shell parser
│   │   └── json_schema.rs        # JSON Schema → ToolSurface
│   ├── analysis/                 # Static analysis
│   │   ├── mod.rs                # Module exports
│   │   ├── capability.rs         # Capability escalation scoring
│   │   ├── cross_file.rs         # Cross-file sanitizer-aware validation (v0.2.2)
│   │   └── supply_chain.rs       # Typosquat detection
│   ├── rules/                    # Detection engine
│   │   ├── mod.rs                # RuleEngine, Detector trait
│   │   ├── finding.rs            # Finding, Severity, Evidence structs
│   │   ├── registry.rs           # Rule metadata registry
│   │   ├── policy.rs             # Policy evaluation (.agentshield.toml)
│   │   └── builtin/              # 12 built-in detectors (SHIELD-001..012)
│   ├── output/                   # Report formatters
│   │   ├── mod.rs                # OutputFormat enum, render()
│   │   ├── console.rs            # Plain text
│   │   ├── json.rs               # JSON
│   │   ├── sarif.rs              # SARIF 2.1.0
│   │   └── html.rs               # Self-contained HTML
│   └── config/                   # .agentshield.toml parsing (policy + scan sections)
├── tests/fixtures/               # Test fixtures (safe + vulnerable)
│   ├── mcp_servers/
│   │   ├── safe_calculator/      # Zero-finding baseline
│   │   ├── safe_filesystem/      # Cross-file validation test (v0.2.2)
│   │   ├── vuln_cmd_inject/      # SHIELD-001 true positive
│   │   ├── vuln_ssrf/            # SHIELD-003 true positive
│   │   └── vuln_cred_exfil/      # SHIELD-002 true positive
│   ├── crewai_project/           # CrewAI adapter test (v0.2.4)
│   └── langchain_project/       # LangChain adapter test (v0.2.4)
├── vscode/                       # VS Code extension (v0.1.0)
│   ├── package.json              # Extension manifest
│   ├── tsconfig.json             # TypeScript config
│   └── src/                      # Extension source (TypeScript)
│       ├── extension.ts          # Activate, commands, auto-scan
│       ├── scanner.ts            # Spawn binary, parse JSON
│       ├── diagnostics.ts        # Finding → vscode.Diagnostic
│       └── types.ts              # JSON interfaces (mirrors Rust)
├── .github/workflows/
│   ├── ci.yml                    # Test + clippy + fmt + smoke
│   └── release.yml               # 5-platform binary builds
└── action.yml                    # GitHub Action (composite)
```

## Common Commands

```bash
# Build
cargo build --release

# Test (95 tests)
cargo test

# Lint
cargo clippy -- -D warnings
cargo fmt --check

# Run CLI
cargo run -- scan tests/fixtures/mcp_servers/vuln_cmd_inject
cargo run -- scan . --ignore-tests --format html --output report.html
cargo run -- list-rules
```

## Huly Skill

When working with Huly issues, projects, labels, milestones, or documents, use
the repo skill at `skills/huly/SKILL.md`.

The skill prefers the official Huly Platform API with token auth. Required
environment variables are:

```bash
HULY_URL=https://huly.app/workbench/<workspace>/
HULY_WORKSPACE=<workspace-slug>
HULY_PROJECT=<project-identifier>
HULY_API_TOKEN=<token>
```

Compatibility token fallbacks are `HULY_TOKEN` and `HULY_APY_TOKEN`. Do not
print tokens or full environment values. Run a read-only project lookup before
creating or updating Huly data, and make write scripts idempotent.

## RTK Usage for Agent Check Loops


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aiconnai/agentshield](https://github.com/aiconnai/agentshield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
