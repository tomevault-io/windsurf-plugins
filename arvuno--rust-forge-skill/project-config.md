---
trigger: always_on
description: This skill makes AI agents generate idiomatic, maintainable, secure, and testable Rust projects. It provides templates, guides, validation scripts, checklists, and enforcement rules that agents follow strictly when producing Rust code.
---

# Rust Forge Skill

## Purpose

This skill makes AI agents generate idiomatic, maintainable, secure, and testable Rust projects. It provides templates, guides, validation scripts, checklists, and enforcement rules that agents follow strictly when producing Rust code.

Agents using this skill will:
- Scaffold projects that pass `cargo clippy -D warnings` and `cargo test`
- Apply correct error handling strategies (`thiserror` for libraries, `anyhow` for applications)
- Enforce memory safety via SAFETY documentation on all unsafe blocks
- Use appropriate layer architecture (API → Service → Repository → Domain)
- Configure CI/CD with MSRV enforcement, security audits, and coverage tracking
- Avoid the 25+ anti-patterns catalogued in this document

---

## When To Use

Use this skill when an agent needs to:

| Scenario | Template |
|---|---|
| New Rust CLI application | `cli-app/` |
| New Rust HTTP API / web service | `axum-api/` |
| Publishable library crate | `library-crate/` |
| Cargo workspace with multiple crates | `workspace-service/` + `scripts/generate_cargo_workspace.sh` |
| Rust FFI wrapper for C/C++ library | `ffi-wrapper/` |
| Rust WASM module for browser or edge runtime | `wasm-module/` |
| Refactor low-quality Rust to idiomatic patterns | `guides/12-rust-anti-patterns.md` + `guides/03-error-handling-anyhow-thiserror.md` |
| Add tests, benchmarks, or fuzz targets | `guides/06-testing-bench-fuzz.md` |
| Add CI/CD pipeline | `ci/github-actions-rust.yml` + `ci/github-actions-rust-security.yml` |
| Add security gates (audit, unsafe review) | `guides/08-security-unsafe-audit.md` + `scripts/audit_unsafe.sh` |
| Audit existing unsafe code | `examples/agent-prompts/unsafe-audit.md` + `scripts/audit_unsafe.sh` |

**Do not use** this skill for: fixing compilation errors in existing code (use `systematic-debugging`), researching Rust internals (use web search), or writing tutorial content.

---

## Self-Testing

The skill pack includes self-test scenarios in `self-tests/` that verify an agent can follow the skill guidance to complete real tasks. Each scenario gives an agent a concrete task plus embedded example code, and validates the agent's output against an expected report shape. See `self-tests/README.md` for execution instructions.

---

## MCP/Agent Integration Layer

This skill includes a machine-readable MCP layer for programmatic loading by AI agents and tool runners.

| File | Purpose |
|---|---|
| `mcp/manifest.example.json` | Machine-readable skill manifest (name, version, capabilities, validation commands) |
| `mcp/tool-contract.md` | Formal agent protocol contract (input schema, output schema, safety invariants) |
| `mcp/agent-loader-instructions.md` | Step-by-step instructions for agents that need explicit guidance |
| `examples/agent-prompts/load-rust-forge-skill.md` | Copy-paste prompt for any agent to load this skill |

### Loading Methods

**Claude Code:** The `Skill` tool reads `SKILL.md` automatically. Use `/skill rust-forge-skill` to explicitly invoke.

**Other agents and tool runners:** Use `mcp/manifest.example.json` for auto-discovery, `mcp/tool-contract.md` for protocol compliance, and `mcp/agent-loader-instructions.md` for step-by-step guidance. See `mcp/README.md` for full details on all supported loading methods (IDE plugins, MCP servers, repo-scanning agents, CLI agents).

### Capability Routing

| Capability ID | Use Case | Required Inputs |
|---|---|---|
| `rust_project_scaffold` | New project from template | `project_path`, `project_type`, `project_name` |
| `rust_project_audit` | Audit existing project | `project_path` |
| `rust_error_refactor` | Refactor error handling | `project_path`, `project_type` |
| `rust_async_service` | Async API with Axum | `project_path`, `project_type` |
| `rust_ffi_audit` | Unsafe/FFI audit | `project_path` |
| `rust_security_validation` | Security gate run | `project_path` |

See `mcp/manifest.example.json` for the full capability definitions including optional inputs and expected outputs.

### Safety Invariants (Non-Negotiable)

All agents must uphold these invariants regardless of task:

1. **No destructive rewrite without explicit user request** — do not delete files outside `project_path`
2. **No unsafe without audit** — every `unsafe` block needs a `SAFETY` comment
3. **No dependency bloat** — confirm necessity before adding deps
4. **No secret exposure** — no API keys or tokens in source, logs, or output
5. **No unwitnessed test passing** — run and report actual validation output
6. **No unvalidated deploy** — verdict must match actual validation results

See `mcp/tool-contract.md` for the full safety invariant definitions.

---

## Agent Operating Rules

Agents **MUST** follow every rule in this section. Violations require written justification.

### Project Inspection Before Changes

```
Before adding a crate, dependency, or module:
1. Read existing Cargo.toml — understand current deps, edition, workspace membership
2. Read existing src/lib.rs or src/main.rs — understand current architecture

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arvuno/rust-forge-skill](https://github.com/Arvuno/rust-forge-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
