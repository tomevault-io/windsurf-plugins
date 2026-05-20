---
trigger: always_on
description: - If the `Sufficit AI Memory` MCP server is available in VS Code, run memory recall before broad repository exploration, repo mapping, or speculative architecture reading.
---

# QuePasa AI Agent Instructions

## AI Memory First - EXECUTE BEFORE BROAD REPOSITORY EXPLORATION

- If the `Sufficit AI Memory` MCP server is available in VS Code, run memory recall before broad repository exploration, repo mapping, or speculative architecture reading.
- Build the first recall query from the current repository, branch, active file, user goal, and the most stable local anchors available: symbol, failing command, failing test, or concrete error.
- Use compact retrieval order: `memory_search` → `memory_timeline` → `memory_get_observations`.
- Save structured memory with `memory_save` using the schema in `schemas/vscode-memory-anchor-checkpoint.schema.json`.
- Save one `task-anchor` after initial routing and a `task-checkpoint` at major milestones: first local hypothesis, first substantive edit, focused validation, deploy/runtime verification, and handoff/summary boundaries.
- Persist compact task state only. Never dump raw chat transcripts, secrets, or oversized logs into memory.
- If the MCP server is unavailable or the current workspace is not configured for it, continue normally and do not block the task.

## AI Agent Startup Checklist - EXECUTE FIRST

**At the start of EVERY conversation or when resuming after summary:**

1. **Run the `AI Memory First` workflow above** whenever the `Sufficit AI Memory` MCP server is available
2. **Review current task context files** if available in the current directory
3. **Read relevant project documentation** based on task type
4. **Read `/.github/copilot-chat-vocabulary.md`** and apply its normalization rules in the full conversation

## Communication Guidelines
- Response Language: Always respond in the same language as the user's query (Portuguese for Portuguese queries, English for English queries)
- Code and Comments: All code, comments, documentation, and technical content must be in English
- Consistency: Maintain language consistency within each response type

## Architecture
- Go-based WhatsApp bot platform with HTTP APIs
- Whatsmeow library integration (go.mau.fi/whatsmeow)
- Message flow: WhatsmeowHandlers → QPWhatsappHandlers → Webhook/RabbitMQ/Dispatching
- Multi-layered APIs: v1, v2, v3 + non-versioned (latest) routes
- Modular packages in src/ subdirectories
- Each directory in `src/apps/<slug>` is an independent frontend app for the same QuePasa API; keep apps isolated by slug with no implicit aliasing, fallback, or semantic coupling between apps

## Core Components
1. Connection: whatsmeow/WhatsmeowConnection
2. Handlers: WhatsmeowHandlers → QPWhatsappHandlers
3. Processing: Cache → Trigger → Webhooks/RabbitMQ
4. API: REST endpoints in api/ with api_handlers+*Controller.go pattern
5. Server: QpWhatsappServer coordinates all operations

## Identifiers
- JId: types.JID from whatsmeow
- WId: String format
- LId: Local identifier (default, hides phone numbers)
  - **IMPORTANT:** LID was created by WhatsApp specifically to hide phone numbers for privacy
  - LIDs NEVER contain phone numbers - they are opaque identifiers
  - Phone number mapping must be obtained from whatsmeow database (whatsmeow_lid_map table)
  - Not all LIDs have phone number mappings available - this is expected behavior
  - Format: `<opaque_id>@lid` (e.g., `121281638842371@lid`)
  - Do NOT attempt to extract phone numbers from LID strings

## Contact Name Priority
ContactInfo fields priority (use `ExtractContactName()`):
1. **FullName** - User's saved name for contact (highest priority - most personal)
2. **BusinessName** - Business account name (WhatsApp Business)
3. **PushName** - Contact's public name (self-chosen)
4. **FirstName** - Generic first name (lowest priority)

## Software Documentation Structure
- README.md: Human-readable documentation
- /docs: Canonical folder for software documentation in this repository
- copilot-instructions.md: Global AI agent guidelines (this file)
- Root `AGENTS.md`: branch-scoped instructions for feature/custom branches only (must not exist on `develop`/`main`/`master`)

## Root AGENTS.md (Task Tracking)
- Purpose: track the current task running in the active custom branch.
- Scope: only the task for that branch; do not mix content from other branches.
- Required sections in `AGENTS.md`:
  - task objective
  - mandatory checklist
  - current status
  - next steps
  - immutable constraints discovered during execution
- Update cadence: update `AGENTS.md` on each relevant step and whenever new vital information is discovered.
- Conversation memory rule: if a detail is critical to avoid future loss in summaries or continuation, persist it in `AGENTS.md`.
- Branch isolation rule: `AGENTS.md` must not be merged into `develop`/`main`/`master` and must not be propagated across unrelated branches.

## Instruction Documents (AI-Only)
- Location: `/.github/instructions/*.instructions.md`.
- Instruction documents are separate from software documentation.
- Use them only as AI operating instructions.
- Do not duplicate or reference specific instruction files in other sections of this document.
- Tags are defined in the instruction filename, before `.instructions.md`.
- Use 4 to 6 hyphen-separated tags in the filename.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nocodeleaks/quepasa](https://github.com/nocodeleaks/quepasa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
