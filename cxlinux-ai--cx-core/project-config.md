---
trigger: always_on
description: CX Linux is the AI-native OS layer for the $50B Linux system administration market. Instead of memorizing commands and googling errors, users describe their intent and the AI executes it safely and intelligently.
---

# GitHub Copilot Instructions for CX Linux

## Project Overview
CX Linux is the AI-native OS layer for the $50B Linux system administration market. Instead of memorizing commands and googling errors, users describe their intent and the AI executes it safely and intelligently.

## What Copilot is Best At Working On

### 🎯 High-Value Areas for AI Assistance

GitHub Copilot excels at helping with these CX Terminal components:

#### 1. **AI Agent Development** ⭐⭐⭐
- **Why:** Repetitive patterns with agent trait implementations
- **Tasks:** Adding new specialized agents (cloud providers, package managers, system introspection)
- **Pattern:** `impl Agent for NewAgent { async fn execute() { ... } }`
- **Impact:** Each new agent multiplies platform capabilities

#### 2. **Voice/Audio Pipeline** ⭐⭐⭐
- **Why:** Complex audio processing with standard patterns
- **Tasks:** Speech transcription, audio buffering, voice command mapping to CLI
- **Components:** `voice/` module with cpal audio capture
- **Pattern:** Async audio stream processing, VAD (Voice Activity Detection)

#### 3. **Learning/ML Components** ⭐⭐⭐
- **Why:** Local intent understanding and ranking over history amplify LLM effectiveness
- **Tasks:** Command suggestion ranking, retrieval over command/history logs, lightweight scoring heuristics
- **Components:** `cx/llm/` routing plus local history/telemetry store for privacy-preserving behavior learning (future advanced ML optional)
- **Pattern:** LLM-guided retrieval and statistical signals over local command history without external data

#### 4. **Rendering & UI Logic** ⭐⭐
- **Why:** Complex Rich layout composition and terminal rendering logic
- **Tasks:** Building Rich layouts (panels, tables, status lines), streaming updates, responsive Typer command output
- **Components:** `cx/cli.py` and terminal UI helpers in `cx/ui/`
- **Caution:** Performance-critical for a smooth terminal experience; suggest benchmarks on large outputs

#### 5. **Protocol/IPC Layer** ⭐⭐
- **Why:** JSON-RPC serialization patterns are mechanical
- **Tasks:** Message types, daemon communication, Unix socket handling
- **Components:** `cx_daemon/` module with 4-byte length-prefixed JSON-RPC
- **Pattern:** Serde serialization with proper error handling

#### 6. **Test Coverage** ⭐⭐⭐
- **Why:** Test boilerplate is highly repetitive
- **Tasks:** Unit tests, integration tests, mock implementations, error scenarios
- **Target:** 95%+ coverage on new code
- **Pattern:** Tokio async tests with proper cleanup

#### 7. **CLI Command Implementation** ⭐⭐
- **Why:** Clap argument parsing follows predictable patterns
- **Tasks:** Adding new `cx` subcommands with help text and validation
- **Components:** CLI modules for user-facing commands
- **Pattern:** Natural language intent parsing with safety validation

### ⚠️ Areas Requiring Caution

Copilot should be more conservative in these areas:

#### Security-Critical Code ❌
- Command validation and sandboxing logic
- Privilege escalation prevention
- Authentication/authorization flows (subscription, license validation)
- **Approach:** Manual review required; suggest defensive patterns

#### Core Terminal Emulation 🔶
- VT/ANSI escape sequence parsing in CX Linux's terminal layer
- PTY handling and process management
- **Approach:** These are mature; avoid unnecessary changes

#### Cryptographic Operations ❌
- SSH key handling, certificate validation
- Encryption/decryption routines
- **Approach:** Only suggest well-vetted libraries (ring, rustls)

### 💡 Optimal Copilot Usage Patterns

**Best practices when using Copilot on this codebase:**

1. **New Agent Scaffolding:** Copilot can generate entire agent implementations from trait signatures
2. **Lua Binding Boilerplate:** Let Copilot write FFI bindings after you define the Rust signature
3. **Test Generation:** Provide function signature + expected behavior, get comprehensive tests
4. **Error Handling:** Copilot excels at adding `anyhow::Result` error propagation
5. **Async Patterns:** Trust Copilot for Tokio async/await boilerplate
6. **Documentation:** Effective at generating rustdoc comments from function signatures
7. **Refactoring:** Good at mechanical refactors (rename, extract function, update imports)

**When to ignore Copilot suggestions:**

- Security-critical validation logic (always manually review)
- Performance-critical hot paths (profile before accepting)
- Architecture decisions (require human judgment)
- Breaking changes to public APIs (consider compatibility)

## MANDATORY PRIORITIES

### 1. BSL 1.1 License Protection ⚠️
- ALL code suggestions must include BSL 1.1 license header
- NEVER suggest Apache, MIT, or other permissive licenses
- Business Source License protects competitive advantage for 4 years
- Owned by AI Venture Holdings LLC

```python
"""
Copyright (c) 2026 AI Venture Holdings LLC
Licensed under the Business Source License 1.1
You may not use this file except in compliance with the License.
"""
```

### 2. Zero Documentation OS Layer
- Suggest self-documenting natural language interfaces

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cxlinux-ai/cx-core](https://github.com/cxlinux-ai/cx-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
