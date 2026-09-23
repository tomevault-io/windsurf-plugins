---
trigger: always_on
description: Checklist for working on this Surfingkeys codebase.
---

# AGENTS.md

Checklist for working on this Surfingkeys codebase.

## Before editing
- Read the surrounding file and existing patterns first; mirror naming and style.
- Prefer reusing existing utilities in `src/content_scripts/common/` over reimplementing logic (see "Avoid duplicating utils" below).
- Verify how modules import from `./utils.js` before adding exports; many helpers are already exported.
- On ANY fix or feature, the moment two defensible behaviors compete and you cannot have both, STOP and ask which one — before writing the code. Name each option as the behavior a user would notice and what it gives up, not as an implementation. This is not about big changes: the choice is usually small and buried (which side to err on, what to do with the awkward case, whether to be strict or forgiving), which is exactly why it slips through as a decision nobody made. Do not settle it yourself and write the comment that justifies it: a plausible rationale in the diff is what stops the user noticing there was a choice at all, and they are the one who lives with it.

## Common utils to reuse (do not reinvent)
- `getTextRect(node, offset[, endNode, endOffset])` — builds a range and returns its client rects **without touching the page selection**. Useful for positioning overlays near text.
- `createElementWithContent(tag, content, attrs)` — create DOM elements with class/content.
- `llmRequest(messages, onChunk, onDone)` — async LLM request with streaming (book/release `llmResponse`). Use the shared helper instead of hand-rolling RUNTIME messaging.
- `src/content_scripts/ui/llmtools.js` — tools exposed to the LLM in the omnibar chat. Add a new tool as one declaration (`name`, `description`, `parameters`, `confirmAs`, `run`) there; `schemasFor(provider)` handles the per-provider wire format. `run(params, ctx)` gets `ctx` from the host (llmchat.js), for what only it can reach — currently `pageMarkdown()`, which resolves `{markdown, picked}`, and `highlight(query)`, which resolves `{count}` — plus the factory's own scratch space (`tabSnapshots`, `llmTabs`, `llmTabGroups`). `confirmAs(params, ctx)` is handed the same scope, since what a call would do can depend on what earlier calls did. Every call is confirmed by the user unless the tool is in `settings.llmAllowedTools` or the user has allowed it on the current origin, so `confirmAs` must state plainly what the call does; add `warn(params)` when a specific argument is the risk.
- A tool that serves text in CHUNKS (`read_page`, `read_tab`) must cut every chunk from one reading of it, and report the offset that continues it: re-reading a live page per call hands the model overlapping or skipped text with nothing to show that anything went wrong. Snapshots live for one question — llmchat.js drops its own and calls `dropSnapshots()` on a new question and after any mutating tool.
- The tabs the chat opens are the user's to clean up, so `open_url` and `read_tab` share one registry of them (`llmTabs`, `llmTabGroups` in the factory scope): a tab `open_url` creates joins ONE tab group, and a tab whose page the model has been handed IN FULL is pointed at the next URL instead of another tab being created. Reuse waits for the whole page for a reason: chunked text ends in the offset that continues it, and taking the tab away while such an offset is outstanding leaves the rest of that page reachable nowhere — not in the tab, which now holds something else, and not in the snapshot, which the mutating call drops. So a half-read tab keeps its page however many tabs that leaves, `read_tab` says which of the two states the tab is in, and reuse deletes only that tab's snapshot. Only tabs `open_url` opened are ever navigated or grouped, and the tab it opened is identified by being NEW — the tab list is read before and after — never by holding the address, since the same URL may already be open in a tab of the user's, and navigating one of those would replace a page they were reading. That registry must survive `dropSnapshots()`: a new question does not un-open a tab, and forgetting them starts a fresh pile every time.
- Reaching another tab means the background: the frontend iframe cannot address a tab it does not live in, so it is `RUNTIME('getTabMarkdown')` → `chrome.tabs.sendMessage(tabId, ..., {frameId: 0})` → the `runtime.on('getTabMarkdown')` handler in front.js. That handler must answer SYNCHRONOUSLY — runtime.js never returns true from its message listener, so the channel closes when the handler returns — and the background must use the callback form of `sendMessage` and check `chrome.runtime.lastError`, since `chrome` in Firefox has no promises. A tab with no content script in it (browser page, PDF viewer, unloaded tab) fails there, and the model has to be told what to do instead rather than handed "Could not establish connection".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brookhong/Surfingkeys](https://github.com/brookhong/Surfingkeys) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
