---
trigger: always_on
description: This project builds a Zotero sidebar AI agent: a separate AI chat column inside Zotero that can read the current item, PDF text, annotations, selected text, screenshots, and future Zotero write tools through a local harness.
---

# CLAUDE.md

## Project Overview

This project builds a Zotero sidebar AI agent: a separate AI chat column inside Zotero that can read the current item, PDF text, annotations, selected text, screenshots, and future Zotero write tools through a local harness.

## Common Commands

```bash
npm test
npm run build
cp .scaffold/build/zotero-ai-sidebar.xpi zotero-ai-sidebar.xpi
cp .scaffold/build/zotero-ai-sidebar.xpi /home/qwer/.zotero/zotero/24q8duho.default/extensions/zotero-ai-sidebar@local.xpi
```

After installing the XPI, restart Zotero:

```bash
cd ~/Downloads/Zotero_linux-x86_64
./zotero
```

## Modification Guidelines

### Codex-style Agent Direction

- Keep Zotero context access model-driven: the model decides whether it needs the current item, annotations, PDF search passages, exact PDF ranges, full PDF text, screenshots, or future write tools.
- Do not add local keyword, regex, or semantic intent routing for user requests such as summarizing, explaining, continuing, selecting chapters, or creating notes.
- The local harness should expose structured tools, validate arguments, execute Zotero operations, enforce budgets, and return structured tool outputs or errors.
- Treat `maxToolIterations` as a safety fuse for the tool loop, not as task-type routing logic.
- Keep context budgets and limits centralized in `src/context/policy.ts`; avoid scattered magic numbers.
- Preserve the context ledger design: old PDF/full-text context is recorded as history metadata, not blindly replayed every turn.

### Claudian-style Chat UI Direction

- Keep the chat output clean and readable, similar to Claudian: spacious message flow, clear assistant/user separation, and compact controls near the composer.
- Render assistant output as Markdown where practical: headings, lists, code blocks, blockquotes, links, inline code, and strong text.
- Show reasoning/thinking in a clear collapsible block instead of mixing it into the final answer.
- Show Zotero tool-call traces visibly in the conversation so users can understand what local context/tools were used.
- Keep screenshot/image attachments visible in the UI and ensure they are actually sent to multimodal providers, not just displayed locally.
- Avoid UI elements that cause the right sidebar to jump while the user scrolls or selects PDF text.

### Better Notes-inspired Note Editing Direction

- Treat the note panel as a visual companion to the AI chat, not a replacement for chat behavior. The target layout remains `PDF/Reader | note panel | AI chat`.
- Prefer Zotero's official `<note-editor>`/`EditorInstance` rich editor for note editing. It already handles ProseMirror-style rendered editing, formatted text, lists, Enter, Backspace/Delete, arrow keys, selection, and autosave inside Zotero's focus system.
- Borrow Better Notes' interaction model, not its whole runtime: users edit rendered rich text directly; do not show Markdown source for clicked paragraphs; do not use block-by-block cards, `+` insertion buttons, or a separate preview toggle for normal editing.
- Keep note editing isolated from AI chat state. Opening, editing, saving, or closing the note panel must not re-render or reset chat messages, composer drafts, model selections, streaming state, or tool traces.
- Do not fight Zotero Reader/PDF keyboard handlers with aggressive global `keydown` capture around a custom `contenteditable`. If keyboard focus conflicts appear, first prefer the official note editor or iframe-level focus integration.
- Hide nonessential note-editor chrome only for layout fit, but avoid patching Zotero editor internals unless the behavior is verified against the target Zotero version.

## Architecture Notes

- Native DOM sidebar code lives mainly in `src/modules/sidebar.ts`; avoid reintroducing React UI in the Zotero pane unless crash behavior has been revalidated.
- Provider adapters live in `src/providers/`; OpenAI uses the Responses API tool loop and Anthropic uses message streaming.
- Zotero local tools live in `src/context/agent-tools.ts` and should remain structured function-call tools.
- Prompt/context formatting lives in `src/context/message-format.ts`.
- Chat history persistence lives in `src/settings/chat-history.ts`; preserve messages, context traces, thinking summaries, and image metadata.
- Harness design notes live in `docs/HARNESS_ENGINEERING.md`; update that document when changing tool-loop or context semantics.

## Development Lessons

- Zotero versions share the same user profile unless launched with `./zotero -P`; installed XPIs live under `~/.zotero/zotero/<profile>/extensions/`, not inside the Zotero binary folder.
- One XPI should support Zotero 7/8/9 when APIs are compatible; keep compatibility in `addon/manifest.json` with `strict_min_version` and `strict_max_version`.
- Keep provider config local in Zotero prefs. API keys, model IDs, Base URLs, max tokens, reasoning settings, and YOLO mode must not be hardcoded in source.
- For OpenAI Responses with `store: false`, do not rely on persisted response item IDs. Replay only the current conversation inputs, function calls, and function-call outputs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xuhan-rgb/zotero-ai-sidebar](https://github.com/xuhan-rgb/zotero-ai-sidebar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
