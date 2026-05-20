---
trigger: always_on
description: > **This document is the highest-priority specification for this project. All developers (including AI Agents) MUST comply unconditionally.**
---

# opencode-acp Development Specification

> **This document is the highest-priority specification for this project. All developers (including AI Agents) MUST comply unconditionally.**

---

## 1. Project Overview

### 1.1 What Is ACP

**Active Context Pruning (ACP)** is an [OpenCode](https://opencode.ai) plugin that implements model-driven context management. Instead of passively truncating context at a hard limit, ACP exposes a `compress` tool to the AI model, letting it decide **when** and **what** to compress into high-fidelity summaries.

ACP is a hardened fork of [DCP](https://github.com/Tarquinen/opencode-dynamic-context-pruning) with **35 bug fixes**, including state persistence, token reporting, GC deactivation, 268x logger speedup, and auto-recovery for reversed boundaries.

### 1.2 Tech Stack

| Category | Technology |
|----------|-----------|
| Language | TypeScript (strict, ESM) |
| Runtime | Node.js |
| Build | `tsup` (bundling) + `tsc --emitDeclarationOnly` (types) |
| Test Runner | Node.js built-in: `node --import tsx --test tests/*.test.ts` |
| Package Manager | npm |
| Linting/Formatting | Prettier |
| Plugin SDK | `@opencode-ai/plugin` >=1.4.3, `@opencode-ai/sdk` >=1.4.3 |
| Tokenizer | `@anthropic-ai/tokenizer` |
| Config Parsing | `jsonc-parser` |
| Validation | `zod` |

### 1.3 Repository Info

| Field | Value |
|-------|-------|
| npm package | `opencode-acp` |
| Current version | 1.1.0 |
| GitHub | https://github.com/ranxianglei/opencode-acp |
| License | AGPL-3.0-or-later |
| Author | ranxianglei |

---

## 2. Architecture

### 2.1 Module Map

```
opencode-acp/
├── index.ts                          # Plugin entry point — wires hooks, tools, commands, config
├── lib/
│   ├── hooks.ts                      # Plugin hook handlers (system prompt, message transform, command, event, text-complete)
│   ├── config.ts                     # Three-layer config: global → config-dir → project, with DCP migration
│   ├── logger.ts                     # Structured logging (logs/acp/)
│   ├── auth.ts                       # Plugin authentication
│   ├── token-utils.ts                # Token counting utilities
│   ├── message-ids.ts                # Message ID mapping (raw ↔ mNNNNNN refs)
│   ├── compress-permission.ts        # Permission management for compress tool
│   ├── protected-patterns.ts         # File pattern protection logic
│   ├── host-permissions.ts           # Host-based permission system
│   │
│   ├── compress/                     # Compression subsystem
│   │   ├── pipeline.ts               # Shared prepare/finalize pipeline for both modes
│   │   ├── range.ts                  # Range-mode compress tool (contiguous spans → block summaries)
│   │   ├── message.ts                # Message-mode compress tool (individual message summaries)
│   │   ├── search.ts                 # Boundary resolution: maps IDs → message indices
│   │   ├── state.ts                  # Block allocation, state mutation, wrapping
│   │   ├── message-utils.ts          # Message-level utilities for compression
│   │   ├── protected-content.ts      # Protected content injection into summaries
│   │   ├── range-utils.ts            # Range-level utility functions
│   │   ├── timing.ts                 # Compression timing tracking
│   │   ├── types.ts                  # Shared type definitions (ToolContext, BoundaryReference, etc.)
│   │   └── index.ts                  # Barrel export
│   │
│   ├── messages/                     # Message processing pipeline
│   │   ├── inject/
│   │   │   ├── inject.ts             # Nudge injection (context-limit, turn, iteration) + message ID injection
│   │   │   └── utils.ts              # Anchor management, context usage calculation, budget computation
│   │   ├── prune.ts                  # Replace compressed ranges with summaries, strip tool outputs
│   │   ├── sync.ts                   # Sync compression blocks with actual messages (deactivate orphans)
│   │   ├── priority.ts               # Message priority computation
│   │   ├── query.ts                  # Message query utilities
│   │   ├── shape.ts                  # Message shape analysis
│   │   ├── reasoning-strip.ts        # Strip reasoning tokens from messages
│   │   ├── utils.ts                  # General message utilities
│   │   └── index.ts                  # Barrel export
│   │
│   ├── prompts/                      # Prompt system
│   │   ├── index.ts                  # System prompt renderer (base + extensions)
│   │   ├── store.ts                  # 6 editable prompts, file-based overrides at 3 levels
│   │   ├── system.ts                 # Base system prompt template
│   │   ├── compress-message.ts       # Message-mode compress prompt
│   │   ├── compress-range.ts         # Range-mode compress prompt
│   │   ├── context-limit-nudge.ts    # Context limit nudge template
│   │   ├── turn-nudge.ts             # Turn nudge template
│   │   ├── iteration-nudge.ts        # Iteration nudge template
│   │   └── extensions/
│   │       └── nudge.ts              # Block aging warnings + message priority guidance
│   │
│   ├── state/                        # State management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ranxianglei/opencode-acp](https://github.com/ranxianglei/opencode-acp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
