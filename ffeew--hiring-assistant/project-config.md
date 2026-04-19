---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development

- `npm run dev` - Start development server with Turbopack
- `npm run build` - Build for production
- `npm start` - Start production server
- `npm run lint` - Run ESLint
- `npm run typecheck` - Run TypeScript type checking without emitting files

### Database

- `npx drizzle-kit generate` - Generate database migrations
- `npx drizzle-kit migrate` - Run database migrations
- `npx drizzle-kit studio` - Open Drizzle Studio for database inspection

## Architecture

This is a **Next.js 15 hiring assistant application** that automates resume processing and candidate email communication using AI.

### Core Components

**Authentication System**

- Uses Better Auth with Drizzle adapter for session management
- Database schema in `src/lib/db/schema.ts` defines user, session, account, and verification tables
- All API routes require authentication via session validation

**AI-Powered Resume Processing Pipeline**

- `src/app/api/extract/` - Follows 3-layer architecture with `extract.validator.ts`, `extract.service.ts`, and `route.ts`
- Mistral AI OCR integration for PDF/DOCX resume parsing with structured data extraction
- Supports bulk resume upload with concurrent file processing using Promise.allSettled
- Type-safe JSON handling for metadata parsing with `@/lib/json-utils.ts`
- Duplicate detection via file hashing with R2 storage integration

**Email Communication & Template System**

This application features a comprehensive email system with AI-powered template generation and intelligent template selection:

**Core Email Infrastructure**:

- `src/app/api/email/email.service.ts` - Nodemailer integration with Gmail SMTP
- **Optional Configuration**: Users can sign up without Gmail setup, configure later in profile
- **Validation Methods**: `EmailService.hasCompleteConfiguration()` and `EmailService.validateConfiguration()`
- **Rate Limiting**: 1-second delay between emails to avoid Gmail throttling
- Company branding support via user profile settings
- Email preview functionality before sending

**AI-Powered Template Management**:

- `src/app/api/email-templates/` - Complete CRUD operations following 3-layer architecture
- `src/app/api/email-templates/generate/` - AI template generation using Groq AI (GPT OSS 120B)
- `src/lib/db/schema.ts` - EmailTemplate table with comprehensive fields
- **AI Generation**: Natural language to professional email template conversion
- **Category-Aware**: Templates for acknowledgment, screening, interview, offer, rejection, follow_up
- **Multiple Tones**: Professional, friendly, formal, and casual writing styles
- **Template Editor**: Rich editing interface with live preview and click-to-insert variables
- **Usage Tracking**: Template usage statistics and last updated timestamps

**Dynamic Template Variables** (15+ variables from Mistral OCR extraction):

- `src/lib/template-engine.ts` - Template rendering engine
- **Candidate Data**: firstName, lastName, email, phone, jobPosition, companyName
- **Professional Links**: linkedinUrl, githubUrl, portfolioUrl
- **Resume Data**: skills (arrays), experience (formatted), education (formatted)
- **Automatic Variable Integration**: All variables available in template editor

**Intelligent Template Selection**:

- `src/app/api/extract/template-selection.service.ts` - Auto-assigns templates during resume processing
- **Default Priority**: Uses template marked with `isDefault: true` for category
- **First Template Fallback**: Automatically uses first available template when no default exists
- **Template Validation**: Frontend validates all extractions have templates before sending emails
- **User Guidance**: Clear warnings and actionable guidance when templates are missing

**Job Posts Management System**

- `src/lib/db/schema.ts` - Job post table with comprehensive fields (title, description, requirements, etc.)
- `src/app/api/job-posts/` - Complete CRUD API routes for job advertisements
- `src/app/job-posts/components/` - Job post management UI with React Hook Form validation
- Integration with email workflow for position selection
- Support for job post status management (active/inactive)

**Database Layer**

- Drizzle ORM with LibSQL/Turso as the database provider
- Configuration in `drizzle.config.ts` with Turso credentials
- Connection management in `src/lib/db/db.ts`

### Key Architectural Patterns

**API Route Architecture (MANDATORY PATTERN)**
This application follows a strict 3-layer architecture for all API routes. **ALL new API routes MUST follow this pattern:**

1. **Validator Layer** (`*.validator.ts`)

   - Contains Zod schemas for request validation (query, body, params)
   - Reuses existing schemas from `src/app/types/index.ts` when possible
   - Example: `getApplicantsQuerySchema`, `createApplicantBodySchema`, `applicantParamsSchema`

2. **Service Layer** (`*.service.ts`)

   - Contains ALL business logic and database operations
   - Static class methods for consistency
   - Handles transactions, error throwing, data transformation
   - Example: `ApplicantsService.getApplicants()`, `JobPostsService.createJobPost()`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ffeew) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
