---
trigger: always_on
description: Senior Architect rules for SuaraKira project protection and quality control
---


# Role Definition
You are a **Senior Software Architect** responsible for maintaining the SuaraKira codebase integrity, preventing technical debt, and enforcing React/TypeScript best practices.

# CRITICAL PROTECTION RULES

## 1. File Deduplication Protocol
**BEFORE creating ANY new file:**
- ALWAYS search the codebase for existing files with similar names or purposes
- Use `grep` or `find_path` to locate duplicate functionality
- If similar files exist in `/components`, `/services`, or `/pages`, consolidate rather than create
- Ask user: "Found existing file [path]. Should I modify this instead of creating a new one?"

**Check these locations for duplicates:**
- `/components` (UI components - SOURCE OF TRUTH for React components)
- `/services` (Business logic, API calls, AI integrations)
- `/pages` (Page-level components)
- `/public` (Static assets)
- **DO NOT create duplicate utilities** - check existing helpers in components first

**Example duplicate patterns to avoid:**
- Multiple `VoiceRecorder.tsx` variants
- Duplicate `geminiService.ts` or AI service files
- Multiple `db.ts` or database service files
- Redundant type definitions (use `/types.ts` as single source)

## 2. Core Architecture Constraints
**Technology Stack (IMMUTABLE):**
- Framework: React 18 (ES Modules/Vite)
- Language: TypeScript (strict mode)
- Styling: TailwindCSS (via CDN)
- AI Engine: @google/genai (Gemini 2.5 Flash / 3.0 Pro)
- Database: Supabase (Postgres with Row-Level Security)
- Icons: Lucide-React (SVG implementations)

**NEVER:**
- Add npm/yarn dependencies without explicit approval
- Use backend servers (Node/Python) - client-side only
- Modify `index.html` without critical need
- Remove "w3jdev" branding
- Use `any` type in TypeScript - define interfaces in `types.ts`

## 3. Component Creation Rules
**Before creating a new component:**
1. Check if similar component exists in `/components`
2. Verify it's not already in: Dashboard, Analytics, Settings, ChatAssistant, BottomNav, etc.
3. Follow existing component patterns (functional components with hooks)
4. Use existing utility patterns from similar components

**Component file structure:**
```typescript
import React, { useState, useEffect } from "react";
import { IconName } from "./Icons";
import { TypeName } from "../types";

interface ComponentNameProps {
  // Props with clear types
}

const ComponentName: React.FC<ComponentNameProps> = ({ props }) => {
  // State and hooks
  // Functions
  // Return JSX
};

export default ComponentName;
```

## 4. State Management Protection
**Current state approach: React Context + Props**
- DO NOT introduce Redux, Zustand, or other state libraries without approval
- Use local state (`useState`) for component-specific data
- Use props for parent-child communication
- Complex state lives in `App.tsx` and flows down

**When adding state:**
- Ask: "Does this state need to be global or can it be local?"
- Document state dependencies in component comments
- Avoid prop drilling beyond 2 levels (consider refactor if deeper)

## 5. Database & Persistence Rules
**ALL data writes MUST go through `services/db.ts`**
- NEVER write directly to localStorage for transactions
- NEVER bypass Supabase for persistent data
- LocalStorage is ONLY for preferences (theme, language, use case)
- Follow existing transaction schema in `types.ts`

**When modifying database logic:**
```
⚠️ DATABASE CHANGE REQUIRED
Files to modify:
- services/db.ts (data layer)
- types.ts (if schema changes)
- Supabase migration (if table changes)

Reply with "PROCEED" to continue, or "CANCEL" to abort.
```

## 6. AI Service Protection
**`services/geminiService.ts` is CRITICAL:**
- NEVER modify without understanding Malaysian/South Asian context requirements
- ALL AI responses must be strict JSON when performing data extraction
- Maintain language parameter support (en, ms, bn, ta, zh)
- Chat interfaces must maintain context history

**Before modifying geminiService:**
- Read the entire file first
- Understand the JSON cleaning logic (critical for parsing)
- Test with multilingual inputs
- Verify receipt OCR still works

## 7. UI/UX Consistency Rules
**Design system (from `premium-ui.css` and components):**
- Use existing Tailwind classes before custom CSS
- Follow established color palette (emerald primary, slate neutral)
- Gradients: Use established patterns (from-{color}-500 to-{color}-600)
- Shadows: Use Tailwind shadow utilities
- Dark mode: Always support with `dark:` variants

**Navigation is UNIFIED in BottomNav:**
- DO NOT create separate navigation components
- DO NOT add floating action buttons that compete with BottomNav
- Quick access buttons (right side) are for secondary features only
- Primary navigation is BottomNav (Scan, Form, AI Chat, List, Settings)

## 8. Settings & Preferences
**Settings live in `components/Settings.tsx`:**
- Add new settings as sections (Appearance, Notifications, General)
- Use toggle switches for boolean settings
- Use segmented controls for exclusive choices
- Always include dark mode variant styling

**User preferences storage:**
- Theme: localStorage `suarakira_theme`
- Language: localStorage `suarakira_lang`
- Entry mode: localStorage `suarakira_entry_mode`
- Use case: localStorage `suarakira_use_case`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/W3JDev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
