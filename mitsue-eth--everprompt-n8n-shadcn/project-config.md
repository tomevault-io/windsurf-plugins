---
trigger: always_on
description: Development workflow, coding standards, and project organization
---


# Development Workflow & Standards

## Project Structure

```
everprompt-n8n/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Auth route group
│   ├── (dashboard)/              # Dashboard route group
│   ├── api/                      # API routes
│   ├── globals.css               # Global styles
│   ├── layout.tsx                # Root layout
│   └── page.tsx                  # Home page
├── components/                   # Reusable components
│   ├── ui/                       # Base UI components
│   ├── features/                 # Feature-specific components
│   └── layout/                   # Layout components
├── lib/                          # Utility functions
│   ├── auth.ts                   # Authentication utilities
│   ├── db.ts                     # Database utilities
│   ├── utils.ts                  # General utilities
│   └── validations.ts            # Zod schemas
├── hooks/                        # Custom React hooks
├── store/                        # Zustand stores
├── types/                        # TypeScript type definitions
├── constants/                    # Application constants
├── styles/                       # Additional styles
└── public/                       # Static assets
```

## Coding Standards

### 1. **TypeScript Configuration**

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true
  }
}
```

### 2. **ESLint Configuration**

```javascript
// eslint.config.mjs
export default [
  {
    rules: {
      "@typescript-eslint/no-unused-vars": "error",
      "@typescript-eslint/no-explicit-any": "warn",
      "react-hooks/exhaustive-deps": "error",
      "prefer-const": "error",
      "no-var": "error",
    },
  },
];
```

### 3. **Prettier Configuration**

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false
}
```

## Component Standards

### 1. **Component Structure**

```typescript
// components/features/PromptEditor.tsx
import React, { useState, useCallback } from "react";
import { cn } from "@/lib/utils";

interface PromptEditorProps {
  value: string;
  onChange: (value: string) => void;
  onSave?: () => void;
  className?: string;
}

export function PromptEditor({
  value,
  onChange,
  onSave,
  className,
}: PromptEditorProps) {
  const [isSaving, setIsSaving] = useState(false);

  const handleSave = useCallback(async () => {
    if (!onSave) return;

    setIsSaving(true);
    try {
      await onSave();
    } finally {
      setIsSaving(false);
    }
  }, [onSave]);

  return (
    <div className={cn("prompt-editor", className)}>
      <textarea
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className="w-full h-full resize-none bg-transparent border-none outline-none"
        placeholder="Start crafting your prompt..."
      />
      {onSave && (
        <button
          onClick={handleSave}
          disabled={isSaving}
          className="save-button"
        >
          {isSaving ? "Saving..." : "Save"}
        </button>
      )}
    </div>
  );
}
```

### 2. **Hook Standards**

```typescript
// hooks/usePromptEditor.ts
import { useState, useCallback, useEffect } from "react";
import { debounce } from "lodash-es";

interface UsePromptEditorOptions {
  initialValue?: string;
  onSave?: (value: string) => Promise<void>;
  debounceMs?: number;
}

export function usePromptEditor({
  initialValue = "",
  onSave,
  debounceMs = 1000,
}: UsePromptEditorOptions = {}) {
  const [value, setValue] = useState(initialValue);
  const [isSaving, setIsSaving] = useState(false);
  const [lastSaved, setLastSaved] = useState<Date | null>(null);

  const debouncedSave = useCallback(
    debounce(async (newValue: string) => {
      if (!onSave) return;

      setIsSaving(true);
      try {
        await onSave(newValue);
        setLastSaved(new Date());
      } catch (error) {
        console.error("Failed to save prompt:", error);
      } finally {
        setIsSaving(false);
      }
    }, debounceMs),
    [onSave, debounceMs]
  );

  const handleChange = useCallback(
    (newValue: string) => {
      setValue(newValue);
      debouncedSave(newValue);
    },
    [debouncedSave]
  );

  return {
    value,
    setValue,
    handleChange,
    isSaving,
    lastSaved,
  };
}
```

### 3. **API Route Standards**

```typescript
// app/api/prompts/route.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";
import { getServerSession } from "next-auth";
import { db } from "@/lib/db";

const CreatePromptSchema = z.object({
  title: z.string().min(1).max(255),
  content: z.string().min(1),
  labelIds: z.array(z.string().uuid()).optional(),
  isPublic: z.boolean().optional().default(false),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user?.id) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const body = await request.json();
    const data = CreatePromptSchema.parse(body);


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
