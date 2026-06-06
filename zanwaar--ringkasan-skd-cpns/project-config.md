---
trigger: always_on
description: This is a **Docusaurus 3-based documentation and interactive learning platform** for Indonesian Civil Service Exam (SKD CPNS) preparation. The project combines:
---

# Ringkasan SKD CPNS - AI Agent Instructions

## Project Overview

This is a **Docusaurus 3-based documentation and interactive learning platform** for Indonesian Civil Service Exam (SKD CPNS) preparation. The project combines:
- Static markdown documentation on SKD exam topics (TWK, TIU, TKP)
- Interactive React components for practice questions and AI-driven quiz generation
- Google Gemini AI integration to dynamically generate exam questions

**Key URL structure:** Base deployed at `https://zanwaar.github.io/ringkasan-skd-cpns/`

---

## Architecture & Data Flows

### Core Stack
- **Framework:** Docusaurus 3.5.2 (Next-gen static site generator)
- **UI Framework:** React 18 + TypeScript
- **Styling:** Tailwind CSS 3.4.13 + styled-components 6.1.13
- **Math Rendering:** KaTeX (via `remark-math` & `rehype-katex`)
- **AI Integration:** Google Generative AI (`@google/genai` ^1.30.0)
- **Diagram Visualization:** react-flow-renderer for interactive flowcharts

### Folder Structure
```
docs/               # Markdown content (organized by SKD sections: TIU, TWK, TKP)
src/
  components/       # React components
    ai-funpice/     # AI quiz generation module (HomeTIU, MateriTIU, ModalGenerateSoal)
    diagram/        # ReactFlow-based diagram components (CircleSeating, GaleriDiagram, etc.)
    shared/         # Shared utilities (data.jsx with kuis templates, materi.tsx)
  pages/            # Docusaurus pages (index, ai-funpice, quiz, materi-tiu)
  utils/            # apiConfig.ts (Gemini API key management)
  clientModules/    # setupEnv.ts (runtime environment variable injection)
  css/              # Global styles
```

### Critical Data Flows

**1. AI Quiz Generation Flow:**
```
SubkategoriCard (click) 
  → ModalGenerateSoal (prompt + user settings)
  → AIModelGemini.chatSession.sendMessage()
  → Google Gemini API (gemini-2.5-flash model)
  → JSON response stored in localStorage
  → quiz.tsx reads from localStorage and renders
```

**2. Environment Variable Resolution (Two-Stage):**
- **Build time:** `process.env.REACT_APP_GOOGLE_GEMINI_AI_API_KEY` via `.env` + docusaurus2-dotenv
- **Runtime:** `setupEnv.ts` injects into `window.REACT_APP_GOOGLE_GEMINI_AI_API_KEY`
- **Access:** `getGeminiApiKey()` in `apiConfig.ts` checks both sources

**3. Content Organization:**
- `HomeTIU` (HomepageFeatures pattern) displays category structure
- `MateriTIU` iterates through `materiTIU` data structure (hard-coded in component)
- Content referenced across: `materi-tiu.tsx`, `ai-funpice.tsx`, shared `data.jsx`

---

## Developer Workflows

### Local Development
```bash
npm install              # Install dependencies (requires Node ≥18)
npm start                # Start Docusaurus dev server (port 3000)
npm build                # Production build
npm run typecheck        # Run TypeScript validation
npm run deploy           # Build + push to gh-pages branch
```

### Key Commands
- **Type Checking:** `npm run typecheck` (not run in CI by default)
- **Build Optimization:** Tailwind CSS is pre-processed in `tailwind-config.cjs`
- **Environment Setup:** Must set `GOOGLE_GEMINI_AI_API_KEY` in `.env` for AI features to work

### Common Debugging Patterns
- **API Key Issues:** Check both `process.env` (build-time) and `window` object (runtime) in `apiConfig.ts`
- **Quiz Generation Failures:** Look at `ModalGenerateSoal.tsx` - validates JSON response before localStorage
- **Math Rendering:** KaTeX stylesheets loaded via CDN in `docusaurus.config.ts`, ensure `<link>` tags preserved

---

## Project-Specific Conventions

### Component Patterns

**Docusaurus Layout Wrapper (for pages):**
```tsx
import Layout from "@theme/Layout";
import { useDocusaurusContext } from "@docusaurus/useDocusaurusContext";

// Pages use Layout + useDocusaurusContext() for consistency
export default function MyPage(): JSX.Element {
  const { siteConfig } = useDocusaurusContext();
  return <Layout><main>...</main></Layout>;
}
```

**Data-Driven UI (TIU Material):**
Material categories are defined as static objects within components (e.g., `materiTIU` in `HomeTIU.tsx`), NOT fetched externally. Structure:
```tsx
const materiTIU = {
  tes_tiu: {
    "1": { kemampuan: string, subkategori: { "1.1": { judul, deskripsi, prompt } } }
  }
};
```

**AI Prompt Storage:**
Prompts stored in `subkategori[key].prompt` field and passed to `ModalGenerateSoal`. Prompt should specify JSON output format for parser.

### URL & Routing
- Uses Docusaurus client-side routing (e.g., `/ringkasan-skd-cpns/ai-funpice`)
- Base URL is `/ringkasan-skd-cpns/` (set in `docusaurus.config.ts`)
- All internal links must account for base URL in redirects

### Styling Approach
- **TailwindCSS first** for components (utility classes)
- **Module CSS** for Docusaurus theme overrides (`index.module.css` in pages)
- **Styled-components** for complex, reusable styled containers (see `PosDiagram`, `Funpice`)
- **Markdown:** Global styles in `src/css/custom.css`

### State Management
- **No Redux/Zustand** - uses React hooks + `localStorage` for persistence
- Quiz data stored in `localStorage.generatedQuestions` (JSON string from AI)
- Session data in component state

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zanwaar/ringkasan-skd-cpns](https://github.com/zanwaar/ringkasan-skd-cpns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
