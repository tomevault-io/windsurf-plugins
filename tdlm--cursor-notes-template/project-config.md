---
trigger: always_on
description: All meeting notes files in `02-Meetings/` must follow this format:
---

# Notes Organization Rules

## Meeting Notes Naming Convention

All meeting notes files in `02-Meetings/` must follow this format:

**Format:** `[YYYY]-[MM]-[DD]-[title].md`

**Examples:**
- `2025-01-15-team-standup.md`
- `2025-02-10-client-discussion.md`
- `2025-03-20-coaching-session.md`

**Rules:**
- Date format: YYYY-MM-DD (ISO 8601)
- Date comes first for chronological sorting
- Title should be lowercase with hyphens separating words
- Title should be descriptive but concise
- No spaces in filenames (use hyphens instead)

## File Organization

- **00-Inbox/**: Single inbox for all files awaiting processing
- **01-General/**: General notes and references
- **02-Meetings/**: **ALL meeting notes** (follow naming convention above)
  - Organized by year and month: `02-Meetings/[YYYY]/[MM]/`
  - All business meetings, personal meetings, discussions, and coaching sessions
  - Business-related meeting notes go here (NOT in `08-Business/`)
- **03-Emails/**: Important personal and business emails
  - Organized by year and month: `03-Emails/[YYYY]/[MM]/`
- **04-Documents/**: PDFs, contracts, receipts, statements converted to Markdown
  - Organized by year and month: `04-Documents/[YYYY]/[MM]/`
- **05-People/**: Notes about people in your life (contacts, relationship notes, etc.)
  - Reference materials about people
  - Meeting notes about people should go to `02-Meetings/`
- **06-Projects/**: Project-specific documentation
  - Side projects and personal projects
- **07-Ideas/**: Ideas, brainstorms, and concepts (see Idea Processing section below)
- **08-Business/**: Business-related **information and documentation only** (NOT meeting notes)
  - Reference materials, business plans, ongoing documentation
  - Meeting notes about business should go to `02-Meetings/`
- **09-Personal/**: Personal reference documents only (NOT meeting notes)
  - Career planning, learning plans, etc. (non-meeting reference materials)
  - Meeting notes go to `02-Meetings/`
  - **recipes/**: Recipe collection (timeless reference materials, no date-based folders)

## Inbox Processing

**IMPORTANT:** When files are present in `00-Inbox/`, automatically process them without requiring explicit user request.

### 00-Inbox Processing

When processing files in `00-Inbox/`:
- **Automatically process all files** (except `README.md` which stays in inbox)
- **PDF/DOCX files**: Convert to Markdown first using conversion scripts:
  - PDF files → `node scripts/convert_pdf.js <file> --output-dir 00-Inbox/`
  - DOCX files → `node scripts/convert_docx.js <file> --output-dir 00-Inbox/`
  - Original files are deleted after successful conversion
- **PDF/DOCX files in attachment folders**: Convert to Markdown using conversion scripts:
  - PDF files → `node scripts/convert_pdf.js <file> --output-dir <attachment-folder>/`
  - DOCX files → `node scripts/convert_docx.js <file> --output-dir <attachment-folder>/`
  - Original files are deleted after successful conversion
  - **IMPORTANT**: All PDF and DOCX files in attachment folders must be converted to Markdown. Only Markdown files should remain in attachment folders after processing.
- **Markdown files**: Format for better readability:
  - Normalize spacing (consistent line breaks between sections)
  - Ensure proper heading hierarchy
  - Clean up excessive whitespace
  - Fix common markdown formatting issues
  - Improve list formatting consistency
  - Add appropriate spacing around code blocks, tables, etc.
- **Content detection**: Analyze files to determine appropriate destination based on keywords and content:
  - Check for keywords in filenames and content: "meeting", "email", "document", "project", "idea", "business", "personal", "recipe", etc.
  - Route files based on detected content type:
    - Meeting notes (dated discussions, syncs, sessions, keywords: "meeting", "call", "discussion", "session", "sync") → `02-Meetings/[YYYY]/[MM]/` with proper naming: `[YYYY]-[MM]-[DD]-[title].md`
    - Email content (keywords: "email", "message", "correspondence") → `03-Emails/[YYYY]/[MM]/` with proper naming: `[YYYY]-[MM]-[DD]-[title].md`
    - Documents (keywords: "document", "contract", "agreement", "letter", "statement", "receipt", "invoice") → `04-Documents/[YYYY]/[MM]/` with proper naming: `[YYYY]-[MM]-[DD]-[title].md`
    - Project documentation (keywords: "project", "task", "milestone") → `06-Projects/` (or date-based `06-Projects/[YYYY]/[MM]/` if applicable)
    - Ideas and concepts (keywords: "idea", "concept", "brainstorm", "what if", "app idea", "game idea", "business idea") → **Trigger Idea Flow** (see Idea Processing section)
    - Business reference materials (keywords: "business", "company", "corporate", "strategy") → `08-Business/[YYYY]/[MM]/` with proper naming: `[YYYY]-[MM]-[DD]-[title].md` (NOT meeting notes)
    - Personal reference materials (keywords: "personal", "career", "resume", "cv") → `09-Personal/[YYYY]/[MM]/` with proper naming: `[YYYY]-[MM]-[DD]-[title].md` (NOT meeting notes)
    - Recipes (keywords: "recipe", "cooking", "dish", "meal", "ingredients") → `09-Personal/recipes/` with descriptive filename (no date prefix, e.g., `chicken-tikka-masala.md`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tdlm/cursor-notes-template](https://github.com/tdlm/cursor-notes-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
