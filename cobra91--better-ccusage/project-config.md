---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Monorepo Structure

This is a monorepo containing multiple packages. For package-specific guidance, refer to the individual CLAUDE.md files:

- **Main CLI Package**: @apps/better-ccusage/CLAUDE.md - Core better-ccusage CLI tool and library
- **MCP Server Package**: @apps/mcp/CLAUDE.md - MCP server implementation for better-ccusage data
- **Documentation**: @docs/CLAUDE.md - VitePress-based documentation website

## About better-ccusage

better-ccusage is a fork of the original ccusage project that addresses a critical limitation: while ccusage focuses exclusively on Claude Code usage with Anthropic models, better-ccusage extends support to external providers that use Claude Code with different providers like Anthropic, Zai, Dashscope, and many models like GLM-xx, kat-coder.

### Why the Fork?

The original ccusage project is designed specifically for Anthropic's Claude Code and doesn't account for:

- **Zai** providers that use Claude Code infrastructure with their own models
- **GLM-xx, kat-coder, kimi and Minimax** models from other AI providers
- Multi-provider environments where organizations use different AI services through Claude Code

better-ccusage maintains full compatibility with ccusage while adding comprehensive support for these additional providers and models.

### Key Differences

| Feature                      | Original ccusage | better-ccusage |
| ---------------------------- | ---------------- | -------------- |
| Anthropic Models             | ✅               | ✅             |
| Zai Provider                 | ❌               | ✅             |
| GLM\* Models                 | ❌               | ✅             |
| kat-coder                    | ❌               | ✅             |
| kimi\* Models                | ❌               | ✅             |
| MiniMax Models               | ❌               | ✅             |
| Multi-Provider Support       | ❌               | ✅             |
| Automatic Provider Detection | ❌               | ✅             |
| Cost Calculation by Provider | ❌               | ✅             |
| Original ccusage Features    | ✅               | ✅             |

### Automatic Model Detection

**No Manual Provider Prefix Management Required**

better-ccusage automatically detects and supports new AI providers without code changes. The pricing system uses intelligent fallback matching:

1. **Exact Match**: Direct lookup for model name (e.g., `"kimi-for-coding"`)
2. **Provider Prefix Match**: Suffix matching for qualified names (e.g., `"moonshot/kimi-for-coding"`)
3. **Fuzzy Match**: Scored partial matching for variations

This eliminates the need to maintain provider prefix whitelists and ensures automatic support for:

- Moonshot AI (`kimi-*` models)
- MiniMax (`MiniMax-M2`)
- Any future provider without code modifications

Each package has its own development commands, dependencies, and specific guidelines. Always check the relevant package's CLAUDE.md when working within that package directory.

### Apps Are Bundled

All projects under `apps/` ship as bundled CLIs/binaries. Treat their runtime dependencies as bundled assets: list everything in each app's `devDependencies` (never `dependencies`) so the bundler owns the runtime payload.

## Guide for lsmcp mcp

You are a professional coding agent concerned with one particular codebase. You have
access to semantic coding tools on which you rely heavily for all your work, as well as collection of memory
files containing general information about the codebase. You operate in a frugal and intelligent manner, always
keeping in mind to not read or generate content that is not needed for the task at hand.

When reading code in order to answer a user question or task, you should try reading only the necessary code.
Some tasks may require you to understand the architecture of large parts of the codebase, while for others,
it may be enough to read a small set of symbols or a single file.
Generally, you should avoid reading entire files unless it is absolutely necessary, instead relying on
intelligent step-by-step acquisition of information. Use the symbol indexing tools to efficiently navigate the codebase.

IMPORTANT: Always use the symbol indexing tools to minimize code reading:

- Use `search_symbol_from_index` to find specific symbols quickly (after indexing)
- Use `get_document_symbols` to understand file structure
- Use `find_references` to trace symbol usage
- Only read full files when absolutely necessary

You can achieve intelligent code reading by:

1. Using `index_files` to build symbol index for fast searching
2. Using `search_symbol_from_index` with filters (name, kind, file, container) to find symbols
3. Using `get_document_symbols` to understand file structure
4. Using `get_definitions`, `find_references` to trace relationships
5. Using standard file operations when needed

## Working with Symbols

Symbols are identified by their name, kind, file location, and container. Use these tools:

- `index_files` - Build symbol index for files matching pattern (e.g., '\*_/_.ts')

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cobra91/better-ccusage](https://github.com/cobra91/better-ccusage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
