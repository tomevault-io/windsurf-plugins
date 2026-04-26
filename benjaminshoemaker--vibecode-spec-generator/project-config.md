---
trigger: always_on
description: **Purpose**: This file orients automated agents (Claude Code, GitHub Copilot, Cursor, etc.) to the repository structure, architecture, testing requirements, and expectations for automated work.
---

# AGENTS.md

**Purpose**: This file orients automated agents (Claude Code, GitHub Copilot, Cursor, etc.) to the repository structure, architecture, testing requirements, and expectations for automated work.

Keep this file minimal, readable, and authoritative. If something here conflicts with other docs, stop and ask a human.

---

## Project Overview

**Vibe Scaffold** is a Next.js 15.1.0 application that implements a multi-step wizard interface for generating technical specification documents using AI-powered chat conversations. The AI assistant is called "Vibe Scaffold Assistant" in the chat interface.

### Core Concept
Users progress through 4 sequential steps, each involving:
1. **Chat Phase**: Interactive conversation with AI to gather requirements
2. **Generation Phase**: AI synthesizes chat history into a structured markdown document
3. **Approval Phase**: User reviews and approves before proceeding to next step

### Document Flow
1. **Step 1 - One Pager**: High-level product vision and requirements
2. **Step 2 - Dev Spec**: Technical specification and architecture details
3. **Step 3 - Prompt Plan**: Implementation checklist and staged development plan
4. **Step 4 - AGENTS.md**: Agent guidance and workflow documentation

---

## Key Files & Structure

### Configuration & State
- `app/store.ts` — Zustand store with localStorage persistence (key: `wizard-storage`)
- `app/types.ts` — TypeScript interfaces for state, steps, and configs
- `app/wizard/steps/step{1-4}-config.ts` — Step configurations (instructions, prompts, document inputs)

### Components
- `app/wizard/page.tsx` — Main wizard orchestrator with navigation, state management, and sidebar
- `app/wizard/components/WizardStep.tsx` — Per-step component handling chat/preview/generation with example modal
- `app/wizard/components/ChatInterface.tsx` — Custom streaming chat implementation (not using @ai-sdk/react hooks)
- `app/wizard/components/DocumentPreview.tsx` — Markdown renderer with raw/rendered toggle
- `app/wizard/components/FinalInstructionsModal.tsx` — Completion modal with download, copy command, and email subscribe

### API Routes (Edge Runtime)
- `app/api/chat/route.ts` — Streaming chat endpoint using Vercel AI SDK `streamText()`
- `app/api/generate-doc/route.ts` — Streaming document generation endpoint
- `app/api/subscribe/route.ts` — Email subscription endpoint
- `app/api/spikelog/route.ts` — Analytics event logging endpoint
- `app/api/log-metadata/route.ts` — Spec metadata logging endpoint

### Utilities
- `app/wizard/utils/sampleDocs.ts` — Sample documents for quick testing/development
- `app/wizard/utils/stepAccess.ts` — Step access validation (prevents skipping steps)
- `app/utils/analytics.ts` — Google Analytics tracking utilities
- `app/utils/spikelog.ts` — Custom analytics event logging

### Tests
- `tests/` — Vitest test suite with 200 tests (see `tests/README.md`)
- `tests/unit/` — Unit tests (store, utilities, components)
- `tests/integration/api/` — API integration tests

---

## Architecture Deep Dive

### State Management
**Zustand** with localStorage persistence ensures state survives page refreshes:
- Key: `wizard-storage`
- Each step stores: `chatHistory`, `generatedDoc`, `approved` status
- Navigation locked until current step approved
- State mapping in `app/wizard/page.tsx` line 18: `stepKeyMap`

**Step State Structure** (`app/types.ts`):
```typescript
StepData {
  chatHistory: Message[]      // All chat messages for this step
  generatedDoc: string | null // Generated markdown document
  approved: boolean           // Whether step is complete
}
```

**Wizard State** includes:
- `currentStep: number` — Current step (1-4)
- `isGenerating: boolean` — Whether document generation is in progress (transient, not persisted)
- `resetCounter: number` — Incremented on reset to force component re-renders
- `steps: { onePager, devSpec, checklist, agentsMd }` — Per-step data

### Chat Implementation
Custom streaming implementation (not using `useChat` hook due to version compatibility):
- Manual fetch from `/api/chat` and reads streamed text chunks
- Appends streamed chunks directly to the latest assistant message
- Implementation in `app/wizard/components/ChatInterface.tsx`

**Stream Handling Logic**:
```typescript
const reader = response.body?.getReader();
const decoder = new TextDecoder();
// Read chunks and append to assistant message
const chunk = decoder.decode(value);
assistantMessage = {
  ...assistantMessage,
  content: assistantMessage.content + chunk,
};
setMessages([...updatedMessages, assistantMessage]);
```
**Note**: This assumes API returns plain text chunks; if streaming format changes, update this code.

### Document Generation
- `/api/generate-doc` receives: `chatHistory`, `stepName`, `documentInputs` (previous docs for context), `generationPrompt`
- Step 2+ can reference earlier documents (e.g., Step 2 receives Step 1's one-pager)
- Streaming generation for progressive document display
- Document context passed via step config's `documentInputs` array
- AGENTS.md documents get attribution appended: `<!-- Generated with vibescaffold.dev -->`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benjaminshoemaker/vibecode_spec_generator](https://github.com/benjaminshoemaker/vibecode_spec_generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
