---
trigger: always_on
description: **pi-web** (`@iswitthere/pi-web-desktop`) is a web-based UI for the [pi coding agent](https://github.com/badlogic/pi-mono). It provides a browser workspace for session browsing, real-time chat, model configuration, skill management, and project file preview — all powered by the pi SDK (`@earendil-works/pi-coding-agent`).
---

# AGENTS.md - Project Guide for Coding Agents

## Project Overview

**pi-web** (`@iswitthere/pi-web-desktop`) is a web-based UI for the [pi coding agent](https://github.com/badlogic/pi-mono). It provides a browser workspace for session browsing, real-time chat, model configuration, skill management, and project file preview — all powered by the pi SDK (`@earendil-works/pi-coding-agent`).

- **Purpose**: Give pi users a visual alternative to the CLI with session browsing, file exploration, and settings management.
- **Target Audience**: Developers using pi as their coding agent who want a richer UI.
- **Key Features**: Session browsing & forking, real-time SSE chat, file explorer & viewer, model/plugins/skills configuration, pi CLI theme system (load & preview JSON themes), Git worktree management, Electron desktop app support, i18n (English & 中文).

---

## Fork Strategy & Upstream Integration Policy

This project began from upstream pi-web `v0.7.16`, but is now a deliberately diverged **Electron-first desktop product**. Upstream pi-web remains an important source of Pi SDK compatibility, security, correctness, and reliability improvements; it is **not** the UI/UX baseline for this repository.

### Product Priority

1. **Preserve the desktop product experience.** Frameless Electron behavior, the custom title bar, Phosphor/provider icons, IA Writer Quattro/Lilex typography, advanced theme system, process-timeline message UI, right-side navigation, enhanced Markdown, and desktop sidebar interactions are protected product assets.
2. **Align upstream core behavior.** Prefer porting upstream fixes for security boundaries, Pi SDK/API compatibility, model/session/auth correctness, filesystem safety, and SSE reliability.
3. **Port behavior, not whole components.** When an upstream core change needs UI support, implement its observable behavior inside the local component architecture. Do not wholesale replace local `components/` files, `AppShell.tsx`, theme systems, or CSS with upstream Web-first implementations. The i18n registry/provider/catalog is an approved standalone architecture migration, but its desktop startup behavior and local message coverage must be preserved.
4. **Migrate coupled features as a unit.** Do not partially port cross-layer changes. For example, model scope support must cover SDK resolution, API response/cache contracts, `AgentSession` construction, and local UI feedback—not merely model-selector filtering.
5. **Keep each upstream port reversible.** Separate SDK/runtime, security, data integrity, auth, SSE, and UI changes into focused commits with focused tests and manual validation.

### Mandatory Merge Rules

- Treat `ref-repos/` as read-only comparison material. It is never staged, committed, copied wholesale, or used as a replacement source tree.
- Never overwrite local `package.json`: preserve Electron packaging/scripts, fonts, icons, and desktop release tooling while manually reconciling upstream runtime dependencies.
- Treat `components/AppShell.tsx`, `SessionSidebar.tsx`, `ChatWindow.tsx`, `ChatInput.tsx`, `MarkdownBody.tsx`, `FileViewer.tsx`, `app/globals.css`, and `hooks/useTheme.ts` as high-conflict desktop assets. Apply targeted behavioral changes only.
- i18n may align with upstream through a focused `lib/i18n` registry/provider/catalog migration. Do not replace local `AppShell` or settings UI to obtain it; preserve `app/layout.tsx` pre-hydration language bootstrap, existing `pi-language` migration, `data-language`, and all desktop-specific translations. The migration is complete: all call sites use `useI18n()`; no `useLanguage()` shim remains.
- Security updates must be adapted to the Electron server lifecycle. In particular, changing listener binding, Host/Origin validation, CORS, or `PI_WEB_PASSWORD` must be validated with `electron/main.js` readiness checks and BrowserWindow startup; never retain wildcard API CORS merely for LAN convenience.
- Upstream UI enhancements such as resizable panels may be adopted only by manually integrating their interaction/accessibility behavior with the local title bar, responsive layout, CSS variables, and i18n.

Keep upstream-comparison notes outside the public repository; `ref-repos/` is intentionally ignored and local-only.

---

## Technology Stack

| Category | Technology | Version |
|----------|-----------|---------|
| **Runtime** | Node.js / Next.js | Next 16.2.12 |
| **UI Library** | React | ^19.2.4 |
| **Language** | TypeScript | ^5 |
| **Styling** | Tailwind CSS | ^4.2.2 |
| **Markdown** | react-markdown + rehype/remark plugins | ^10.1.0 |
| **Math** | KaTeX (rehype-katex) | ^0.16.47 |
| **Diagrams** | Mermaid | ^11.14.0 |
| **Syntax Highlight** | react-syntax-highlighter | ^16.1.1 |
| **Document Preview** | mammoth (DOCX) | ^1.12.0 |
| **Icons** | Phosphor Icons, LobeHub icons | |
| **Fonts** | IA Writer Quattro, Lilex (mono) | |
| **Desktop** | Electron + electron-builder | ^43.2.0 / ^26.15.3 |
| **Linting** | ESLint (eslint-config-next) | ^9 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isWittHere/pi-web-desktop](https://github.com/isWittHere/pi-web-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
