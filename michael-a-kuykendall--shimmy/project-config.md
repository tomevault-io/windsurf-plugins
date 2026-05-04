---
trigger: always_on
description: **Workflow for Each Issue:**
---

# ⚠️ CRITICAL SERVER RULE: NEVER cancel background servers with Ctrl+C! Use `&` or separate terminals!
# If you start a server (shimmy serve, python -m http.server, etc.) and then cancel it, IT WON'T RUN ANYMORE.
# Either use trailing `&` for background OR use different terminal tabs. You've done this mistake 12+ times today!

# 📋 CURRENT STATUS - December 7, 2025

## Active Work: Issue Resolution & Vision Module Development 🎯

### IMMEDIATE PRIORITY: Fix All Open Issues (17 total)
**Workflow for Each Issue:**
1. **Create Feature Branch**: `git checkout -b fix/issue-NNN-description`
2. **Implement Fix**: Make minimal, targeted changes
3. **Test Thoroughly**: Run local tests + regression suite
4. **Pass Release Gates**: `./scripts/dry-run-release.sh`
5. **Create PR**: Push branch, create PR with detailed description
6. **Merge & Monitor**: Merge PR, ensure CI passes, handle any issues

### Open Issues Priority Order:
1. **Critical (Blockers)**: #152 (Docker build), #140 (GGML assert), #139 (Unicode), #127 (MLX smoke), #114 (Distribution)
2. **High**: #113 (Open WebUI), #142 (AMD GPU), #147 (Multi-file models)
3. **Medium**: #145 (Model support), #144 (MLX default), #143 (uvx support)
4. **Low**: #153 (Swagger), #137 (Demo), #141 (OpenAI response.create), #150 (Server loads model), #151 (How shimmy works)

### LONG-TERM: Vision Module Development
- **Port Seer Tool**: Node.js vision analysis → Rust shimmy-vision feature
- **Licensing**: Keygen integration for paid vision features
- **Timeline**: After all issues resolved, implement vision module

### Development Workflow Rules:
- **NEVER work on main**: Always create feature branches
- **Test before commit**: `./scripts/dev-test.sh` or `cargo test`
- **Release gates mandatory**: `./scripts/dry-run-release.sh` before PR
- **Clean commits**: `cargo fmt`, `cargo clippy -- -D warnings`
- **Detailed PRs**: Include issue link, reproduction steps, test results

---

# Copilot / AI Assistant Operating Guide for Shimmy

This file teaches any AI assistant how to work effectively inside this repository. Keep replies lean, perform actions directly, and favor incremental verified changes.

## CRITICAL RULES - NEVER VIOLATE

### 1. NEVER Print Fake Validation
**WRONG**: `echo "✅ Build successful"` or `printf "All tests passing"`
**RIGHT**: Actually check: `ls -lh target/release/shimmy.exe && echo $? && ./shimmy --version`

- Never use echo/printf to print success messages
- Always verify with actual commands (ls, grep, test exit codes, run the binary)
- If you can't verify it, say "I cannot verify this yet" - don't fake it

### 2. NEVER Use `!` in Bash Commands
**WRONG**: `echo "Build finished!"` or `rg "println!" src/`
**RIGHT**: `printf "%s\n" "Build finished"` or `rg 'println\!' src/`

- Bash interprets `!` as history expansion (event not found error)
- Use printf instead of echo when printing messages with !
- **ALWAYS escape ! in regex patterns**: Use `'println\!'` not `"println!"`
- This happens constantly - CHECK EVERY COMMAND with ! before running

### 3. ALWAYS Use `&` for Background Processes
**WRONG**: Long-running commands without `&` (blocks terminal)
**RIGHT**: `command args &` (runs in background, keeps terminal available)

- Use `&` for servers, builds, uploads, or any long-running process
- This prevents blocking the terminal and allows continued work
- Essential for workflow efficiency on expensive compute instances

### 4. ZERO TOLERANCE FOR WARNINGS
**RULE**: Fix ALL warnings immediately when encountered - never proceed with warnings present
**ACTION**: Stop and fix each warning properly (understand the issue, implement correct solution)

- Warnings indicate poor software engineering that must be corrected
- No warnings allowed in any build output - achieve completely clean builds
- Fix warnings at their source, only suppress if genuinely unavoidable (like auto-generated code)
- This is non-negotiable - warnings = incomplete work that must be finished

### 3. Python Command is `py` NOT `python3`
**WRONG**: `python3 script.py`
**RIGHT**: `py script.py`

- Windows uses `py` launcher, not `python3`
- Check yourself before running Python commands

### 3. Read Documentation BEFORE Trial-and-Error
**WRONG**: Try random commands, see what works
**RIGHT**: `fetch_webpage` to get official docs, then execute correct command

- Your training data is 2+ years old
- APIs change, flags change, behavior changes
- Read current docs FIRST, especially for: cargo, git, build tools
- One doc lookup saves 10 failed attempts

## Mission
Shimmy is a single-binary local inference shim (GGUF + optional LoRA) exposing simple HTTP/SSE/WebSocket endpoints plus a CLI. Goal: fast, frictionless local LLM token streaming that can front other tools (e.g. punch-discovery, RustChain) and act as a drop‑in development aide.

## Core Components
- `src/engine/llama.rs`: llama.cpp backend via `llama-cpp-2` (feature `llama`).
- `src/api.rs`: `/api/generate` (POST, JSON) with optional SSE streaming and `/ws/generate` WebSocket streaming.
- `src/server.rs`: axum server wiring.
- `src/templates.rs`: prompt template families (ChatML, Llama3, OpenChat).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Michael-A-Kuykendall/shimmy](https://github.com/Michael-A-Kuykendall/shimmy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
