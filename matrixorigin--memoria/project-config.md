---
trigger: always_on
description: USE WHEN working with Memoria memory tools - storing, retrieving, correcting, purging memories
---


<!-- memoria-version: 0.2.3-->

# Memory Integration (Memoria Lite)

You have persistent memory via MCP tools. Memory survives across conversations.

## 🔴 MANDATORY: Every conversation start

Call `memory_retrieve` with a **semantic query** derived from the user's message BEFORE responding.

**Query rules:**
- ✅ Extract key concepts → "benchmark optimization", "graph retrieval bug"
- ❌ Don't use meta-queries → "all memories", "everything", "list all"

**After retrieval:**
- Results → use as reference, verify against current context
- "No relevant memories" → normal for new users, proceed
- ⚠️ warnings → inform user, offer `memory_governance`
- If results come back → use them as **reference only**. Treat retrieved memories as potentially stale or incomplete — always verify against current context before acting on them. Do NOT blindly trust memory content as ground truth.
- If "No relevant memories found" → this is normal for new users, proceed without.
- If ⚠️ health warnings appear → inform the user and offer to run `memory_governance`.

## 🔴 MANDATORY: Every conversation turn
After responding, decide if anything is worth remembering:
- User stated a preference, fact, or decision → `memory_store`
- User corrected a previously stored fact → `memory_correct` (not `memory_store` + `memory_purge`)
- You learned something new about the project/workflow → `memory_store`
- Do NOT store: greetings, trivial questions, things already in memory.

**Deduplication is automatic.** The system detects semantically similar memories and supersedes old ones. You do not need to check for duplicates before storing.

If `memory_store` or `memory_correct` response contains ⚠️, tell the user — it means the embedding service is down and retrieval will degrade to keyword-only search.

## 🟡 When NOT to store (noise reduction)
Do NOT call `memory_store` for:
- **Transient debug context**: temporary print statements, one-off test values, ephemeral error messages
- **Vague or low-confidence observations**: "might be using X", "probably prefers Y" — wait for confirmation
- **Conversation-specific context** that won't matter next session: "currently looking at line 42", "just ran the test"
- **Information already in memory**: if `memory_retrieve` already returned it, don't store again
- **Trivial or obvious facts**: "user is writing code", "user asked a question"

## 🟡 Working memory lifecycle — CRITICAL for long debug sessions
`working` memories are session-scoped temporary context. They **persist and will be retrieved in future sessions** unless explicitly cleaned up.

**When to purge working memories:**
- Task or debug session is complete → `memory_purge(topic="<task keyword>", reason="task complete")`
- You stored a working memory that turned out to be wrong → `memory_purge(memory_id="...", reason="incorrect conclusion")`
- User says "start fresh", "forget what we tried", "let's try a different approach"
- Only purge completed tasks — leave active task working memories for next session

**Promote or purge as you go:**
- Hypothesis confirmed → `memory_store` the conclusion as `semantic`, then `memory_purge` the working memory
- Hypothesis disproven → `memory_purge` the working memory immediately
- Don't wait until session end to promote — do it as soon as you know

**When a working memory contradicts current findings:**
- Do NOT keep both. Purge the stale one immediately: `memory_purge(memory_id="...", reason="superseded by new finding")`
- Then store the correct conclusion as `semantic` (not `working`) if it's a durable fact

**Anti-pattern to avoid:** Storing "current bug is X" as working memory, then later finding out it's Y, but keeping both. The stale "bug is X" memory will keep surfacing and misleading future retrieval.

## 🟡 Correction workflow (prefer correct over store+purge)
When the user contradicts a previously stored fact:
1. **Always use `memory_correct`** — not `memory_store` + `memory_purge`. This preserves the audit trail.
2. **Prefer query-based correction**: `memory_correct(query="formatting tool", new_content="Uses ruff for formatting", reason="switched from black")` — no need to look up memory_id first.
3. **Only use `memory_purge`** when the user explicitly asks to forget something entirely, not when updating a fact.

## 🟡 Deduplication before storing
Before storing a new memory, consider:
- Did `memory_retrieve` at conversation start already return a similar fact? → skip or `memory_correct` instead
- Is this a refinement of something already stored? → use `memory_correct` with the original as query
- When in doubt, `memory_search` with the key phrase first — if a match exists, correct it rather than creating a duplicate

## Tool reference

### Write tools
| Tool | When to use | Key params |
|------|-------------|------------|
| `memory_store` | User shares a fact, preference, or decision | `content`, `memory_type` (default: semantic), `session_id` (optional) |
| `memory_correct` | User says a stored memory is wrong | `memory_id` or `query` (one required), `new_content`, `reason` |
| `memory_purge` | User asks to forget something | `memory_id` (single or comma-separated batch, e.g. `"id1,id2"`) or `topic` (bulk keyword match), `reason` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matrixorigin/memoria](https://github.com/matrixorigin/memoria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
