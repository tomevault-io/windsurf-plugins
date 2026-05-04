---
trigger: always_on
description: **Direct commits, merges, or pushes to `main` are strictly forbidden — no exceptions.**
---

# Copilot Instructions for office-coding-agent

## ⛔ NEVER PUSH OR MERGE DIRECTLY TO MAIN

**Direct commits, merges, or pushes to `main` are strictly forbidden — no exceptions.**

All changes must go through a pull request on a feature branch:

1. `git checkout -b <branch-name>` — create a feature branch
2. Make and commit changes on that branch
3. `git push -u origin <branch-name>` — push the branch
4. Ask the user to review and merge the PR on GitHub

**Never run:**

- `git push origin main`
- `git merge <branch> ` while on `main`
- `git commit --no-verify` to bypass hooks and then push to `main`

Branch protection is enforced on GitHub (ruleset ID `13260767`). Any attempt to push directly to `main` will be rejected by the server.

**PRs must be merged via squash merge only.** Merge commits and rebase merges are disabled in the repository settings. When asking the user to merge a PR, always tell them to use **"Squash and merge"** on GitHub.

## ⛔ UI GOLDEN RULE: LOOK AND FEEL IDENTICAL TO VS CODE

**The add-in must look and feel identical to VS Code's Copilot Chat — no exceptions.**

Every UI element, interaction, and visual detail must match what a user sees in VS Code's Copilot Chat panel. When in doubt, open VS Code's Copilot Chat and copy what you see.

### Design System
- **Colors:** Use `--vscode-*` CSS custom properties from `src/styles/vscode-theme.css` — NEVER hardcode colors or use non-VS-Code values
- **Icons:** Use codicons (`@vscode/codicons`) for ALL icons — NEVER use lucide-react or other icon libraries for new icons
- **Typography:** VS Code font stack (13px Segoe UI) — NEVER change the base font size or family
- **Spacing & Radii:** Use VS Code corner radii (2px small, 4px medium, 8px large) and compact spacing — NEVER use large padding or oversized elements
- **Focus:** 1px solid `--vscode-focusBorder` outline — NEVER use box-shadow rings

### Chat Components
- **Messages:** Full-width, no chat bubbles. Assistant messages have a 22px Copilot avatar + "Copilot" label header
- **User messages:** Plain text, no border or background box
- **Thinking indicator:** Shimmer gradient animation (`chat-thinking-shimmer`) on text — NEVER use spinners or pulse animations
- **Tool invocations:** Collapsible sections with codicon chevron + shimmer on running tool name
- **Follow-up suggestions:** Blue link-colored text (`textLink-foreground`) with sparkle codicon — NEVER use bordered cards
- **Welcome screen:** Centered Copilot avatar, compact heading, blue link-colored suggestion items
- **Composer input:** VS Code input background/border, 8px border-radius, no toolbar divider line

### When Adding New UI
1. First, check how VS Code's Copilot Chat does it (open VS Code or check `microsoft/vscode` source)
2. Copy the HTML structure and CSS patterns from VS Code
3. Use the same class naming conventions and design tokens
4. If VS Code doesn't have a precedent for the feature, use the closest VS Code UI pattern (panel, quick pick, dialog, etc.)

## Project Overview

**office-coding-agent** is a Microsoft Office add-in with a single task pane UI and host-routed AI runtime behavior. The current implementation is **Excel-first**, but tools and prompts are selected by host (`excel`, `powerpoint`, etc.) to support future hosts without changing the UI.

## Key Technologies

- **React 18** + **Radix UI** + **Tailwind CSS v4** — task pane UI (MessageList, ChatComposer, AssistantMessage, UserMessage, MarkdownContent, ToolProgress, ActionBar); **VS Code design system** via `vscode-theme.css` (`--vscode-*` CSS custom properties) and **codicons** (`@vscode/codicons`) for all icons
- **GitHub Copilot SDK** (`@github/copilot-sdk`) — session management, streaming events, tool registration
- **WebSocket + JSON-RPC** — browser-to-proxy transport (`src/lib/websocket-client.ts`, `src/lib/websocket-transport.ts`)
- **Express + HTTPS** — local proxy server (`src/server.mjs`) that bridges WebSocket to the Copilot CLI
- **Zustand 5** — state management with persistence via `officeStorage` (OfficeRuntime.storage)
- **Vite 7** — bundling, dev server (HMR via middleware mode in Express)
- **TypeScript 5** — type safety
- **Vitest** — unit + integration testing (jsdom env)
- **Mocha** — E2E tests inside real Office Desktop hosts (Excel, PowerPoint, Word, Outlook)
- **Playwright** — browser UI tests for task pane flows

## Architecture

The add-in routes messages through a **local proxy server** — the browser cannot call the Copilot API directly.

```
Browser task pane (React)
         ↓ WebSocket (wss://localhost:3000/api/copilot)
Node.js proxy server  (src/server.mjs + src/copilotProxy.mjs)
         ↓ @github/copilot-sdk (manages CLI lifecycle internally)
GitHub Copilot API
```

The `useOfficeChat` hook creates a `WebSocketCopilotClient`, opens a `BrowserCopilotSession`, and maps incoming `SessionEvent` objects to `ChatMessage[]` state with per-message `thinkingText` fields.

### Prompt and CLI Agent System

The add-in uses a split prompt architecture with host targeting:

- **`src/services/ai/BASE_PROMPT.md`** — universal base prompt (progress narration + presenting choices)
- **`src/services/ai/prompts/*_APP_PROMPT.md`** — host-level app prompt (Excel/PowerPoint/Word/Outlook)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbroenne/office-coding-agent](https://github.com/sbroenne/office-coding-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
