---
trigger: always_on
description: This file provides guidance for AI coding agents (Cursor, Copilot, Claude Code) working on this codebase.
---

# AGENTS.md

This file provides guidance for AI coding agents (Cursor, Copilot, Claude Code) working on this codebase.

## Project Overview

**@lgrammel/llama-cpp-provider** is a llama.cpp provider for the Vercel AI SDK, implementing the `LanguageModelV4` interface. It loads llama.cpp directly into Node.js memory via native C++ bindings for local LLM inference.

**Platform Support**: macOS only (Apple Silicon or Intel)

**Monorepo Structure**: This project uses pnpm workspaces with packages in `packages/` and examples in `examples/`.

## Quick Reference

| Task | Command |
|------|---------|
| Install dependencies | `pnpm install` |
| Build everything | `pnpm build` |
| Build TypeScript only | `pnpm build:ts` |
| Build native only | `pnpm build:native` |
| Run all tests once | `pnpm test:run` |
| Run unit tests | `pnpm test:unit` |
| Run integration tests | `pnpm test:integration` |
| Run E2E tests | `TEST_MODEL_PATH=./models/model.gguf pnpm test:e2e` |
| Run example | `pnpm --filter @examples/basic generate-text` |
| Clean build artifacts | `pnpm clean` |

## Setup & Installation

### Prerequisites

- **macOS** (Apple Silicon or Intel) - required
- **Node.js** >= 18.0.0
- **pnpm** >= 9.0.0
- **CMake** >= 3.15
- **Xcode Command Line Tools**

```bash
# Install Xcode Command Line Tools (if not already installed)
xcode-select --install

# Install CMake via Homebrew (if not already installed)
brew install cmake

# Install pnpm (if not already installed)
npm install -g pnpm
```

### Installation Steps

```bash
# Clone and enter the repository
git clone https://github.com/lgrammel/ai-sdk-llama-cpp.git
cd ai-sdk-llama-cpp

# Install dependencies (this also builds the native addon)
pnpm install

# Build TypeScript
pnpm build:ts
```

The `pnpm install` step automatically:
1. Detects macOS and verifies platform compatibility
2. Compiles llama.cpp as a static library with Metal support
3. Builds the native Node.js addon

### Updating And Building llama.cpp

The llama.cpp source is fetched during package installation from the `llamaCpp` config in `packages/llama-cpp-provider/package.json`. To update the pinned upstream revision:

1. Choose the upstream commit from `https://github.com/ggerganov/llama.cpp`.
2. Update `packages/llama-cpp-provider/package.json`:
   - Keep `llamaCpp.repo` pointing at the upstream repository unless intentionally changing forks.
   - Set `llamaCpp.commit` to the new commit SHA.
3. Remove the existing local checkout and native build artifacts:

```bash
pnpm --filter @lgrammel/llama-cpp-provider clean
```

4. Reinstall or build the native addon so the postinstall script fetches the new llama.cpp revision:

```bash
pnpm install
# or, if dependencies are already installed and llama.cpp is present:
pnpm build:native
```

5. Verify the TypeScript and native bindings still compile:

```bash
pnpm build:ts
pnpm test:run
```

If llama.cpp API changes break the native wrapper, update `packages/llama-cpp-provider/native/llama-wrapper.cpp`, `packages/llama-cpp-provider/native/llama-wrapper.h`, and `packages/llama-cpp-provider/native/binding.cpp` as needed, then rerun `pnpm build:native`.

## Project Structure

```
├── packages/
│   └── ai-sdk-llama-cpp/       # Main library package
│       ├── src/                # TypeScript source code
│       │   ├── index.ts        # Public exports
│       │   ├── llama-cpp-provider.ts    # Provider factory function
│       │   ├── llama-cpp-language-model.ts  # LanguageModelV4 implementation
│       │   ├── native-binding.ts   # Native module bindings
│       │   └── json-schema-to-grammar.ts   # JSON schema to GBNF grammar converter
│       ├── native/             # C++ native bindings
│       │   ├── binding.cpp     # N-API binding layer
│       │   ├── llama-wrapper.cpp   # llama.cpp wrapper implementation
│       │   └── llama-wrapper.h # llama.cpp wrapper header
│       ├── tests/              # Unit and integration tests
│       │   ├── unit/           # Unit tests (no model required)
│       │   └── integration/    # Integration tests (mocked native bindings)
│       ├── dist/               # Compiled TypeScript output (generated)
│       └── build/              # Native addon build output (generated)
├── tests/
│   └── e2e/                    # End-to-end tests (requires real model)
│       └── src/                # E2E test files
├── examples/
│   └── basic/                  # Basic usage examples
│       └── src/                # Example source files
│           ├── generate-text.ts
│           ├── stream-text.ts
│           ├── generate-text-output.ts
│           ├── chatbot.ts
│           └── embed-many.ts
├── pnpm-workspace.yaml         # Workspace configuration
└── package.json                # Root package.json with workspace scripts
```

## Testing

### Test Organization

- **Unit tests** (`packages/llama-cpp-provider/tests/unit/`): Test pure functions and class instantiation. No model or native bindings required.
- **Integration tests** (`packages/llama-cpp-provider/tests/integration/`): Test the language model class with mocked native bindings.
- **E2E tests** (`tests/e2e/`): Test actual inference with a real GGUF model file. This is a separate workspace package (`@tests/e2e`).

### Running Tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lgrammel/llama-cpp-provider](https://github.com/lgrammel/llama-cpp-provider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
