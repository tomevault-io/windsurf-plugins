---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

```bash
npm install       # Install dependencies
npm run dev       # Start development server (Vite, hot reload)
npm run build     # Production build → dist/
npm run preview   # Preview production build locally
```

No test suite exists. Validate changes visually via `npm run dev`.

## Architecture

**LLM-Infra-Explorer** is a React SPA for interactively visualizing LLM infrastructure concepts. It deploys as a static site to GitHub Pages.

**Stack:** React 18 + Vite + Tailwind CSS + Framer Motion + Lucide React

### Navigation flow

`main.jsx` → `App.jsx` → `MainDashboard.jsx` (sidebar + tab routing) → individual visualization components

`MainDashboard` manages which tab is active via local `useState`. Clicking a card on `HomeLanding` navigates to the corresponding visualization. The sidebar is collapsible on mobile.

### Visualization modules (`src/components/`)

Each file is a self-contained, interactive visualization:

| Component | Concept |
|---|---|
| `LLMInference.jsx` | Token prefill/decode, KV cache lifecycle, MoE vs Dense, temperature sampling |
| `ParallelStrategies.jsx` | 6D parallel topology (DP/TP/PP/CP/EP/ETP), tensor slicing, GPU mapping |
| `FlashAttention.jsx` | Tiled attention vs standard, SRAM/HBM IO tracking |
| `FlashDecode.jsx` | KV cache splitting, parallel reduction |
| `Engram.jsx` | DeepSeek n-gram memory retrieval, async prefetch |
| `RadixCache.jsx` | SGLang radix tree KV cache, LRU eviction, block reuse |
| `DpAttention.jsx` | DP/TP hybrid attention, KV cache sharding, cross-rank communication |
| `LinearAttention.jsx` | Softmax, kernelized linear attention, recurrent state, and GLA gating |

### Styling conventions

- Dark theme throughout: `bg-slate-950`, `text-slate-100`
- Framer Motion for animated transitions (`motion.div`, `AnimatePresence`)
- `clsx` + `tailwind-merge` for conditional class composition
- Responsive breakpoints: `md:` for sidebar/layout changes

### Visualization component conventions

Every visualization component (`src/components/*.jsx`) follows the durable project conventions below. Page structure and stage count remain concept-dependent; do not force a module into a layout that misrepresents the underlying algorithm.

**Theme:** Components use light mode internally (`bg-slate-50 text-slate-800`, cards as `bg-white border border-slate-200`), contrasting with the dark shell. This "entering a workbench" feel is intentional.

**Top control bar structure (default):** Keep title + subtitle, primary comparison mode, reset/play/step controls, and language switching together when they affect the whole module. Place controls that affect only one lower canvas beside that canvas instead of in the global header.

**i18n pattern:** All text goes through `t(key)`. Every component has a top-level `i18n = { zh: {...}, en: {...} }` object where `zh` and `en` keys are identical. Language is initialized via `getInitialLang()` (`navigator.language` check). Never hardcode display strings in JSX.

**Mathematical notation:** All mathematical expressions must be authored as LaTeX and rendered through the shared KaTeX-backed math component (`MathFormula`). Do not imitate formulas with plain strings, Unicode subscripts/superscripts, or `font-mono`. Keep language-dependent prose in i18n, but keep language-independent LaTeX source outside the translation dictionaries. Complex equations must be paired with a variable explanation or a visualization that makes their role clear.

**Timeline capability:** Use a playback state machine only when ordered execution is part of the learning object:
```js
const [phase, setPhase] = useState('idle');   // 'idle' | 'running' | 'done'
const [step, setStep] = useState(0);
const [isPlaying, setIsPlaying] = useState(false);
// Timeline modules require handleNextStep(), reset(), togglePlay().
```

**Timeline auto-play pattern:**
```js
useEffect(() => {
  if (!isPlaying || isDone) return;
  const timer = setTimeout(handleNextStep, delay); // delay varies per step
  return () => clearTimeout(timer);
}, [isPlaying, step, ...deps]);
```

**Canonical domain model:** Store only user-controlled and genuine lifecycle state. Normalize constrained inputs, then use a pure `getXxxState(...)`, `deriveXxxModel(...)`, or equivalent function to compute all render data. Rendering is always a pure `model → UI` mapping with no side effects.

**Interaction capabilities:** Declare only the capabilities a module needs: timeline, multiple modes, resource metrics, structural comparison, data movement, dense layout, and math. Timeline state such as token/chunk position and active stage is conditional; mode, execution context, dimensions, parameters, selection, and lifecycle state must still drive every dependent canvas, metric, formula, code highlight, and explanation from one model.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skyliulu/LLM-Infra-Explorer](https://github.com/skyliulu/LLM-Infra-Explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
