---
trigger: always_on
description: > Instructions for Claude Code, pi, Cursor, Copilot, and other AI coding agents working on this project.
---

# AGENTS.md — AI Coding Agent Instructions for Feluda

> Instructions for Claude Code, pi, Cursor, Copilot, and other AI coding agents working on this project.

---

## Project Overview

**Feluda** is a Rust-based CLI tool that analyzes project dependencies, identifies their licenses, and flags any that restrict personal or commercial usage or are incompatible with your project's license.

- **Repository:** https://github.com/anistark/feluda
- **Crate:** https://crates.io/crates/feluda
- **Docs:** https://feluda.readthedocs.io
- **License:** MIT
- **Minimum Rust Version:** 1.85
- **Recommended Rust Version:** Latest stable

---

## ⚠️ Core Concepts — Read This First

### What Feluda Does

Feluda scans a project's dependency files, resolves each dependency's license (from local files or the GitHub API), and produces a report. It supports **eight language ecosystems**, multiple output formats, SBOM generation, and CI/CD integration.

### The Analysis Pipeline

```
User invokes feluda [--path ./project] [--language rust] [flags...]
        ↓
src/parser.rs — discover project files, detect language(s)
        ↓
src/languages/<lang>.rs — parse dependency manifest, resolve licenses
        ↓ (local file check first, then GitHub API fallback)
src/licenses.rs — enrich with compatibility, OSI status, restrictiveness
        ↓
src/reporter.rs — format output (text/JSON/YAML/CI/gist)
   or src/table.rs — TUI mode (ratatui)
   or src/generate.rs — NOTICE/THIRD_PARTY_LICENSES files
   or src/sbom/ — SPDX/CycloneDX generation
```

### Language Ecosystems

| Language | Manifest File(s) | Parser Module | Local License Detection |
|----------|-------------------|---------------|------------------------|
| **Rust** | `Cargo.toml` | `src/languages/rust.rs` | `Cargo.toml` license field |
| **Node.js** | `package.json` | `src/languages/node.rs` | `node_modules/*/LICENSE` files |
| **Go** | `go.mod` | `src/languages/go.rs` | — |
| **Python** | `requirements.txt`, `Pipfile.lock`, `pip_freeze.txt`, `pyproject.toml` | `src/languages/python.rs` | — |
| **C** | `configure.ac`, `configure.in`, `Makefile` | `src/languages/c.rs` | — |
| **C++** | `vcpkg.json`, `conanfile.txt`, `CMakeLists.txt`, `MODULE.bazel` | `src/languages/cpp.rs` | — |
| **R** | `DESCRIPTION`, `renv.lock` | `src/languages/r.rs` | — |
| **.NET** | `.csproj`, `.fsproj`, `.vbproj`, `.slnx` | `src/languages/dotnet.rs` | — |

### Critical Rules

1. **Local-first license resolution.** Feluda checks local files before making network requests. The `--no-local` flag overrides this. Respect this order in all language parsers.

2. **GitHub API rate limits matter.** Unauthenticated: 60 req/hr. Authenticated (`--github-token`): 5,000 req/hr. Never make unnecessary API calls. Use the cache system (`src/cache.rs`).

3. **License compatibility is configurable.** The compatibility matrix lives in `config/license_compatibility.toml`. User overrides come from `.feluda.toml` and environment variables. Don't hardcode compatibility rules.

4. **Configuration precedence:** Environment variables > `.feluda.toml` > defaults. This is handled by `figment` in `src/config.rs`. Don't bypass this chain.

5. **Each language parser is self-contained.** A language module in `src/languages/` handles discovery, parsing, and license resolution for its ecosystem. Don't add cross-language coupling.

6. **Error types live in `src/debug.rs`.** All custom errors use `FeludaError` (thiserror-based). Add new variants there, not in individual modules.

---

## After Every Set of Changes

After completing any set of changes, **always** run these in order:

1. **`just format`** — Format all Rust code.
2. **`just lint`** — Run clippy (zero warnings enforced) and rustfmt check.
3. **`just test`** — Run the full test suite.

For a full CI-equivalent check:

4. **`just test-ci`** — Runs format-check → clippy → test (mirrors GitHub Actions).

Do not consider a change complete until all of the above pass cleanly.

### Additional Housekeeping

- **Prefer `just` commands** over raw `cargo` commands. The justfile handles sequencing correctly.
- **Prompt the user if `AGENTS.md` needs updating.** If your changes alter architecture, CLI commands, language support, key file locations, or behavioural conventions, tell the user: *"This change may require an update to AGENTS.md — would you like me to update it?"*

### Planning Documents

- **Check `plan/` for active plans** when a related task is mentioned. Files like `plan/ROADMAP.md` and `plan/*_IMPLEMENTATION.md` contain detailed implementation plans, checklists, and phase tracking.
- **`plan/` is for local planning only.** It is gitignored — never commit it. Use it to understand context, track progress, and follow implementation checklists.
- **Create new planning docs here** when scoping features or multi-step work (e.g., `plan/NEW_LANG_SWIFT.md`, `plan/SBOM_V2.md`). Keep them focused — one doc per initiative.

### Git Discipline

- **Do not run `git add` or `git commit` unless the user explicitly asks.** Stage and commit only on direct request.
- **When the user asks to commit**, review all staged/unstaged changes and prepare:
  - A **brief title** following conventional commits (`feat:`, `fix:`, `chore:`, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anistark/feluda](https://github.com/anistark/feluda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
