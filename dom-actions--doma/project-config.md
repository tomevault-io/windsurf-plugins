---
trigger: always_on
description: Guidance for coding agents helping humans develop DomA in this repository.
---

# AGENTS.md

Guidance for coding agents helping humans develop DomA in this repository.

Read this file together with [`CONTRIBUTING.md`](./CONTRIBUTING.md). Prefer project notes and the current tree over inventing parallel architecture.

Human-facing docs: [`README.md`](./README.md), [`CONTRIBUTING.md`](./CONTRIBUTING.md), [`LICENSE`](./LICENSE) (PolyForm Shield License 1.0.0).

Two engineering logs capture hard-won behavior (read before touching the related areas):

- [`src/services/chat/conversationSwitch.record.md`](./src/services/chat/conversationSwitch.record.md)
- [`src/services/chat/disabledFeatures.record.md`](./src/services/chat/disabledFeatures.record.md)

---

## 1. What you are working on

DomA is a **browser extension** AI agent: side-panel chat → model chooses tools/skills → automate the web (browse, click, fill, collect).

Default desktop build output: `dist/desktop/open`. Users bring their own LLM API keys (BYOK).

```bash
npm install
npm run dev      # watch build (UI + service worker)
npm run build    # production → dist/desktop/open
npm run clean    # remove dist/
```

After changes: **reload the unpacked extension**, then exercise the flow on a real page. Compile success alone is not enough.

Implement against the **current workspace sources**. Do not assume modules exist if they are not present in this tree.

---

## 2. Where to change things

| Goal | File |
| --- | --- |
| Global persona / System Prompt / capability claims | [`src/services/chat/llm/llmTypes.ts`](./src/services/chat/llm/llmTypes.ts) |
| Tool schemas shown to the model | [`src/services/chat/llm/toolsDefinition.ts`](./src/services/chat/llm/toolsDefinition.ts) |
| Tool implementations | [`src/services/chat/browserTools.ts`](./src/services/chat/browserTools.ts) |
| New model provider | Subclass [`llmService.ts`](./src/services/chat/llm/llmService.ts); wire in [`entry.ts`](./src/services/chat/llm/entry.ts) (and [`llmPresets.ts`](./src/services/chat/llm/llmPresets.ts) if needed) |
| Side panel / session orchestration | [`ChatPanel.vue`](./src/components/chat/ChatPanel.vue) — high risk; see below |
| Assistant message presentation | [`AssistantMessageContent.vue`](./src/components/chat/AssistantMessageContent.vue) |

Often needed next: `llm/contextManager.ts`, `chatStorage.ts`, `skills/`, `slashCommands.ts`, MCP under `src/services/mcp*` and chat MCP helpers.

Rough turn flow:

1. UI / composer accepts the user message (`ChatPanel` and composer components)
2. LLM entry picks provider/model (`entry.ts` + `*Service`)
3. Model may call tools from `toolsDefinition.ts`
4. Runtime runs handlers in `browserTools.ts`
5. Results render back through chat UI

---

## 3. DomA-specific engineering lessons

These are the parts that matter more than a fresh directory tour.

### 3.1 Tool changes are a three-file transaction

When adding or changing a tool, update in the same change set:

1. `toolsDefinition.ts` — model-visible name, description, parameters  
2. `browserTools.ts` — implementation and `TOOL_HANDLERS` registration  
3. `llmTypes.ts` — only if the System Prompt claims that capability  

Half-finished PRs that touch only one of these are a recurring failure mode.

### 3.2 Conversation switching in `ChatPanel.vue` is a red line

`ChatPanel.vue` is large and owns conversation binding, tab activation, `loadConversation`, timers, groups, and streaming persistence.

**Before editing tab switch / `loadConversation` / `onActivated` / panel conversation id:** read  
[`conversationSwitch.record.md`](./src/services/chat/conversationSwitch.record.md).

Lessons already paid for:

- During streaming, **`liveMessageDrafts` is authoritative**, not the panel `messages` array. Switching tabs replaces `messages`; naive `addMessage(short chunk) + put` will overwrite IndexedDB with truncated text.
- `upsertAssistantMessage` should update the draft; sync the same object reference into the panel when bound.
- `loadConversation` must flush drafts when leaving and `mergeLiveDraftsIntoMessages` when entering.
- Broad “loading pin” early-returns used to block tab switches and caused stuck UI with MCP / scheduled flows. **Do not reintroduce** wide loading skips casually. Keep the narrower `chat/suppressTabBind` path (e.g. brief-activate for screenshots).
- Scheduled conversations (`scheduled === true`) stay out of normal history; cleanup group/storage/context on finish as documented in the record.

Rule: **minimal, explainable patches only** on the session state machine. No drive-by refactors.

### 3.3 Some features are intentionally disabled, not missing

Read [`disabledFeatures.record.md`](./src/services/chat/disabledFeatures.record.md).

Examples:

- `browser_skill_background_browse` — unstable screenshots / injection on inactive or collapsed-group tabs; removed from tool list and handler registration.
- `ToolThinking` UI — unwired with that tool.

When restoring, follow that file’s checklist. **Do not delete large commented or retained implementations** marked as kept for reconnect. If you find big commented blocks, check the disabled-features log before “cleaning them up.”

### 3.4 Context limits and interaction modes have hard contracts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dom-actions/doma](https://github.com/dom-actions/doma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
