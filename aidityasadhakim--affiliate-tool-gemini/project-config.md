---
trigger: always_on
description: NanoGen is a React-based AI image generator application designed to run on **Gemini Canvas**. It uses Google's Gemini 2.5 Flash Image Preview model to generate images from text prompts with multiple specialized tools (prompt extraction, virtual try-on, product photography, video script generation, and video frame generation).
---

# AGENTS.md - NanoGen (Gemini Canvas App)

## Project Overview

NanoGen is a React-based AI image generator application designed to run on **Gemini Canvas**. It uses Google's Gemini 2.5 Flash Image Preview model to generate images from text prompts with multiple specialized tools (prompt extraction, virtual try-on, product photography, video script generation, and video frame generation).

## Build, Lint & Test Commands

```bash
# Development
npm install          # Install dependencies
npm run dev          # Start dev server (http://localhost:5173)

# Build
npm run build        # Type check + build for production
tsc                  # Type check only (no build)

# Preview
npm run preview      # Preview production build locally

# Note: No test framework currently configured
# Note: No linting tools (ESLint/Prettier) currently configured
```

## CRITICAL: Single-File Architecture

**All modifications must be made to `App.jsx` only.**

Gemini Canvas only processes the `App.jsx` file. Other files exist solely for local development.

**When making changes:**
- ✅ **DO**: Edit `App.jsx`
- ❌ **DO NOT**: Create new component files, split code into modules, or add new `.tsx`/`.ts` files
- ❌ **DO NOT**: Expect changes in other files to reflect in Gemini Canvas
- ⚠️ **API Key**: Line 291/347/510/568/661/808/958/1008 has `const apiKey = ""` - this is intentional. Gemini Canvas runtime injects the API key. Never hardcode API keys.

## Code Style Guidelines

### File Organization & Imports

```javascript
// 1. React core imports first
import React, { useState, useRef, useEffect } from 'react';

// 2. External libraries (icons, etc.)
import { Sparkles, Image, AlertCircle, Download, Loader2 } from 'lucide-react';

// 3. Local imports (if any - avoid for Canvas deployment)
// import { helper } from './utils'; // ❌ Don't do this for Canvas

// 4. Component definition
const App = () => { /* ... */ };
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `App`, `SelectField`, `UploadField` |
| Functions | camelCase | `generateImages`, `handleImageUpload`, `buildEnhancedPrompt` |
| React Hooks | camelCase with `use` prefix | `useState`, `useRef`, `useEffect` |
| Constants | SCREAMING_SNAKE_CASE | `IMAGE_COUNT`, `API_URL` |
| Event Handlers | camelCase with `handle` prefix | `handleDrop`, `handleDragEnter` |
| Boolean vars | camelCase with `is/has/should` | `isLoading`, `isDragging`, `isMobile` |
| CSS classes | kebab-case or Tailwind | `brutalCard`, `brutal-btn` |

### TypeScript Usage

- Project uses **JSX** for main component (not TSX) to maintain Gemini Canvas compatibility
- Type checking configured in `tsconfig.json` with strict mode enabled
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safe property access
- Prefer explicit type annotations for complex objects

```javascript
// ✅ Good: Safe property access
const textPart = result.candidates?.[0]?.content?.parts?.find(p => p.text);

// ✅ Good: Type-safe file handling
const file = e.target.files?.[0];
if (!file) return;
```

### Formatting Standards

**Indentation**: 2 spaces (not tabs)
**Line Length**: Reasonable (no strict limit, but keep readable)
**Quotes**: Single quotes for strings, backticks for templates
**Semicolons**: Yes, always use semicolons
**Trailing Commas**: Use in arrays/objects for cleaner diffs

```javascript
// ✅ Good
const options = [
  { value: 'option1', label: 'Option 1' },
  { value: 'option2', label: 'Option 2' },
];

// ❌ Bad
const options = [
  {value: "option1", label: "Option 1"},
  {value: "option2", label: "Option 2"}
]
```

### Component Patterns

**State Management**: Use React hooks (`useState`, `useRef`, `useEffect`)
**Component Structure**: Inline components within `App` (no separate files)

```javascript
// ✅ Good: Inline component for reusability
const SelectField = ({ label, value, onChange, options, className = '' }) => (
  <div className={`space-y-1 ${className}`}>
    {/* Component JSX */}
  </div>
);

// Inside App component:
const App = () => {
  const [state, setState] = useState(initialValue);
  
  // Helper functions
  const helperFunction = () => { /* ... */ };
  
  // Event handlers
  const handleEvent = () => { /* ... */ };
  
  // Render
  return <div>{/* JSX */}</div>;
};
```

### Error Handling

**User-Facing Errors**: Always in Indonesian
**Console Errors**: Can be in English
**Retry Logic**: Use exponential backoff (see `fetchWithRetry`)

```javascript
// ✅ Good: Retry logic with exponential backoff
const fetchWithRetry = async (retries = 2) => {
  try {
    const res = await fetch(url, { method: 'POST', headers, body });
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res;
  } catch (e) {
    if (retries > 0) {
      await new Promise(r => setTimeout(r, 1000 * (3 - retries))); // Exponential backoff
      return fetchWithRetry(retries - 1);
    }
    throw e;
  }
};

// ✅ Good: User-facing error messages in Indonesian

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aidityasadhakim/affiliate-tool-gemini](https://github.com/aidityasadhakim/affiliate-tool-gemini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
