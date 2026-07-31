---
trigger: always_on
description: Notion's defining idea is that there is no document type. A page is a list of blocks, a block is a row, and a database is just a page with a `properties_schema` whose rows happen to be structured. Headings, toggles, code fences, Kanban cards, and table rows are all the same storage primitive with a different `type` string and a different `properties` JSONB payload. That uniformity is what makes "turn this paragraph into a heading" a one-column update rather than a migration between document mode
---

# Notion — Development with Claude

## Project Context

Notion's defining idea is that there is no document type. A page is a list of blocks, a block is a row, and a database is just a page with a `properties_schema` whose rows happen to be structured. Headings, toggles, code fences, Kanban cards, and table rows are all the same storage primitive with a different `type` string and a different `properties` JSONB payload. That uniformity is what makes "turn this paragraph into a heading" a one-column update rather than a migration between document models.

The consequence is that *ordering* becomes the hard problem. When every block is an independent row and two people are inserting into the same list at once, the position of a block can't be an integer — integers require renumbering siblings, and renumbering is a write that conflicts with every concurrent edit in the same list. So position is a lexicographically sortable string, and inserting between two blocks means finding a string between two strings.

The second hard problem is that collaboration needs an ordering across clients whose clocks disagree. A wall-clock timestamp will happily report that a reply happened before the message it replies to when one laptop is 400ms fast. That's what the Hybrid Logical Clock is for.

**Learning goals:** the block-as-row data model and what it buys, fractional indexing for conflict-free insertion, hybrid logical clocks for causal ordering under clock drift, and the difference between a system that *broadcasts* edits and one that genuinely *converges*.

## Architecture at a Glance (what actually runs)

| Component | Port / detail | Why this one |
|-----------|--------------|--------------|
| **API + WebSocket server** (`backend/src/index.ts`) | **3001** (`npm run dev` → `PORT=3001 tsx watch`) | Express REST plus a `ws` server on the same HTTP server; Vite proxies both `/api` and `/ws` |
| **Four workers** (`backend/src/workers/`) | `dev:worker:notification` / `:export` / `:email` / `:search` | Separate processes so a slow export can't block a notification |
| **PostgreSQL 16** | 5432 (`notion`/`notion_password`, db `notion_db`) | `users`, `workspaces`, `workspace_members`, `pages`, `blocks`, `database_views`, `database_rows`, `page_permissions`, `sessions`, `operations`, `audit_log` |
| **Valkey 7** | 6379 | Sessions and per-page presence (who is viewing, plus cursor position) |
| **RabbitMQ 3.12** | 5672 / management 15672 (`notion`/`notion_local`) | 5 queues with per-queue TTL, prefetch, retry counts, and DLQs |
| **Prometheus + Grafana** | 9090 / 3002 — `--profile observability` | Opt-in; the stack runs fine without them |

The two files that carry the ideas are `backend/src/utils/fractionalIndex.ts` and `backend/src/utils/hlc.ts`. Real-time handling is `backend/src/services/websocket.ts`; queue topology is declared once in `backend/src/shared/queue.ts` as `QUEUES`. Frontend is React 19 + TanStack Router (file-based) + Zustand + Tailwind, with block components under `frontend/src/components/blocks/` (text, heading, list, code, quote, callout, divider, toggle, plus `BlockTypeMenu` for slash commands), database views under `components/database/` (Table, Board, List), and the editor store in `stores/editor.ts`.

## Key Design Decisions

### 1. Block position is a lexicographic string, not an integer

`blocks.position VARCHAR(100)` holds strings over `a`–`z`, and `generatePosition(before, after)` returns a string that sorts strictly between its neighbors. Reading a page is `ORDER BY position` against `idx_blocks_position (page_id, position)`.

Integer positions fail here in a way that gets worse the more collaborative the document is. Inserting at index 3 of a 200-block page means `UPDATE blocks SET position = position + 1 WHERE position >= 3` — 197 row updates for one keystroke's worth of intent. Every one of those rows is now a write that conflicts with any concurrent edit to those blocks, and the update has to be broadcast to every connected client. Two users inserting into the same list simultaneously each renumber the other's target, and the results interleave incorrectly. Fractional indexing turns the same operation into a single-row insert that touches nothing else, so two concurrent inserts in the same place produce two distinct positions and both survive.

What we give up is bounded key length. Repeatedly inserting between the same two blocks makes the string grow one character at a time — pathologically, dragging one block back and forth in the same gap. `VARCHAR(100)` is the ceiling, which is generous but not infinite, and there is no rebalancing pass to compact positions when they get long. The implementation is also alphabet-only (26 symbols) rather than the wider base real libraries use, so keys grow somewhat faster than they need to.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evgenyvinnik/llm-driven-system-design](https://github.com/evgenyvinnik/llm-driven-system-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
