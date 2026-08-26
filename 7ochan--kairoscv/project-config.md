---
trigger: always_on
description: **Project Name:** KairosCV - AI-Powered Resume Optimization Platform
---

# KairosCV - AI Agent Performance Optimization Guide

## 🎯 Project Overview

**Project Name:** KairosCV - AI-Powered Resume Optimization Platform
**Current Status:** Transition to HTML-to-PDF approach (Branch: bharath-013)
**Goal:** Build an MVP that converts any resume format into ATS-optimized PDFs using AI enhancement

**CRITICAL:** This project uses **Next.js 16 + TypeScript**, NOT FastAPI/Python. The stack has evolved from the original design.

---

## 📐 Current Architecture (ACCURATE AS OF NOV 2025)

### Actual Technology Stack
```
Framework:     Next.js 16 (App Router)
Language:      TypeScript
Runtime:       Node.js 18+
Package Mgr:   pnpm
UI:            React 19 + Radix UI + Tailwind CSS
AI:            Google Gemini 1.5 Flash API
Parsers:       pdf-parse, mammoth, pdf-lib
PDF Gen:       Puppeteer (NEW - replacing pdf-lib)
Templates:     Handlebars (NEW)
Validation:    Zod (NEW)
Hosting:       Render.com
```

### Actual File Structure
```
kairosCV/
├── app/
│   ├── api/
│   │   ├── upload/route.ts          # File upload endpoint
│   │   ├── stream/[fileId]/route.ts # SSE progress streaming
│   │   ├── download/[fileId]/route.ts # PDF download
│   │   └── health/route.ts          # Health check
│   ├── page.tsx                      # Main UI
│   └── layout.tsx                    # App layout
├── lib/
│   ├── resume-processor.ts          # Main processing pipeline
│   ├── file-storage.ts              # File I/O utilities
│   ├── parsers/                     # NEW - Enhanced parsers
│   ├── ai/                          # NEW - Gemini integration
│   ├── templates/                   # NEW - HTML templates
│   ├── pdf/                         # NEW - Puppeteer generator
│   └── schemas/                     # NEW - Zod schemas
├── components/                       # React UI components
├── public/                          # Static assets
└── uploads/                         # Temporary file storage
```

### Data Flow (CURRENT)
```
User Upload → Next.js API Route → File Validation →
Parse (PDF/DOCX/TXT) → Extract Sections →
Gemini AI Enhancement → HTML Template Population →
Puppeteer PDF Generation → Download Link (SSE Progress Updates)
```

---

## 🚀 AI Agent Performance Optimization Rules

### Rule 1: Always Read Files Before Editing
**WHY:** Next.js routing and file structure must be preserved exactly
**HOW:** Always use the Read tool before Edit or Write tools
**EXAMPLE:**
```typescript
// ❌ BAD: Writing without reading first
Write("app/api/new-endpoint/route.ts", content)

// ✅ GOOD: Read existing patterns, then create similar
Read("app/api/upload/route.ts")
// ... then create new endpoint following the same pattern
```

### Rule 2: Use Type-Safe Patterns
**WHY:** TypeScript catches errors at compile time, not runtime
**HOW:** Always define types/interfaces before implementation
**EXAMPLE:**
```typescript
// ✅ Define schema first with Zod
export const ResumeDataSchema = z.object({
  contact: ContactSchema,
  experience: z.array(ExperienceSchema),
  // ...
})

export type ResumeData = z.infer<typeof ResumeDataSchema>

// Then use in functions
function processResume(data: ResumeData): Promise<Buffer> {
  // TypeScript will validate all property access
}
```

### Rule 3: Follow Next.js App Router Conventions
**WHY:** Incorrect routing breaks the entire application
**HOW:** Use exact Next.js 16 patterns for API routes and pages

**API Route Pattern:**
```typescript
// app/api/endpoint/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  // Set runtime to nodejs for server-side features
  export const runtime = 'nodejs'

  try {
    const data = await request.json()
    // ... process
    return NextResponse.json({ success: true })
  } catch (error) {
    return NextResponse.json(
      { error: 'Error message' },
      { status: 500 }
    )
  }
}
```

**Dynamic Route Pattern:**
```typescript
// app/api/download/[fileId]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { fileId: string } }
) {
  const { fileId } = params
  // ... process
}
```

### Rule 4: Implement Progressive Enhancement
**WHY:** Users should see progress, not blank screens
**HOW:** Use Server-Sent Events (SSE) for real-time updates

**SSE Pattern (already implemented):**
```typescript
// Create ReadableStream for SSE
const stream = new ReadableStream({
  async start(controller) {
    const send = (data: object) => {
      const message = `data: ${JSON.stringify(data)}\n\n`
      controller.enqueue(encoder.encode(message))
    }

    // Send progress updates
    send({ stage: 'parsing', progress: 20, message: 'Parsing resume...' })
    // ... process
    send({ stage: 'complete', progress: 100, download_url: '/api/download/...' })

    controller.close()
  }
})

return new Response(stream, {
  headers: {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive'
  }
})
```

### Rule 5: Validate All User Input
**WHY:** Security and data integrity are critical
**HOW:** Use Zod schemas for validation, sanitize all text

**Validation Pattern:**
```typescript
// Define schema
const UploadSchema = z.object({
  file: z.instanceof(File),
  // ...
})

// Validate in API route

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [7ochan/KairosCV](https://github.com/7ochan/KairosCV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
