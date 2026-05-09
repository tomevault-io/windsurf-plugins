---
trigger: always_on
description: This document describes the AI system that is currently implemented in OpenSCAD Studio.
---

# AGENTS.md - AI Agent Architecture

This document describes the AI system that is currently implemented in OpenSCAD Studio.

## Overview

OpenSCAD Studio runs the AI copilot entirely on the client side.

- The same React/TypeScript AI stack is used in both the standalone web app and the Tauri desktop app.
- Model requests are still made directly from the frontend with the Vercel AI SDK.
- Tauri provides desktop shell features such as native file dialogs, filesystem access, native rendering, and a desktop-only localhost MCP bridge for external agents.
- OpenSCAD rendering is client-side: web uses `openscad-wasm` in a Web Worker, while the desktop app uses a bundled native OpenSCAD binary invoked via Tauri IPC commands.

The top-level `README.md` is user-facing. Keep it focused on product-level information and avoid turning it into an engineering index; architecture, roadmap, analytics, and implementation details should live in assistant/developer docs instead.

## Design Context

### Users

OpenSCAD Studio should optimize first for hobbyist makers. These users are building precise 2D and 3D models for fabrication, prototyping, printing, laser cutting, and personal projects. They value tools that help them move quickly without feeling simplified or toy-like, and they want an environment that supports experimentation while still feeling trustworthy for exact work.

### Brand Personality

The brand personality is build, speed, precision. The product should feel professional, technically credible, and efficient. It should support focused making work with confidence and clarity rather than leaning on playful novelty or overt AI-first theatrics.

### Aesthetic Direction

Preserve and tighten the current aesthetic instead of replacing it. The existing theme system is a strength and should remain intact, with Solarized Dark continuing as the default baseline unless intentionally changed elsewhere. The visual direction should take cues from tools like Ableton Live and Affinity Designer: dense but deliberate, polished, capable, and tuned for serious creative work. Avoid anything that feels generic, glossy, or obviously AI-generated.

### Design Principles

1. Respect maker workflows. Prioritize fast iteration, precise feedback, and layouts that help users stay in flow while modeling.
2. Tighten, do not reinvent. Improve hierarchy, spacing, contrast, and polish within the current design language instead of introducing a disconnected visual reset.
3. Feel professionally technical. Interfaces should communicate capability and precision without becoming cold, cluttered, or intimidating.
4. Preserve theme flexibility. New UI work must fit naturally into the existing theme system and work well in both dark and light themes.
5. Avoid AI-generic styling. Favor intentional, grounded, tool-like design over flashy gradients, empty decoration, or trendy "generated" aesthetics.

## Current Architecture

```
┌─────────────────────────────────────────────────────────────┐
│ React Frontend (shared by Web + Tauri desktop)             │
│ ├── WelcomeScreen.tsx      (welcome composer)              │
│ ├── AiPromptPanel.tsx      (chat transcript + composer)    │
│ ├── AiComposer.tsx         (shared text/image input)       │
│ ├── useAiAgent.ts          (AI state, streaming, drafts)   │
│ ├── aiService.ts           (model + tool definitions)      │
│ └── useOpenScad.ts         (rendering + diagnostics)       │
└─────────────────────────────────────────────────────────────┘
                 │
                 │ HTTPS from the client
                 ▼
        ┌───────────────────┐
        │ Anthropic API     │
        └───────────────────┘
                 │
                 ▼
        ┌───────────────────┐
        │ OpenAI API        │
        └───────────────────┘

Desktop-only shell services:
┌─────────────────────────────────────────────────────────────┐
│ Tauri / Rust                                                │
│ ├── native menus                                            │
│ ├── file open/save/export commands                          │
│ ├── native OpenSCAD binary rendering (render.rs)            │
│ ├── localhost MCP server for external agents (mcp.rs)       │
│ ├── working-directory/history helpers                       │
│ └── desktop packaging/runtime                               │
└─────────────────────────────────────────────────────────────┘
```

## Security Model

### API keys

AI API keys are currently stored client-side in obfuscated localStorage-backed state, including when the app runs inside the Tauri webview.

- This is a convenience tradeoff for a shared web + desktop AI implementation.
- It is not equivalent to backend-only secret storage.
- The current architecture intentionally prioritizes one shared AI stack across web and desktop over secret isolation.

Relevant code:

- `apps/ui/src/stores/apiKeyStore.ts`
- `apps/ui/src/hooks/useAiAgent.ts`
- `apps/ui/src/services/aiService.ts`

### Editing model

The AI editing flow is still diff-based and validated client-side:

1. The model reads the current OpenSCAD code through tools.
2. The model proposes exact-string edits through `apply_edit`.
3. The edit is validated in TypeScript.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zacharyfmarion/openscad-studio](https://github.com/zacharyfmarion/openscad-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
