---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file browser-console script (`claude-chat-exporter.js`) that exports a claude.ai conversation to a markdown file. It runs by pasting the whole file into the browser devtools console while a conversation is open (a one-click bookmarklet in `docs/index.html` wraps the same script). There is **no build system, package manager, test suite, or lint config** — the whole repo is the script, `README.md`, `docs/index.html`, this file, `LICENSE`, `.github/FUNDING.yml`, and `.claude/skills/` — none of which any code reads.

To test a change: `node --check claude-chat-exporter.js` for syntax, then paste the edited script into the console on a real claude.ai conversation and observe the on-page status indicator and downloaded `.md` file.

**`main` is the release channel.** No tags, no releases, no build artifact: the install page fetches `claude-chat-exporter.js` straight from `main`, and the update check compares against `main`. GitHub Pages serves `docs/` from `main`. **Merging to `main` is publishing** — every script change immediately becomes what new installs get and flips existing bookmarklets to "update available". The script's *hash* is the version signal; there is no version number anywhere.

## Core design decision

The script **reads nothing from Claude's rendered page**. It takes the conversation directly from claude.ai's own **internal API**, whose response already contains each message's **source markdown** (tables, math, code, etc.) — so fidelity is byte-perfect with zero conversion logic. **Reject any change that reads the conversation from the page** — DOM scraping, driving Claude's own buttons, intercepting the clipboard, or parsing HTML into markdown. All of it works against the point of the project.

**The script contains zero CSS selectors** — no `querySelector`, no `aria-label` matching, no class names — which is what makes Claude's markup and its localisation unable to break it. It does legitimately touch the DOM, so don't "fix" that: `document.createElement` and `document.body` for the download link and status box, `document.cookie` for `lastActiveOrg`, `window.location` for the conversation id. Reading *Claude's* markup is the prohibited part, not touching the DOM at all.

## How the export works

The whole flow lives in `setupClaudeExporter()` (one closure, invoked at the bottom) and is a straight pipeline — one fetch, transform, download:

1. **`fetchConversationData()`** — same-origin GET to the internal endpoint
   `/api/organizations/{orgId}/chat_conversations/{conversationId}?tree=true&rendering_mode=messages&render_all_tools=true`,
   authenticated by the user's existing session cookie. `conversationId` comes from the URL path; `orgId` from the `lastActiveOrg` cookie.

2. **`getOrderedMessages(data)`** — turns the response into an ordered `[{ sender, text, created_at, truncated, stop_reason }]`:
   - **Ordering** follows the *current branch*: walk from `data.current_leaf_message_uuid` up the `parent_message_uuid` chain and reverse (so a regenerated response exports the path actually on screen). Falls back to sorting `chat_messages` by `index`.
   - **Content**: each message's `content` is an array of typed blocks (`text`, `thinking`, `tool_use`, `tool_result`). Walk them **in order** — emit `text` blocks and `tool_use` blocks (via `renderToolUse`), skip `thinking`/`tool_result` — so text and special elements stay interleaved. Messages that end up empty are dropped (the API also returns hidden/system messages the UI never shows). `renderToolUse` renders only the three content-bearing tools (`artifacts`→`content`, `create_file`→`file_text`, `visualize:show_widget`→`widget_code`) as titled fenced code blocks with an API-derived kind label (`Artifact:` / `File:` / `Widget:`, no emoji); **every other tool** (web search, bash, file view/edit, display widgets, unknown) is skipped (their names are logged once at `console.debug` — hidden from users, but a maintainer can enable "Verbose" to spot a new content tool to add). Artifacts are special — `collectArtifacts` folds each `id` through `create` / `update` (an `old_str`→`new_str` diff) / `rewrite` into its final content, and it's rendered **once, at its last edit** (matched by `version_uuid`).

3. **`buildMarkdown(messages)`** — emits YAML frontmatter (`title`, `source`, `model`, `exported`) then one `# Human — <timestamp>` / `# Claude — <timestamp>` header per turn (H1 so Claude's own `##`/`###` content nests beneath it — keeps the outline correct for RAG chunking and Obsidian). No separate title heading or `---` rules. Appends an inline word-based notice to **incomplete** messages via `incompleteNote()` — `truncated` → **Truncated**, `stop_reason: 'user_canceled'` → **Interrupted** (everything else, including a length-limited response, is left unannotated rather than guessing at an unobserved `stop_reason` value). Returns `{ markdown, interrupted, truncated }` — **counts, not a boolean**, because the two mean different things to the reader and only one of them is a problem:

| Signal | Meaning | Status box |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agarwalvishal/claude-chat-exporter](https://github.com/agarwalvishal/claude-chat-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
