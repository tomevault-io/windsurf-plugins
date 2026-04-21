---
trigger: always_on
description: > This file provides context for Claude when working on this codebase.
---

# CLAUDE.md - AI Prompt Engine

> This file provides context for Claude when working on this codebase.

## Project Overview

**AI Prompt Engine** is a visual interface that transforms user selections into production-ready prompts for AI code generation. Instead of generating code directly, it generates expertly-crafted prompts that users can take to any AI model (Claude, GPT, Gemini) for consistent, high-quality results.

### Core Value Proposition
- **Model-agnostic**: Generated prompts work with any AI
- **Zero API costs**: Runs entirely client-side
- **Reusable**: Save and share prompts as templates
- **Educational**: Users learn prompt engineering

## Tech Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| Framework | Next.js 14 (App Router) | Server Components, routing |
| UI Library | Shadcn/UI | Toggle groups, cards, copy button, toasts |
| State | React Hook Form + Zod | Form state and validation |
| Styling | Tailwind CSS | Utility-first styling |
| Storage | localStorage | Persist user configs and saved prompts |
| Icons | Lucide React | Consistent iconography |

## Project Structure

```
src/
├── app/
│   ├── page.tsx              # Main prompt builder interface
│   ├── library/
│   │   └── page.tsx          # Saved prompts library
│   └── layout.tsx            # Root layout with providers
├── components/
│   ├── ui/                   # Shadcn components
│   ├── stack-selector.tsx    # Tech stack toggle groups
│   ├── spec-kit-editor.tsx   # Standards/rules editor
│   ├── prompt-preview.tsx    # Generated prompt display
│   ├── prompt-actions.tsx    # Copy, download, share buttons
│   └── model-hints.tsx       # AI model optimization tips
├── lib/
│   ├── prompt-builder.ts     # Core prompt generation logic
│   ├── templates/            # Prompt section templates
│   │   ├── role.ts
│   │   ├── context.ts
│   │   ├── spec-kit.ts
│   │   ├── task.ts
│   │   ├── constraints.ts
│   │   ├── output-format.ts
│   │   └── verification.ts
│   ├── presets.ts            # Pre-configured stack combinations
│   └── storage.ts            # localStorage utilities
├── hooks/
│   ├── use-prompt-config.ts  # Form state management
│   └── use-saved-prompts.ts  # Library CRUD operations
└── types/
    └── prompt-config.ts      # TypeScript interfaces
```

## Code Conventions

### File Naming
- **Files**: `kebab-case` (e.g., `stack-selector.tsx`, `prompt-builder.ts`)
- **Components**: `PascalCase` (e.g., `StackSelector`, `PromptPreview`)
- **Functions**: `camelCase` (e.g., `buildPrompt`, `getStoredConfigs`)
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g., `DEFAULT_CONFIG`, `STACK_OPTIONS`)

### Component Patterns
```tsx
// Prefer named exports for components
export function StackSelector({ config, onChange }: StackSelectorProps) {
  // Component logic
}

// Colocate types with components when specific to that component
interface StackSelectorProps {
  config: PromptConfig;
  onChange: (config: Partial<PromptConfig>) => void;
}
```

### State Management
- Use React Hook Form for the main configuration form
- Use Zod schemas for runtime validation
- Persist to localStorage on config changes
- No global state library needed (prop drilling is fine for this scale)

## Core Data Types

```typescript
// types/prompt-config.ts

interface PromptConfig {
  // Stack Selections
  frontend: "nextjs" | "vite" | "remix" | "astro";
  backend: "node" | "bun" | "go" | "python";
  database: "drizzle" | "prisma" | "none";
  auth: "clerk" | "authjs" | "supabase" | "none";
  styling: "tailwind" | "css-modules" | "styled-components";
  
  // Project Details
  projectName: string;
  projectType: "saas" | "ecommerce" | "blog" | "api" | "custom";
  description?: string;
  
  // Spec Kit (User's Standards)
  specKit: SpecKit;
  
  // Output Preferences
  outputFormat: "json" | "markdown" | "files-list";
  includeExamples: boolean;
  targetModel: "claude" | "gpt" | "gemini" | "any";
}

interface SpecKit {
  fileNaming: "kebab-case" | "camelCase" | "PascalCase";
  componentNaming: "PascalCase" | "camelCase";
  folderStructure: "feature-based" | "type-based" | "hybrid";
  customRules: string[];
}

interface SavedPrompt {
  id: string;
  name: string;
  config: PromptConfig;
  generatedPrompt: string;
  createdAt: string;
  updatedAt: string;
}
```

## Prompt Architecture

Generated prompts follow this structure:

1. **Role Definition** - Sets the AI's persona
2. **Context Block** - Project type, tech stack, requirements
3. **Standards (Spec Kit)** - Naming conventions, folder structure, rules
4. **Task Specification** - What to generate
5. **Constraints** - What NOT to do, edge cases
6. **Output Format** - Expected response structure (JSON, markdown, etc.)
7. **Examples** - Optional few-shot examples
8. **Verification** - Self-check instructions for the AI

### Prompt Builder Implementation

```typescript
// lib/prompt-builder.ts

export function buildPrompt(config: PromptConfig): string {
  const sections = [
    buildRoleSection(config),
    buildContextSection(config),
    buildSpecKitSection(config),
    buildTaskSection(config),
    buildConstraintsSection(config),
    buildOutputSection(config),
    config.includeExamples ? buildExamplesSection(config) : "",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leighayanid) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
