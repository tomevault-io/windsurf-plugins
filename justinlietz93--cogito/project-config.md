---
trigger: always_on
description: **Generated on:** September 16, 2025 at 11:02 PM CDT
---

# Technical Summary Report

**Generated on:** September 16, 2025 at 11:02 PM CDT

---

## Generated Summary

### Core Mission

- The agent's primary goal is to guide, enforce, and analyze the implementation of software components against a set of predefined engineering standards, security policies, and architectural guidelines.
- Ensure adherence to current best practices (modular monolith, clean architecture, security, scalability, maintainability, testability, framework independence) and drive the roadmap toward future improvements emphasizing robustness, security, reliability, and observability.
- Act as an independent, low-friction assistant tailored for solo developers, making safe, low-risk changes, preserving project context, and diligently following architectural and operational standards.
- Ensure code quality and security by integrating with Codacy's MCP Server, running analyses, and applying fixes.
- Track all work using the GitHub CLI (`/mnt/samsung_ssd/notes/github_tools/cli.py`).

### Constraints & Rules

- **General Code & Architecture:**
  - **CRITICAL**: No source code file shall exceed 500 lines of code (LOC). Break large files into smaller, focused components.
  - Dependencies must flow inward only. Outer layers depend on inner layers via abstractions (interfaces); inner layers must never reference or depend on outer layers.
  - All cross-layer communication must occur via interfaces defined in abstraction projects.
  - Controllers/handlers must be thin; delegate business logic to services/managers.
  - Use DTOs for data transfer.
  - Presentation layer: No direct database access or business rules.
  - Business Logic layer: Contain all business rules and validations; be framework-agnostic. Use repository pattern for data access.
  - Domain layer: Pure models (POCOs), no business logic, framework-independent.
  - Infrastructure/Persistence layer: Implement repository interfaces, handle data persistence/ORM details, no business logic.
  - Design for unit testing by depending on abstractions.
  - Each project/module should have a single responsibility.
  - **CRITICAL**: Never create "shims" when replacing deprecated functions or patterns. Always do a full replacement unless explicitly instructed otherwise to avoid technical debt.
  - Adopt a comprehensive, automated testing pyramid: heavy unit tests (fast, isolated), moderate integration (components interaction), light end-to-end (full flow).
  - Use Test-Driven Development (TDD) or Behavior-Driven Development (BDD). Aim for 80%+ test coverage.
  - Always add brief logged explanations for any programmatic test that explains what the test is actually attempting to prove/disprove.
  - Implement layered validation (client-side, server-side, DB-level) with a fail-early principle. Automate 90% of validation.
- **Docstrings & Documentation:**
  - **CRITICAL**: ALWAYS CREATE FULL, PROFESSIONAL, AND DESCRIPTIVE DOCSTRINGS for all functions and classes.
  - Module docstring MUST state: purpose, external dependencies (CLI/HTTP), fallback semantics, timeout strategy.
  - Public function docstrings MUST include: summary, parameters, return description, raised exceptions/failure modes, side effects (I/O, persistence), and timeout/retry notes where relevant.
- **Timeout & Error Handling:**
  - Always use `get_timeout_config()` for HTTP calls, local CLI invocation, and streaming start phases.
  - Wrap blocking/start segments in `operation_timeout` (supports nesting; restores previous handlers and timers).
  - Never introduce hard-coded numeric timeouts in provider code or related tests.
  - Log provider + operation context on exception paths before fallback.
  - Replace broad silent suppression with explicit `try/except` and structured logging.
  - Required log context keys: `provider`, `operation`, `stage` (start|mid_stream|finalize|retry), `failure_class`, `fallback_used`.
  - Each fallback path must log its trigger exactly once (avoid duplicated messages).
  - On live fetch failure, return cached snapshot (models/metadata) after logging primary cause.
  - Never fail silently or mask the underlying exception type in logs.
- **Subprocess & Security:**
  - Resolve executables via `shutil.which` to an absolute path.
  - Validate executables: basename, regular file, executable bit set, not group/other writable.
  - Use fixed whitelisted argument lists; never `shell=True`.
  - Never interpolate user input into subprocess arguments.
  - Log subprocess failures (warning when falling back; error when aborting) — no silent suppression.
  - Each `# nosec` requires an inline justification (e.g., `# nosec B603 - validated fixed arg list`).
  - **CRITICAL**: IMMEDIATELY after ANY package manager operations (e.g., `npm/yarn/pnpm install`, adding dependencies to `package.json`, `requirements.txt`, `pom.xml`, `build.gradle`), run `codacy_cli_analyze` with `rootPath` and `tool` set to "trivy" (`file` empty/unset). If vulnerabilities are found, stop all other operations, propose and apply fixes, and only continue with the original task after security issues are resolved.
- **Streaming & Metrics:**
  - All streaming implementations MUST use `BaseStreamingAdapter`; remove bespoke streaming loops (no transitional shim).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [justinlietz93/Cogito](https://github.com/justinlietz93/Cogito) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
