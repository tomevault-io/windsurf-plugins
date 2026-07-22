---
trigger: always_on
description: Open-source Chrome extension that auto-captures browser workflows and generates step-by-step guides. No backend, no account, no data leaves the browser.
---

# Mimik

Open-source Chrome extension that auto-captures browser workflows and generates step-by-step guides. No backend, no account, no data leaves the browser.

## What It Does

You click "Record," perform a workflow in your browser, and Mimik automatically captures each action as a step with an annotated screenshot and description. You can edit the guide, replay it on a live page, or export it as a file.

**Core loop: Record → Edit → Replay or Export.**

## Architecture

**Everything runs in the Chrome extension. No backend.**

- Storage: IndexedDB via Dexie.js (browser-local)
- AI descriptions: optional, user provides their own API key in settings
- Export: generated client-side (no server rendering)
- No auth, no database, no hosting, no Docker

### Directory Structure

```
src/
├── core/                    # Business logic (no UI dependencies)
│   ├── capture/             # Recording pipeline
│   │   ├── ai/              # AI description + title generation (Vercel AI SDK)
│   │   │   ├── description.ts   # getAIDescription (DOM context → AI → step text)
│   │   │   ├── title.ts         # generateGuideTitle (steps → AI → guide name)
│   │   │   ├── models.ts        # AI_PROVIDERS config (OpenAI/Anthropic model lists)
│   │   │   ├── prompts.ts       # Prompt templates
│   │   │   └── provider.ts      # createModel factory (OpenAI/Anthropic)
│   │   ├── dom/              # DOM extraction utilities
│   │   │   ├── context.ts       # DOMContext extraction + serialization
│   │   │   ├── element-meta.ts  # extractElementMeta (selector, text, aria, rect)
│   │   │   └── element-utils.ts # findFocusableAncestor, isTextField, etc.
│   │   ├── events/           # Event capture system
│   │   │   ├── handlers.ts      # CaptureController class + startCapture
│   │   │   ├── highlight.ts     # HighlightManager (dashed border overlay)
│   │   │   └── input-session.ts # InputSession (typing lifecycle)
│   │   ├── machine.ts        # xstate capture state machine
│   │   ├── rrweb-recorder.ts # DOM recording for replay
│   │   ├── session.ts        # CaptureSession (lifecycle manager)
│   │   ├── spa-nav.ts        # SPA navigation tracking
│   │   ├── start-notification.ts # Recording notification overlay
│   │   └── step-description.ts   # Fallback rule-based descriptions
│   ├── blur/                # Smart blur: regex presets, DOM scanner, element picker, panel UI
│   ├── export/              # HTML, PDF, Markdown export generators + shared utils
│   └── guides/              # Data layer: types, Dexie DB, CRUD service
├── entrypoints/             # Chrome extension entry points (WXT)
│   ├── background/          # Service worker: state machine, message handlers, tab management
│   ├── content.ts           # Content script: CaptureSession, event listeners, rrweb
│   ├── sidepanel/           # Side panel React mount
│   ├── fullview/            # Full-page view React mount
│   ├── onboarding/          # Onboarding wizard (opens on first install)
│   └── options/             # Settings page React mount
├── lib/                     # Shared utilities
│   ├── messaging.ts         # Extension messaging protocol (webext-core)
│   ├── port.ts              # Long-lived port: background ↔ sidepanel
│   ├── browser-api.ts       # Chrome API wrappers
│   ├── tab-messages.ts      # Content script message types
│   ├── logger.ts            # Logging utility
│   └── utils.ts             # Shared helpers (dates, URLs, cn)
├── stores/                  # Zustand state stores
│   └── fullview.ts          # Fullview UI state (search, counts, guide data)
└── ui/                      # React components
    ├── components/ui/       # shadcn/ui primitives (button, input, dialog, badge)
    ├── fullview/            # Full-page dashboard
    │   ├── components/      # Extracted sub-components (grid, list, search, etc.)
    │   ├── App.tsx
    │   ├── TopNav.tsx
    │   ├── SearchModal.tsx
    │   ├── GuideContent.tsx
    │   ├── LibraryContent.tsx
    │   └── router.ts
    ├── sidepanel/           # Side panel UI
    │   ├── App.tsx
    │   ├── LibraryView.tsx
    │   ├── GuideEditor.tsx
    │   ├── RecordingView.tsx
    │   ├── StepCard.tsx
    │   ├── ExportMenu.tsx
    │   ├── BlurCanvas.tsx
    │   └── ZoomScreenshot.tsx
    ├── onboarding/          # Onboarding wizard UI
    │   └── App.tsx          # 5-step wizard (welcome, AI, blur, pin, done)
    ├── shared/              # Shared UI components
    │   └── SettingsView.tsx  # AI settings (provider, model, API key)
    └── options/             # Settings page
        └── App.tsx
```

## State Management

| Layer | Tool | Purpose |
|-------|------|---------|
| Capture lifecycle | xstate | State machine (IDLE ↔ RECORDING) in background service worker |
| Fullview UI | Zustand | Search modal, guide counts, active guide data |
| Persistence | Dexie (IndexedDB) | Guides, steps, screenshots, rrweb chunks |
| Service worker recovery | sessionStorage | xstate machine snapshot persistence |
| Background → Sidepanel | Port messaging | Real-time state broadcast |
| Cross-context sync | BroadcastChannel | Guide mutations (star, delete) across sidepanel/fullview |

## Extension Entry Points


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [westpoint-io/mimik](https://github.com/westpoint-io/mimik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
