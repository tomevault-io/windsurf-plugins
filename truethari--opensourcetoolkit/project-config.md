---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenSource Toolkit is a modern Next.js 15 application providing 32+ developer and utility tools. It's built with React 19, TypeScript, Tailwind CSS, and uses shadcn/ui components with Radix UI primitives.

## Development Commands

### Core Commands

```bash
npm run dev         # Start development server on port 5001 with Turbopack
npm run build       # Build for production
npm start           # Start production server
npm run lint        # Run ESLint
npm run format      # Format code with Prettier
npm run test        # Run format and lint checks (no unit tests)
```

### Port Configuration

- Development server runs on port 5001 (not 3000)
- Use `--turbopack` flag for faster builds

## Architecture Overview

### Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── (tools)/           # Grouped tool routes
│   ├── layout.tsx         # Root layout with providers
│   └── page.tsx           # Home page
├── components/
│   ├── ui/                # shadcn/ui components
│   ├── tools/             # Tool-specific components
│   ├── general/           # Navigation & layout
│   └── wrappers/          # Layout wrappers
├── config/                # Tool configuration & registry
├── hooks/                 # Custom React hooks
├── lib/                   # Utility libraries
├── providers/             # React providers
├── types/                 # TypeScript definitions
└── utils/                 # Utility functions
```

### Key Architectural Patterns

#### Tool Registration System

All tools are registered in `src/config/index.ts` with metadata including:

- Basic info (id, title, description)
- UI properties (icon, color, category)
- SEO metadata
- Features list
- Tags for search/filtering

#### Component Patterns

- All tool components use `ToolsWrapper` for consistent layout
- Tools are client-side only ("use client" directive)
- State management with React hooks (useState, useCallback, useMemo)
- Copy-to-clipboard functionality for results
- Loading states and error handling

#### Styling System

- Dark theme by default
- CSS variables for theming (HSL color format)
- Tailwind classes with design system tokens
- Responsive design (mobile-first)
- shadcn/ui component library

## Adding New Tools

This section provides a comprehensive guide for adding new tools to the toolkit. Follow these steps carefully to ensure consistency and proper integration.

### Step-by-Step Guide

#### 1. Create Tool Component

Create a new directory and component file:

```
src/components/tools/your-tool-name/index.tsx
```

**Component Structure Requirements:**

```typescript
"use client";

import { toast } from "sonner";
import React, { useState, useCallback, useMemo } from "react";
import { IconName } from "lucide-react";

// UI Components
import { Label } from "@/components/ui/label";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";

import ToolsWrapper from "@/components/wrappers/ToolsWrapper";

export default function YourToolName() {
  // State management
  const [input, setInput] = useState("");

  // Memoized calculations
  const result = useMemo(() => {
    // Expensive calculations here
    return processInput(input);
  }, [input]);

  // Event handlers with useCallback
  const copyToClipboard = useCallback(async (text: string, label: string) => {
    try {
      await navigator.clipboard.writeText(text);
      toast.success(`${label} copied to clipboard`);
    } catch (err) {
      toast.error("Failed to copy to clipboard");
    }
  }, []);

  return (
    <ToolsWrapper>
      {/* Header section */}
      <div className="mb-8 text-center">
        <div className="mx-auto mb-4 flex h-16 w-16 items-center justify-center rounded-2xl bg-blue-600 text-white">
          <IconName className="h-8 w-8" />
        </div>
        <h1 className="mb-2 text-4xl font-bold text-gray-900 dark:text-white">
          Tool Title
        </h1>
        <p className="text-lg text-gray-600 dark:text-gray-300">
          Tool description
        </p>
      </div>

      {/* Main content */}
      <div className="grid gap-6 lg:grid-cols-3">
        {/* Input section */}
        <div className="lg:col-span-1">
          <Card>
            <CardHeader>
              <CardTitle>Input</CardTitle>
            </CardHeader>
            <CardContent className="space-y-4">
              {/* Input fields */}
            </CardContent>
          </Card>
        </div>

        {/* Results section */}
        <div className="lg:col-span-2">
          <Tabs defaultValue="tab1">
            <TabsList>
              <TabsTrigger value="tab1">Tab 1</TabsTrigger>
              <TabsTrigger value="tab2">Tab 2</TabsTrigger>
            </TabsList>
            <TabsContent value="tab1">
              {/* Tab content */}
            </TabsContent>
          </Tabs>
        </div>
      </div>
    </ToolsWrapper>
  );
}
```

**Key Component Patterns:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [truethari/OpensourceToolkit](https://github.com/truethari/OpensourceToolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
