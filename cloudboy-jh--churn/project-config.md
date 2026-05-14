---
trigger: always_on
description: **Developer Guidance for Churn CLI Development**
---

# CLAUDE.md

**Developer Guidance for Churn CLI Development**

This file provides essential context for AI assistants (like Claude Code) working with this codebase. For comprehensive documentation, see [`docs/development/CLAUDE.md`](./docs/development/CLAUDE.md).

---

## Project Overview

**Churn 2.0** - Local-first context engine for AI code agents  
**Published as**: `churn-cli` on npm  
**Current Version**: 2.1.5  
**Repository**: https://github.com/cloudboyjh1/churn2.0

**Core Philosophy:**
- Local-first (no cloud backend, all data stored locally)
- Zero-setup (run from any Git repository)
- Multi-model support (Claude, GPT, Gemini, Ollama)
- Privacy-focused (API keys stored locally, never shared)

---

## Quick Reference

### Technology Stack
- **Runtime**: Bun 1.x (NOT Node.js)
- **Language**: TypeScript 5.6 (ESNext modules, strict mode)
- **UI Framework**: Ink 5.x (React for terminal)
- **Build Tool**: Bun bundler
- **Dependencies**: See package.json for full list

### Essential Commands
```bash
bun install              # Install dependencies
bun run dev              # Run in development mode
bun run type-check       # TypeScript type checking
bun run build            # Build to dist/
bun run compile          # Create standalone binary
```

### Project Structure
```
src/
├── index.tsx           # CLI entry point, phase routing
├── components/         # Ink UI components (Logo, ModelSelect, etc.)
├── engine/             # Business logic (no UI dependencies)
│   ├── config.ts      # Local storage management
│   ├── git.ts         # Repository operations
│   ├── models.ts      # AI provider abstraction
│   ├── analysis.ts    # Parallel file analysis with caching
│   └── reports.ts     # Report generation and export
├── commands/           # Command implementations
└── theme.ts           # Visual design system (#ff5656 vibrant red)
```

---

## Code Style & Conventions

### Import Style
- **ALWAYS** use `.js` extension for local imports (even for `.ts` files)
- ESM only - no CommonJS
```typescript
// ✅ Correct
import { Logo } from "./components/Logo.js";
import { getRepoInfo } from "./engine/git.js";

// ❌ Wrong
import { Logo } from "./components/Logo";
```

### Component Patterns
- Use functional components with hooks
- Always type props with interfaces
- Ink components return `JSX.Element`, not HTML
```typescript
interface MyComponentProps {
  title: string;
  onComplete: () => void;
}

export function MyComponent({ title, onComplete }: MyComponentProps) {
  return <Box>{/* ... */}</Box>;
}
```

### React Hooks Rules
**CRITICAL**: Hooks must be called in the same order every render
- Never call hooks conditionally
- Never call hooks in loops
- Never call hooks in nested functions

```typescript
// ✅ Correct - Hooks at component top level
function MyComponent() {
  const [state, setState] = useState(false);
  
  useEffect(() => {
    if (state) {
      // Conditional logic INSIDE the hook
    }
  }, [state]);
  
  if (state) {
    return <Box>State is true</Box>;
  }
  return <Box>State is false</Box>;
}

// ❌ Wrong - Conditional hook call
function MyComponent() {
  const [state, setState] = useState(false);
  
  if (state) {
    useEffect(() => { /* ... */ }, []); // ❌ Hook called conditionally
  }
  
  return <Box>{/* ... */}</Box>;
}
```

### Error Handling
- Individual file analysis failures should NOT crash entire analysis
- Always display user-friendly error messages
- Log errors but continue processing
```typescript
try {
  const result = await analyzeFile(file);
} catch (error) {
  console.error(`Failed to analyze ${file}: ${error.message}`);
  // Continue processing other files
}
```

### Path Handling
- Display paths: Use `path.relative()` for readability
- Operations: Use `path.resolve()` or absolute paths
- Always handle Windows paths correctly

---

## Design System

### Colors (Theme)
All defined in `src/theme.ts`:
```typescript
Primary:   #ff5656  // Vibrant red (logo, progress bars, highlights)
Secondary: #ff8585  // Light red (gradients)
Text:      #f2e9e4  // Warm white
Gray:      #a6adc8  // Muted purple-gray
Success:   #a6e3a1  // Green
Info:      #8ab4f8  // Blue
Warning:   #f9e2af  // Yellow
Error:     #f38ba8  // Pink
```

### Symbols
```typescript
symbols.tick    // ✓
symbols.cross   // ✗
symbols.pointer // ›
symbols.bullet  // •
```

---

## Common Patterns

### Phase-Based Routing
The app uses a state machine approach with `AppPhase` type:
```typescript
type AppPhase = "init" | "model" | "confirm" | "run" | "review" | "export" | "complete";
```

Each phase renders a different UI component. Phase transitions happen via `setPhase()`.

### Streaming AI Responses
All AI providers support streaming for real-time UI updates:
```typescript
await sendPrompt(modelConfig, messages, {
  onStream: (chunk) => {
    // Update UI with chunk
    setProgress(prev => prev + chunk);
  }
});
```

### File Analysis with Caching
Analysis uses content-based SHA-256 caching (30-day retention):
```typescript
const cache = await loadCache();
const hash = createHash('sha256').update(content).digest('hex');

if (cache[filePath]?.hash === hash && !isCacheExpired(cache[filePath])) {
  return cache[filePath].suggestions; // Use cached result
}
```

---

## Critical Constraints

### File Size Limits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudboy-jh/churn](https://github.com/cloudboy-jh/churn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
