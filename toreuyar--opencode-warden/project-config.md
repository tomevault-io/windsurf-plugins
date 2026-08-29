---
trigger: always_on
description: > **CRITICAL — READ THIS BEFORE ANYTHING ELSE**
---

# AGENTS.md — AI Coding Agent Guide

> **CRITICAL — READ THIS BEFORE ANYTHING ELSE**
>
> **OpenCode Warden is a security plugin. It is FAIL-CLOSED. PERIOD.**
>
> If the LLM is unreachable, inoperable, misconfigured, timed out, or returns garbage — **every LLM-evaluated operation MUST be blocked and every sanitized output MUST be withheld.** No exceptions. No fallbacks. No graceful degradation.
>
> **Graceful degradation is NOT an option for this project.** This is a watchdog, a guard gate, a lock. If the evaluator cannot evaluate, it blocks. If the sanitizer cannot sanitize, it withholds. If you are ever tempted to suggest fail-open behavior, "safe defaults", "allow on failure", "regex-only fallback", or any variant of "keep working without the LLM" — **STOP. You are wrong.** That behavior defeats the entire purpose of a security plugin. It is the equivalent of disabling the lock when the key doesn't fit.
>
> **Never suggest fail-open behavior for any component of this plugin.** Not in code. Not in documentation. Not in comments. Not in tests. Not in configuration defaults. If you see existing fail-open language anywhere, it is a bug — fix it or flag it immediately.
>
> **This does NOT apply when LLM is explicitly disabled** (`llm.enabled: false`). In that case, LLM components are `null` — no LLM calls are attempted, nothing is blocked due to LLM absence. The plugin runs in deterministic-only mode (regex + file path rules). Fail-closed only applies when the LLM is **enabled** but **unreachable at runtime**.

This document is written for AI coding assistants (Claude, GPT, Copilot, Cursor, etc.) that will be working on the OpenCode Warden codebase. It explains what this project is, how it's structured, what conventions to follow, and how to make changes correctly.

## What This Project Is

OpenCode Warden is a **plugin** for [OpenCode](https://opencode.ai), an AI-powered coding assistant. The plugin acts as a security layer that intercepts every tool call the AI makes — reading files, running shell commands, editing code — and:

1. **Blocks** access to sensitive files (`.env`, `*.pem`, `*.key`, etc.)
2. **Scans and redacts** secrets in tool inputs and outputs using 74 regex patterns
3. **Evaluates safety** of tool calls using an optional LLM (10 risk dimensions)
4. **Sanitizes** environment variables before they reach shell commands
5. **Logs** all security events to an audit trail

The plugin is loaded by the OpenCode runtime and registers **hooks** that fire at specific points in the tool execution lifecycle. It does NOT have a UI of its own — it communicates through OpenCode's toast notifications, log panel, and permission system.

## Tech Stack

| Component | Technology |
|---|---|
| Runtime | [Bun](https://bun.sh) |
| Language | TypeScript (strict mode) |
| Test Runner | Bun's built-in (`bun:test`) |
| Schema Validation | [Zod](https://zod.dev) |
| Plugin SDK | `@opencode-ai/plugin` |
| Module System | ESM (`import`/`export`) |
| Build | `bun build src/index.ts --outdir dist --target bun` |

## Commands

```bash
bun install          # Install dependencies
bun test             # Run all tests (157 tests)
bun run typecheck    # TypeScript type checking (tsc --noEmit)
bun run build        # Build for distribution
```

**Always run `bun run typecheck` and `bun test` after making changes.** Both must pass with zero errors.

## Project Structure

```
src/
├── index.ts                      # Plugin entry point — wires everything together
├── types.ts                      # ALL type definitions (single file)
│
├── config/
│   ├── index.ts                  # Config loading: defaults → global → project (deep merge)
│   ├── defaults.ts               # DEFAULT_CONFIG with every field filled in
│   └── schema.ts                 # Zod schemas for user config validation
│
├── detection/
│   ├── index.ts                  # createDetectionEngine() factory
│   ├── engine.ts                 # DetectionEngine class (scan, overlap resolution)
│   ├── redactor.ts               # Simple redaction helpers
│   └── patterns/
│       ├── index.ts              # Aggregates all patterns, applies category/disabled filters
│       ├── api-keys.ts           # 20 API key patterns
│       ├── credentials.ts        # 7 credential patterns
│       ├── private-keys.ts       # 7 private key patterns
│       ├── docker.ts             # 4 Docker patterns
│       ├── kubernetes.ts         # 6 Kubernetes patterns
│       ├── cloud.ts              # 16 cloud provider patterns
│       └── pii.ts                # 7 PII patterns (email, phone, SSN, credit card, IP)
│
├── hooks/
│   ├── input-sanitizer.ts        # tool.execute.before — blocks files, scans args, safety eval
│   ├── output-redactor.ts        # tool.execute.after — redacts secrets in output
│   ├── permission-handler.ts     # permission.ask — integrates with OpenCode permissions
│   ├── env-sanitizer.ts          # shell.env — strips secrets from environment variables
│   ├── compaction-context.ts     # session.compacting — injects security context
│   └── security-policy.ts        # Builds security policy markdown for AI context
│
├── llm/
│   ├── index.ts                  # LlmSanitizer class (output sanitization via LLM)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [toreuyar/opencode-warden](https://github.com/toreuyar/opencode-warden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
