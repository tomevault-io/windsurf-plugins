---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Contents

1. [Project Overview](#project-overview)
2. [Critical Rules](#critical-rules)
3. [Architecture](#architecture)
4. [Key Files](#key-files)
5. [Development Commands](#development-commands)
6. [How-To Guides](#how-to-guides)
7. [Common Patterns](#common-patterns)

---

## Project Overview

NeuroLink is a unified AI development platform shipping as both a **TypeScript SDK** and **CLI**. It wraps 13+ AI providers (OpenAI, Anthropic, Google AI Studio, Vertex, AWS Bedrock, Azure, Mistral, LiteLLM, SageMaker, Hugging Face, Ollama, OpenAI-compatible) behind a single consistent API, with full MCP support, multimodal file processing, RAG pipelines, observability, and a workflow engine.

---

## Critical Rules

These are non-negotiable. Violating them breaks the build or introduces bugs.

1. **Dynamic imports only in registry** — All providers must use dynamic imports inside factory functions in `providerRegistry.ts`. Static imports create circular dependencies.
2. **Types in canonical location** — All type definitions go in `src/lib/types/`. Never create type files inside feature subdirectories.
3. **Gemini tools + JSON schema are mutually exclusive** — Google AI Studio and Vertex AI cannot use tools and `structuredOutput` with a JSON schema simultaneously. It's an API limitation. Design workflows to use one or the other.
4. **CLI ≠ SDK** — CLI can use manual MCP connections; the SDK cannot. Keep concerns separate.
5. **Backward compatibility** — Public SDK API must not break existing callers.
6. **`formatProviderError` must return, never throw** — Any provider error formatter must return the error object, not throw it.
7. **Zero `interface` — always use `type`** — Never use `interface`. Always use `type X = { ... }`. The only exception is `declare global { interface Window { ... } }` which TypeScript requires for declaration merging. Use intersection (`&`) instead of `extends`.
8. **No "Types" suffix in type filenames** — Files inside `src/lib/types/` must not contain "Types" or "Type" in their name. The folder IS the types folder — `mcp.ts` not `mcpTypes.ts`, `auth.ts` not `authTypes.ts`.
9. **Unique type names across all files** — Every exported type name must be globally unique across all files in `src/lib/types/`. Use domain prefixes to disambiguate:
   - Client SDK types: `Client*` prefix (e.g., `ClientAuthConfig`, `ClientToolInfo`, `ClientStreamResult`)
   - CLI types: `Cli*` prefix (e.g., `CliGenerateResult`, `CliStreamChunk`)
   - Server types: `Server*` prefix (e.g., `ServerAuthConfig`)
   - Stream types: `Stream*` prefix (e.g., `StreamToolCall`, `StreamToolResult`)
   - Processor types: `Processor*` prefix (e.g., `ProcessorRetryConfig`)
   - Workflow judge types: `Judge*` prefix (e.g., `JudgeScoreResult`)
10. **Barrel uses `export *` only** — `src/lib/types/index.ts` must only contain `export * from "./file.js"` lines. No selective exports (`export type { X, Y }`), no aliases (`X as Y`). If adding `export *` causes a name collision, rename the type at the source with a domain prefix per rule 9.
11. **No local `types/` directories** — There must be no `types/` directory anywhere except `src/lib/types/`. No `src/lib/observability/types/`, no `src/lib/workflow/core/types/`, etc. Move those types into the canonical `src/lib/types/` folder.
12. **No type re-exports from non-type files** — Files outside `src/lib/types/` must not re-export types (`export type { X } from`). Consumers should import types from `src/lib/types/` directly. Module `index.ts` files should only re-export runtime values (classes, functions, constants), never types.

13. **Barrel-only imports for internal types** — Code outside `src/lib/types/` must import internal types from the barrel (`../types/index.js` or `../types`), never from specific type files (`../types/rag.js`, `../types/mcp.js`). External library types (`zod`, `@ai-sdk/provider`, etc.) can be imported normally. Files inside `src/lib/types/` are exempt (they import from each other).

**Enforcement:** All rules (2, 7-13) are enforced by custom ESLint rules in `eslint-rules/`. Run `pnpm run lint` (or the pre-commit hook) — no shell scripts, no regex heuristics, everything AST-based.

| Rule     | ESLint rule                              |
| -------- | ---------------------------------------- |
| 2        | `neurolink/no-local-type-alias`          |
| 7        | `neurolink/no-interface`                 |
| 8        | `neurolink/no-types-suffix-filename`     |
| 9        | `neurolink/unique-type-names`            |
| 10       | `neurolink/types-barrel-exports-only`    |
| 11 & 11b | `neurolink/no-local-types-folder`        |
| 12       | `neurolink/no-type-export-outside-types` |
| 13       | `neurolink/barrel-type-imports`          |

---

## Architecture

### Pattern: Factory + Registry

Every extensible system (providers, processors, chunkers, rerankers) follows the same pattern:

```
Factory  →  creates instances
Registry →  holds factory functions (via dynamic import)
```

- `ProviderFactory` + `ProviderRegistry` — AI providers
- `ProcessorRegistry` — file/multimodal processors
- `ChunkerFactory` + `ChunkerRegistry` — RAG chunking strategies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juspay/neurolink](https://github.com/juspay/neurolink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
