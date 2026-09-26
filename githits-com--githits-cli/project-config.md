---
trigger: always_on
description: GitHits companion for the backend - provides MCP server and command-line tools for code example search.
---

# githits Agent Instructions

GitHits companion for the backend - provides MCP server and command-line tools for code example search.

We strive to produce high quality code that can easily be maintained. Focus is on long term development speed, not on quick wins.

This document contains the most important instructions that need to be kept always in context.

## General

- Use very concise output and neutral tone
- If unclear about anything or stuck, please stop and ask for clarification
- Always verify assumptions
- Don't jump into coding, plan and assess the impact first
- Read more detailed documentation when needed
- Remember your MCP tools and use them when needed

## Architecture

Philosophy: "Create architecture that is performant and easy to test"

- Focus on building structures that are performant and scalable
- Build architecture that is easy to test
- Isolate functionality into sensible small modules
- Follow single responsibility principle
- Prefer public helper modules to lots of private methods
- Use dependency injection for external services (REST client, etc.)
- Do not eagerly validate network/proxy/environment configuration while constructing command dependencies when the command has local-only or no-network paths. Defer validation until the first network operation and add regression tests for malformed env values on local paths.
- GitHits information tools advertise `readOnlyHint: true`, including internal result storage, caching, preparation, and research-thread state. Keep this policy covered by catalog tests; assess any future user-facing write tool separately.
- For MCP/agent-facing tools, avoid coupled optional flags and default-true booleans. Design schemas for real agent calls, including empty strings, empty arrays, and explicit `false` values.
- For MCP tool discovery, treat the tool name plus the first description sentence as a standalone selection surface. A verified claude.ai deferred-tool catalog rendered at most 80 characters: a sentence longer than 79 characters appeared as its first 79 plus an ellipsis, while connector descriptions and MCP server instructions did not reach selection. Lead with the natural user question and the tool's distinct job, keep that sentence within 79 characters when it must render whole, and avoid internal periods (including abbreviations, version literals, and filenames) because the observed sentence boundary is otherwise ambiguous. Keep registry counts/lists, argument mechanics, and follow-up routing after it. Also keep the first 80 raw description characters useful for clients that expose a raw prefix. Do not rely on neighboring tools for context. Add first-sentence/first-80 contract tests and run descriptor-only agent evals for description changes.
- For GraphQL/API-backed tools, treat minimal data fetching as part of the tool contract. Before adding or changing selected fields, compare the query against every consumer (text, verbose, JSON, MCP, CLI, and internal callers), use conditional fields or separate queries for mode-specific data, and add tests that assert the wire variables/selections for compact and detailed modes.

See `docs/guidelines/ARCHITECTURAL_GUIDELINES.md` for detailed planning checklist and design principles.

## Tool output UX

Human-readable tool text is an optimized product surface, not raw field
serialization. Before editing it, inspect real output and preserve existing
strengths: lead with the outcome, group related evidence, remove repetition and
scaffolding, and retain stable follow-up locators, actions, and trust facts.
Wrap free prose to the caller's width, keep formatter-authored punctuation ASCII
while preserving backend Unicode, and never make color carry meaning. When CLI
and MCP need the same information, share one formatter per tool with color and
width as inputs; keep JSON lossless for machines. A complete data dump is not
good output merely because it is complete.

## Testing

Philosophy: "If it is not tested, it is likely broken"

**Critical Rules:**

- Use `bun test` for running tests
- Use `bun run smoke:mcp` and `bun run smoke:cli` when changing MCP tools, CLI commands, shared formatters, auth/error envelopes, or MCP/CLI parity behavior. These are live-capable local suites, not the normal unit suite; they must pass unauthenticated by validating auth handling, and provide deeper coverage when authenticated. After building, also run `bun run smoke:cli:built` and `bun run smoke:mcp:built` when changing smoke launch behavior or CI product validation; these secret-free modes execute `dist/cli.js` under Node.
- Use `bun run agent:e2e` when changing MCP instructions, tool descriptions, or agent-facing tool behavior. This is a human/agent-driven qualitative eval, not a deterministic CI gate. Pick targeted workloads from `eval/agentic/README.md`; run both Claude and Codex for broad instruction changes when practical. Inspect `tool-calls.json` and `final.json` for actual tool use and the neutral answer/confidence, `metrics.json` for derived token/cost/duration/tool-call metrics, and `isolation-violations.json` for trace-validation failures, not just harness pass/fail. Treat usefulness or quality as reportable only when a later grading stage provides it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [githits-com/githits-cli](https://github.com/githits-com/githits-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
