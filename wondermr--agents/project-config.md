---
trigger: always_on
description: IMPORTANT: These instructions OVERRIDE any default behavior. You MUST follow them exactly.
---

# CRITICAL: Agents-Core Routing Protocol

IMPORTANT: These instructions OVERRIDE any default behavior. You MUST follow them exactly.

## BLOCKING REQUIREMENT — MCP Routing

**Before answering ANY user query, you MUST call `route_and_load(query)` from the Agents-Core MCP server.** This is not optional. Do NOT answer directly — always route first.

This applies to ALL queries: coding, research, questions, documentation, debugging — everything.

### Routing Flow

1. **ALWAYS call `route_and_load(query)` FIRST** with the user's message.
   - On multi-turn: pass previous `context_hash` for delta mode.

2. **Handle the response based on status:**
   - `SUCCESS_SAMPLED` → Display `response` to the user as-is.
   - `SUCCESS` → Use `system_prompt` as context for your answer.
   - `ROUTE_REQUIRED` → **STOP all other actions.** Do NOT call any other tools in parallel.
     Pick best agent from `candidates`, call `get_agent_context(agent_name, query)` as your ONLY next action.
     Wait for its response before doing anything else.
   - `NO_CHANGE` → Context unchanged. Keep current persona.
   - `ERROR` → Answer directly (only in this case).

3. **Post-flight (after EVERY response):**
   - Respond in the same language as the user's query (auto-detect). Exceptions: code blocks, technical terms, tool/CLI output, and the mandatory footer labels `Agent`, `Skills`, `Implants`, `Rules` stay in English.
   - Append at the end (labels in English, values are canonical IDs): **Agent**: [name] · **Skills**: [skills] · **Implants**: [implants] · **Rules**: [rules]
   - Call `log_interaction(agent_name, query, response_content)` — this **always** appends an entry to `history.md` and (if Langfuse is configured) sends a generation trace. For meaningful turns (decisions, fixes, refactors, new features), additionally pass `intent=`, `action=`, `outcome=`, `files=[...]`, `tags=[...]` to curate the entry; otherwise the raw query/response are used.

4. **Repository memory (first session per repo):**
   - On the first session in an unfamiliar repo, call `describe_repo()` once to bootstrap the managed Repository Memory section in CLAUDE.md. Subsequent calls are no-ops unless `force_refresh=True` or the repo manifest changes.

## Available MCP Tools

| Tool | Purpose |
|---|---|
| `route_and_load(query)` | **MUST call first** — routes to best specialist agent |
| `get_agent_context(agent_name, query)` | Load a specific agent (after ROUTE_REQUIRED) |
| `load_implants(task_type)` | Load reasoning strategies (debugging/analysis/creative/planning) |
| `list_agents()` | List all available agents |
| `log_interaction(agent_name, query, response_content, intent?, action?, outcome?, files?, tags?)` | End-of-turn logger — **always** appends to `history.md` (deduped) and, if Langfuse is configured, records a generation trace. Curate the history entry with the optional `intent`/`action`/`outcome` params. |
| `clear_session_cache()` | Clear routing cache (use when switching contexts) |
| `describe_repo(force_refresh=False)` | One-shot repo bootstrap — writes summary into CLAUDE.md |
| `read_history(limit?, since?, query?)` | Recent entries or lazy semantic recall over history |

## Environment

- MCP server: `Agents-Core` (stdio transport, Python/FastMCP)
- Agents: `agents/[name]/system_prompt.mdc`
- Skills: `skills/skill-*.mdc`
- Implants: `implants/implant-*.mdc`
- Capabilities: `agents/capabilities/registry.yaml`
- Config: `.env` (LANGFUSE_* optional, ANTHROPIC_API_KEY for document OCR)

## Fallback (if MCP is unavailable)

If `route_and_load` fails or Agents-Core MCP is not connected:
1. Read `agents/` to find the right agent directory
2. Read `agents/[name]/system_prompt.mdc`
3. Follow the prompt manually

---

## Enrichment layers (order in every system prompt)

1. **Base agent system_prompt** — agent persona from `agents/<name>/system_prompt.mdc`.
2. **Rules** (`rules/rule-*.mdc`) — **always-on, universal, no semantic retrieval, no opt-out**. Loaded by `src/engine/rules.py`. Architectural invariant: rules apply to every agent without exception. Per-agent guidance belongs in `skills/`. Toggle via `RULES_ENABLED=0`.
3. **Skills** (`skills/skill-*.mdc`) — semantic retrieval + per-agent opt-in via `preferred_skills` / `capabilities`. Caveman-style output compression lives here as `skill-caveman-tokenomics`, opt-in via the `concise-output` capability.
4. **Capability Directives** — terse one-liners from `agents/capabilities/registry.yaml`.
5. **Implants** (`implants/implant-*.mdc`) — semantic retrieval, cognitive reasoning patterns.

## Repository Structure

```
src/
  server.py            — MCP server: route_and_load(), get_agent_context(), clear_session_cache()
  engine/
    router.py          — SemanticRouter: cache lookup, keyword matching, agent catalog
    vector_store.py    — NumpyVectorStore: numpy-based cosine similarity store
    embedder.py        — FastEmbed wrapper (model configurable via EMBEDDING_MODEL env var)
    config.py          — Thresholds, paths, env-based configuration (RULES_ENABLED, RULES_DIR)
    enrichment.py      — Prompt enrichment with rules/skills/implants by tier (lite/standard/deep)
    rules.py           — Universal always-on rules layer (no retrieval, no opt-out)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WonderMr/Agents](https://github.com/WonderMr/Agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
