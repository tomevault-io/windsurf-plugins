---
trigger: always_on
description: Build, run, and automate on the Aster Agents platform via its REST API — create and update agents, knowledge bases, file uploads, skills, tags, and user invitations; invoke agents and read their results; all from code. Use whenever the task involves programmatically managing or running an Aster org (asteragents.com) from Claude Code, scripts, or any external application using an Aster API key.
---


# Building on Aster Agents via the API

You are helping someone build on the [Aster Agents](https://www.asteragents.com) platform from the outside — using their organization's API key, not platform-internal access. Everything here works with a standard API key; no database or internal tooling required.

## First: make sure there's an API key

Before calling anything, confirm an API key is available — this is the one prerequisite.

1. Look for `ASTER_API_KEY` in the environment. **If it's missing, stop and walk the user through getting one:** Control Hub → **Settings → API Access** at [asteragents.com](https://www.asteragents.com) → copy the key (a 6-month, org-scoped bearer token), then `export ASTER_API_KEY="<key>"`. Never hardcode or commit it.
2. Verify it works before building anything:
   ```bash
   curl -s https://www.asteragents.com/api/agents -H "Authorization: Bearer $ASTER_API_KEY" | head
   ```
   A JSON array of agents means you're connected. A `401` means the key is missing/expired/wrong — have the user mint a fresh one (don't retry).
3. The key is scoped to **one** organization. To work across several orgs, the user needs one key per org (see "Maintain agents across many organizations").

Once the key is set and verified, follow the workflows below.

## Ground rules

1. **Base URL**: `https://www.asteragents.com/api`
2. **Auth**: `Authorization: Bearer $ASTER_API_KEY` on every request. The key comes from Control Hub → Settings → API Access, is valid for **6 months**, and is scoped to one organization. Store it in an env var; never hardcode it.
3. **Everything is org-scoped.** You can only see and touch resources in the key's org. Admin endpoints (`/admin/*`, `/agents/sync-tools`) additionally require the key's user to be `org:admin`.
4. **Consult [reference.md](reference.md)** for exact request/response shapes before calling any endpoint — field names are precise and a few are surprising (see Gotchas). Do not guess fields.
5. **The platform ships fast.** For agent-building concepts (prompting, tools, KBs, extraction schemas, models), fetch the live docs at `https://docs.asteragents.com/<path>.md` (append `.md` to any docs path — e.g. `/features/build-an-agent.md`, `/features/choosing-a-model.md`, `/promptguide.md`). Never assume the model list or tool catalog from memory.

## The core workflows

### Create an agent (the right order)

1. `GET /tools` — pull the live tool catalog. Pick tools by `name`; copy each tool's `description` + `parameters` **verbatim** into your payload. Tools needing an integration show `requiredIntegration`; they only appear once the org connects it (pass `?all=true` to see what's gated).
2. Compose the `tools` field as an **object keyed by tool name** — not an array:
   ```json
   {
     "search_knowledge_base": {
       "description": "...from /tools...",
       "parameters": { "...from /tools..." : "..." },
       "config": { "accessibleKnowledgeBaseIds": [123, 124] }
     },
     "call_agent": { "description": "...", "parameters": {}, "config": { "callableAgentIds": ["45"] } }
   }
   ```
   Per-tool `config` is where cross-references live: `accessibleKnowledgeBaseIds` (search_knowledge_base, numbers), `writableKnowledgeBaseIds` (write_to_knowledge_base, numbers), `callableAgentIds` (call_agent, **strings**), `skillIds` (load_skill).
3. `POST /agents` with `name` (the only required field), `systemPrompt`, `model` (full `provider:model-id` string — check `/features/choosing-a-model.md` for current IDs), `tools`, `conversationStarters`, `stage` (`development` until tested, then `released`), `showInChat`.
4. **Update = same endpoint with `id`.** Include `id` and the API updates only the fields you send (it's a real partial update) — *except* for two fields that reset to their defaults when omitted, see the next rule. **Always read-modify-write**: `GET /agents` → take the live object → change the field(s) you want → `POST /agents` with `id` and the unchanged `stage`/`showInChat` echoed back. This is the safe pattern, especially when scripting across many agents.
5. **Two fields silently reset on update if omitted** — burned into the platform's create-or-update path: **`stage`** snaps back to `development` and **`showInChat`** snaps back to `true`. So a bare `POST /agents { id, systemPrompt }` will un-hide hidden agents and demote released ones. Echo both fields on every update. (`tagIds`, by contrast, is left untouched when omitted — only send it when you mean to change tags.)
6. **Clone pattern**: GET the source agent, strip `id` and `emailSlug`, edit, POST.

### Maintain agents across many organizations

Each API key is scoped to **one** org — there is no cross-org admin endpoint. Fleet management = **one key per org + a loop you run client-side.** Keep a map of `{ orgName → apiKey }` (each from that org's Control Hub → Settings → API Access) and iterate.

Common patterns, all built from the endpoints above:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asterlabs-ai/build-on-aster](https://github.com/asterlabs-ai/build-on-aster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
