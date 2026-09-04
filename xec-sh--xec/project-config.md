---
trigger: always_on
description: Xec is a modern, type-safe command execution system built with TypeScript. It provides a unified API for executing commands across local, SSH, Docker, and Kubernetes environments with a syntax inspired by Google's zx.
---

# Xec Universal Command Execution System - Development Guide

## 🎯 Project Mission
Xec is a modern, type-safe command execution system built with TypeScript. It provides a unified API for executing commands across local, SSH, Docker, and Kubernetes environments with a syntax inspired by Google's zx.

## ⚠️ CRITICAL: Zero-Tolerance Reliability Policy

**This is a HIGH-RELIABILITY SYSTEM**. Commands executed through Xec may affect critical infrastructure. We DO NOT make assumptions, approximations, or partial implementations:

- **100% Runtime Compatibility**: All code MUST work identically on Node.js, Bun, and Deno
- **100% Platform Compatibility**: Linux, macOS and Windows. The unit tier runs
  on all three in CI. Xec's own behaviour is identical everywhere — escaping,
  path composition, glob separators, line endings, process termination; what a
  *shell* understands is not, and that limit is documented rather than papered
  over.
- **100% Type Safety**: No `any` types in public APIs, 100% type coverage
- **Zero Warnings**: No TypeScript warnings, no linter warnings, no deprecation warnings
- **Minimal Core Dependencies**: Core depends on `ssh2` only, loaded lazily when an
  SSH target is used. Every addition must be justified in review.
- **No "Good Enough"**: If it's not perfect, it's not ready
- **No Workarounds**: Fix the root cause, not the symptom
- **No Assumptions**: Test everything, verify everything, prove everything
- **No Silent Failures**: Every error must be caught, logged, and handled appropriately

**REMEMBER**: This framework executes commands that could affect production systems. A single bug could have catastrophic consequences. There is NO room for error.

## 📁 Monorepo Structure
```
xec/
├── apps/
│   └── xec/          # CLI application (@xec-sh/cli)
│── website/          # Documentation site (Docusaurus)
├── packages/
│   ├── core/         # Shell execution engine (@xec-sh/core)
│   ├── ops/          # DevOps operations library (@xec-sh/ops)
│   ├── kit/          # TUI/CLI components (@xec-sh/kit)
│   ├── loader/       # Script loading & modules (@xec-sh/loader)
│   └── testing/      # Shared test utilities (@xec-sh/testing)
├── experiments/      # Experimental features and prototypes
├── turbo.json        # Build orchestration
└── CLAUDE.md         # This file - project specification
```

## 🏗 Architecture

### Dependency Graph
```
┌──────────────────┐
│  @xec-sh/cli     │  Thin CLI wrapper
└────────┬─────────┘
         │
┌────────▼─────────┐
│  @xec-sh/ops     │  DevOps operations library
└────────┬─────────┘  (deploy, pipeline, workflow, health, discovery)
         │
┌────────▼─────────┐
│  @xec-sh/core    │  Shell execution engine
└────────┬─────────┘  (SSH, Docker, K8s adapters)
         │
┌────────┴────┬────────────┬──────────────┐
│ @xec-sh/kit │ @xec-sh/   │ @xec-sh/     │
│ TUI/CLI     │  loader    │  testing     │
│ components  │ Scripts &  │ Test utils   │
└─────────────┘ modules    └──────────────┘
                └──────────┘
```

### Core Components

#### @xec-sh/core
```
src/
├── core/                    # Core engine components
│   ├── execution-engine.ts  # Main execution orchestrator
│   ├── command.ts          # Re-exports types from types/command.js
│   ├── result.ts           # Result implementation & re-exports from types/result.js
│   ├── error.ts            # Error class implementations
│   ├── enhanced-error.ts   # Enhanced error classes with suggestions
│   ├── process-context.ts  # Process context & promise building
│   ├── process-output.ts   # ProcessOutput implementation
│   └── pipe-implementation.ts # Pipe functionality
├── types/                  # TypeScript type definitions
│   ├── command.ts          # Command, adapter options, stream types
│   ├── process.ts          # ProcessPromise, PipeTarget, PipeOptions
│   ├── result.ts           # ExecutionResult interface
│   ├── error.ts            # Error context & suggestion types
│   ├── execution.ts        # ExecutionConfig, ExecutionEngineConfig, Docker options
│   ├── engine.ts           # CallableExecutionEngine type
│   ├── events.ts           # Event system types
│   └── disposable.ts       # Disposable interface
├── adapters/               # Environment adapters
│   ├── base-adapter.ts     # Abstract base & shared logic
│   ├── local/              # Local process execution
│   │   ├── index.ts        # Local adapter implementation
│   │   └── runtime-detect.ts # Runtime detection utilities
│   ├── ssh/                # SSH with connection pooling
│   │   ├── index.ts        # SSH adapter implementation
│   │   ├── ssh.ts          # SSH connection management
│   │   ├── ssh-key-validator.ts # SSH key validation
│   │   ├── secure-password.ts # Secure password handling
│   │   └── connection-pool-metrics.ts # Connection pool metrics
│   ├── docker/             # Docker container execution
│   │   ├── index.ts        # Docker adapter implementation
│   │   ├── docker-api.ts   # Docker client wrapper
│   │   └── docker-fluent-api.ts # Fluent API for Docker
│   ├── kubernetes/         # Kubernetes pod execution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xec-sh/xec](https://github.com/xec-sh/xec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
