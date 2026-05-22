---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**OneFile** is a free, open-source web application that combines multiple files into a single AI-ready prompt. It solves the common frustration of file upload limits on AI platforms like ChatGPT, Claude, and Gemini by allowing users to upload entire folders, documents, and code projects, then export them as a single formatted text file.

**Architecture**: Next.js 15 standalone application with server-side document processing

## Common Commands

### Development
- `npm run dev` - Start the Next.js development server (localhost:3000)
- `npm install` - Install all dependencies

### Build & Deploy
- `npm run build` - Build the Next.js application for production
- `npm start` - Start production Next.js server
- `npm run preview` - Build and preview Cloudflare deployment
- `npm run deploy` - Build and deploy to Cloudflare Pages

### Code Quality
- `npm run lint` - Run ESLint on the codebase
- `npm run cf-typegen` - Generate TypeScript types for Cloudflare environment

## Technology Stack

### Frontend & Backend
- **Next.js 15** with App Router (Server Components by default)
- **React 19** for UI components
- **TypeScript** for type safety
- **Tailwind CSS** for styling
- **shadcn/ui** for UI component library
- **Framer Motion** for animations

### Document Processing (Server-Side)
- **pdfjs-serverless** for PDF text extraction
- **mammoth** for DOCX document processing
- **JSZip** for PPTX slide extraction
- **xlsx** for Excel spreadsheet parsing
- **csv-parse** for CSV file parsing

### Deployment
- **Cloudflare Pages** with OpenNext adapter
- **Next.js Edge Runtime** compatible

### Key Features
1. **Universal File Support**: PDFs, Word docs, PowerPoint, Excel, CSV, code files, markdown, and more
2. **Smart Filtering**: Automatically ignores binary files, dependencies (node_modules, .git), and build artifacts
3. **GitIgnore Support**: Respects .gitignore files in uploaded directories
4. **Drag & Drop**: Support for both individual files and entire folder structures
5. **Client-Side Processing**: Text file extraction happens in the browser for speed
6. **Server-Side Document Processing**: Complex formats (PDF, DOCX, PPTX, XLSX) processed via API route

## File Structure

```
src/
├── app/                        # Next.js App Router
│   ├── layout.tsx             # Root layout with theme provider
│   ├── page.tsx               # Main application page
│   ├── about/                 # About page
│   ├── api/
│   │   └── extract-text/      # Server-side document processing API
│   │       └── route.ts       # Handles PDF, DOCX, PPTX, XLSX extraction
│   └── sitemap.ts             # SEO sitemap generation
├── components/
│   ├── FileUpload.tsx         # Drag & drop upload interface
│   ├── FileList.tsx           # Display uploaded files
│   ├── FilePreview.tsx        # Preview individual file content
│   ├── InfoDialog.tsx         # About/info modal
│   ├── theme-provider.tsx     # Dark/light theme support
│   ├── theme-toggle.tsx       # Theme switcher button
│   ├── magicui/
│   │   └── confetti.tsx       # Confetti animation component
│   ├── icons/                 # Social media icons
│   └── ui/                    # shadcn/ui components
├── lib/
│   └── utils.ts               # Utility functions (cn, etc.)
├── utils/
│   └── files.ts               # Core file processing logic
│       ├── processFile()      # Extract content from files
│       ├── processEntry()     # Process FileSystemEntry objects
│       ├── GitignoreParser    # Parse and apply .gitignore rules
│       ├── isPathIgnored()    # Check if path should be filtered
│       └── generatePromptText() # Format files into final output
├── constants/
│   └── files.ts               # File type configurations
│       ├── ALLOWED_EXTENSIONS # Supported file extensions
│       ├── ALLOWED_MIME_TYPES # Supported MIME types
│       └── IGNORED_PATHS      # Directories to skip
└── types/
    └── index.ts               # TypeScript type definitions
```

## Core Functionality

### File Processing Flow

1. **Upload**: User drags/drops files or folders, or uses file/folder picker
2. **Filtering**:
   - Check against `IGNORED_PATHS` (node_modules, .git, dist, etc.)
   - Parse and apply .gitignore rules if present
   - Validate file extensions and MIME types
   - Skip images and binary files
3. **Processing**:
   - **Text files**: Read directly in browser using FileReader
   - **Documents** (PDF/DOCX/PPTX/XLSX): Send to `/api/extract-text` for server-side processing
4. **Output**: Combine all file contents into single formatted text with file paths
5. **Export**: Copy to clipboard or download as `onefile-prompt.txt`

### GitIgnore Support

The application includes a custom `GitignoreParser` class that:
- Automatically detects and parses .gitignore files in uploaded directories
- Applies gitignore patterns with proper wildcard support
- Handles negation patterns (!)
- Tracks skipped files and provides user feedback

### Document Processing API

**Endpoint**: `POST /api/extract-text`

Processes complex document formats server-side:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wahibonae/onefile](https://github.com/wahibonae/onefile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
