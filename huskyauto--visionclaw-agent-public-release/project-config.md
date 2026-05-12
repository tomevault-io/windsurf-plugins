---
trigger: always_on
description: > Companion to `replit.md`. This file is the standard "agent instructions" surface that Claude Code, Cursor, OpenAI Codex CLI, and Gemini CLI all read on entry. Replit Agent reads `replit.md`. Both files should stay in sync.
---

# AGENTS.md — Working in the VisionClaw Codebase

> Companion to `replit.md`. This file is the standard "agent instructions" surface that Claude Code, Cursor, OpenAI Codex CLI, and Gemini CLI all read on entry. Replit Agent reads `replit.md`. Both files should stay in sync.

## Identity

You are a coding agent working on **VisionClaw** — a multi-tenant agentic AI platform (Express/TypeScript backend, React/Vite frontend, Drizzle/Postgres, Stripe + Coinbase billing, Replit Auth). Owner: Robert Washburn. Public mirror: https://github.com/Huskyauto/VisionClaw-Agent-Public-Release.

## The Cardinal Discipline: Search Before Reading

**Inspired by the SocratiCode agent-instruction pattern.** Whenever you need to find or understand something in this codebase, **search before you read**.

Wrong: open files speculatively, grep blindly, read 1000-line files end-to-end.
Right: query an index first, then read only the lines the index points you to.

In this repo you have three indexes already wired up:
1. **Hybrid BM25 + pgvector knowledge search** — `server/embeddings.ts::vectorSearchKnowledge` does reciprocal-rank-fusion of full-text and vector search over the agent_knowledge table.
2. **Skill-RAG with LLM-as-judge** — `server/skill-rag.ts::enhanceRetrieval` decides whether a query needs HyDE rewrite, multi-hop decomposition, focus narrowing, or a direct answer.
3. **Tool registry** — `server/tools/index.ts` exposes 296 tools, all described in `TOOL_DEFINITIONS`. Use `getAllToolDefinitions()` rather than grepping the registry. (Authoritative counts: `docs/CURRENT_PLATFORM_TOTALS.md`.)

If the answer is not in those indexes, *then* fall back to ripgrep, *then* fall back to reading whole files. Reading whole files is the last resort, not the first.

## Hard Rules (non-negotiable)

1. **`shared/schema.ts` edits are allowed but require care.** Before changing it: inspect the live DB with `psql $DATABASE_URL` to confirm current column types and indexes; never alter a primary-key column type (serial ↔ varchar — generates destructive `ALTER TABLE`); run `npm run db:push` (or `npm run db:push --force` if the safe push refuses); verify with another `psql` query before declaring done. Sidecar files (e.g. `.local/code-health-checkpoint.json`) remain a fine alternative when the state is genuinely transient.
2. **Never `sql.raw()` user input.** Use parameterised queries via Drizzle.
3. **Never log or print secrets.** Treat `STRIPE_*`, `OPENAI_*`, `ANTHROPIC_*`, `GEMINI_*`, `MCP_API_KEY`, OAuth tokens, and webhook secrets as poison.
4. **Owner privacy:** "Bob Washburn" privately, "Robert Washburn" publicly (i.e. anything that touches the public mirror).
5. **No emojis in code, prompts, or commit messages** unless explicitly requested.
6. **Force-push pre-authorised** for both private and public repos — but only via the `Auto Git Push` and `Public Mirror Push` workflows, never directly.

## Architecture Map (entrypoints)

- `server/index.ts` — Express bootstrap, registers all routes including `registerMcpRoutes` for the MCP server at `/api/mcp/sse`.
- `server/routes.ts` — REST API routes (5,700+ lines; use search-before-reading).
- `server/mcp-server.ts` — MCP stdio + SSE transports, multi-tenant key derivation.
- `server/tools/` — 296 tool implementations, registered in `tools/index.ts`.
- `server/skill-rag.ts` — Skill-RAG pipeline (hybrid search + LLM judge + fix-skills).
- `server/code-health.ts` — Static-analysis "BS detector" (resumable since R74.13v).
- `server/seed-persona-prompts.ts` — `PERSONA_DOCS` registry for all 16 personas.
- `client/src/` — React + Vite frontend (wouter routing, shadcn/ui, TanStack Query).
- `shared/schema.ts` — Drizzle schema (edits allowed, see Hard Rules above).

## Working Loop

1. **Reproduce / understand** — search the indexes (knowledge, skill-rag, tool registry) for prior context.
2. **Plan in writing** — for non-trivial work, draft a `.local/session_plan.md` so the user can audit your intent.
3. **Make the smallest correct change** — prefer editing 5 lines in 1 file over rewriting a module.
4. **Verify** — run the affected workflow, hit the affected endpoint, or write a 10-line repro script. Do not declare "done" without proof.
5. **Document** — update `replit.md` and (if architectural) this file.
6. **Commit** — let `Auto Git Push` handle it. Public-mirror-affecting changes also trigger `Public Mirror Push`.

## When to Defer to Felix

If a task spans multiple specialist domains (engineering + writing + research, etc.), do not try to do it all yourself. Use the `delegate_task` tool to dispatch to Felix (persona id 2), the CEO. He'll fan out to the right specialists in parallel and synthesize the result.

<!-- vc-supply-chain:start -->

## Supply-Chain Discipline (R98.9 — added May 4 2026 after CVE-sweep RED week)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Huskyauto/VisionClaw-Agent-Public-Release](https://github.com/Huskyauto/VisionClaw-Agent-Public-Release) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
