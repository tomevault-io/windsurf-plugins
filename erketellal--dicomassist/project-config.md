---
trigger: always_on
description: A web-based DICOM viewer with LLM chat integration. The core innovation is **smart slice filtering**: instead of dumping 400+ slices into an LLM, we intelligently select 10-20 relevant slices based on a clinical hint (what the doctor suspects), DICOM metadata, and a two-call LLM architecture.
---

# CLAUDE.md — DICOMassist

## Project Overview

A web-based DICOM viewer with LLM chat integration. The core innovation is **smart slice filtering**: instead of dumping 400+ slices into an LLM, we intelligently select 10-20 relevant slices based on a clinical hint (what the doctor suspects), DICOM metadata, and a two-call LLM architecture.

This is a portfolio project. Public GitHub repo + demo video. Goal: showcase clinical product knowledge + technical engineering skills.

## Tech Stack

- **Frontend**: React 18 + Vite + TypeScript
- **Viewer**: Cornerstone3D v4 (`@cornerstonejs/core@^4`, `@cornerstonejs/tools@^4`, `@cornerstonejs/dicom-image-loader@^4`)
- **Styling**: Tailwind CSS
- **Icons**: lucide-react
- **LLM**: Abstracted service layer (Claude API + Ollama, provider-agnostic interface)
- **LLM API access**: Client-side calls with user-provided API key (runtime input, not bundled)
- **Data**: Local DICOM files only (drag-and-drop), no backend

## Vite Configuration (Critical)

Cornerstone3D requires specific Vite config. This is non-negotiable:

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { viteCommonjs } from '@originjs/vite-plugin-commonjs';

export default defineConfig({
  plugins: [
    react(),
    viteCommonjs(), // Required: dicom-parser is still CommonJS
  ],
  optimizeDeps: {
    exclude: ['@cornerstonejs/dicom-image-loader'],
    include: ['dicom-parser'],
  },
  worker: {
    format: 'es',
  },
  assetsInclude: ['**/*.wasm'], // needed for codec WASM files
});
```

## Project Structure

```
DICOMassist/
├── src/
│   ├── viewer/              # Cornerstone3D setup, viewports, toolbar
│   │   ├── CornerstoneInit.ts       # One-time init of core + tools + imageLoader
│   │   ├── ViewportGrid.tsx          # Viewport layout (stack + MPR + grid)
│   │   ├── viewportUtils.ts          # Shared viewport info extraction helpers
│   │   ├── EmptyViewportOverlay.tsx  # Series picker for empty grid slots
│   │   ├── Toolbar.tsx               # Tool buttons (W/L, Zoom, Pan, Scroll, Length, etc.)
│   │   ├── DicomDropZone.tsx         # Drag-and-drop file loading
│   │   └── LoadingOverlay.tsx        # Prefetch progress indicator
│   ├── dicom/               # Metadata extraction and parsing
│   │   ├── MetadataExtractor.ts      # Extract DICOM tags, group by series, compute fields
│   │   ├── orientationUtils.ts       # Anatomical plane detection from direction cosines
│   │   └── types.ts                  # DICOM metadata type definitions
│   ├── filtering/           # Slice selection logic
│   │   ├── SliceSelector.ts          # Apply LLM selection plan to actual slices
│   │   ├── SliceExporter.ts          # Convert selected slices to JPEG for LLM
│   │   └── types.ts                  # SelectedSlice type
│   ├── llm/                 # LLM integration (provider-agnostic)
│   │   ├── LLMServiceFactory.ts      # Claude + Ollama service implementations
│   │   ├── PromptBuilder.ts          # Constructs prompts from metadata + hint
│   │   ├── useLLMChat.ts             # React hook: pipeline orchestration + state
│   │   └── types.ts                  # LLMService interface, SelectionPlan, ChatMessage
│   ├── ui/                  # App-level UI components
│   │   ├── SpotlightPrompt.tsx       # Cmd+K / Ctrl+K overlay prompt input
│   │   ├── ChatSidebar.tsx           # Collapsible sidebar with chat history
│   │   ├── PipelineView.tsx          # Pipeline step visualization
│   │   ├── PlanPreviewCard.tsx       # Selection plan summary card
│   │   ├── AssistantMessage.tsx      # Formatted LLM response with interactive slice refs
│   │   ├── SeriesBrowser.tsx         # Series list for grid slot selection
│   │   ├── MetadataPanel.tsx         # Shows extracted DICOM metadata summary
│   │   └── SettingsPanel.tsx         # LLM provider configuration (Claude/Ollama)
│   ├── utils/
│   │   └── logger.ts                 # Dev-gated console logging
│   ├── App.tsx
│   └── main.tsx
├── screenshots/             # Screenshots for README
├── CLAUDE.md
├── README.md
└── package.json
```

## Core User Flow

This is the single end-to-end workflow the MVP must support:

```
1. User drags DICOM folder onto the app
2. App loads files via Cornerstone3D fileManager (progressive loading with progress bar)
3. Metadata extracted from all DICOM headers (fast, no pixel decoding)
4. Files organized by series, sorted by instance/position
5. Viewer displays the primary axial series with standard tools
   - Primary series = axial series with the most slices (tie-break: lowest series number)
   - Axial detection uses Image Orientation Patient direction cosines, NOT series description
6. User hits Cmd+K → Spotlight-style prompt overlay appears
7. User types clinical hint: "Patient with hepatitis C history, evaluate for HCC"
8. FIRST LLM CALL (text-only, cheap):
   - Input: metadata summary + clinical hint
   - Output: structured SelectionPlan (which series, slice range, window/level, sampling)
9. App applies the plan:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erketellal/DICOMassist](https://github.com/erketellal/DICOMassist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
