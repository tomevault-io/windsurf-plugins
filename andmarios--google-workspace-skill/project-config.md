---
trigger: always_on
description: Manage Google Workspace with Docs, Sheets, Slides, Drive, Gmail, Calendar, and Contacts. Create professional documents, engaging presentations, reports from markdown. Convert markdown to Google Docs/Slides/PDF. Full editing, formatting, file management, email, and scheduling.
---


# Google Workspace Skill

Manage Google Workspace documents, spreadsheets, presentations, drive files, emails, calendar events, and contacts via CLI.

## Purpose

**Google Docs:** Read, create, export (markdown/pdf/docx/txt/html/rtf/epub/odt), insert/append text, find-replace, format (text, paragraph, extended), tables (insert, style, merge, row/column ops), headers/footers, lists/bullets, page breaks, section breaks, document styling, images, named range replacement

**Google Sheets:** Read, create, write/append data, full cell formatting (fonts, colors, alignment, number formats), borders, merge/unmerge cells, row/column sizing, freeze panes, conditional formatting, move rows/columns, copy-paste, auto-fill, trim whitespace, text-to-columns, chart updates

**Google Slides:** Read, create presentations, add/delete slides, text boxes, images, full text formatting (fonts, colors, effects, superscript/subscript, links), paragraph formatting (alignment, spacing, indentation), shapes (create and style), tables, element transforms (scale/rotate), grouping, alt text, Sheets chart embedding

**Google Drive:** Upload, download, search, share, create folders, move, copy, delete, comments, replies, shared drives, change tracking, revision management

**Gmail:** List, read, send, reply, search emails, history sync, batch label operations, label management

**Calendar:** List calendars, create/update/delete calendars, events, move events, color definitions, subscriptions

**Contacts:** List, create, update, delete contacts, groups, photos, directory search (Workspace), batch operations

**Convert:** Markdown to Google Docs, Slides, or PDF

## When to Use

- User requests to read, create, or edit a Google Doc, Sheet, or Slides presentation
- User wants to upload, download, search, or share Drive files
- User wants to send, read, or search emails
- User wants to create or manage calendar events
- User wants to manage contacts
- User wants to convert Markdown to Google formats
- Keywords: "Google Doc", "spreadsheet", "presentation", "slides", "Drive", "upload", "share", "email", "calendar", "contacts"

## Quick Start: Common Workflows

### Create a professional document from markdown
```bash
uvx gws-cli convert md-to-doc /path/to/file.md -t "Document Title"
```

### Create or enhance documents with rich content
When creating documents from scratch or enhancing converted documents, use all available tools:
- **Image generation** (DALL-E, etc.) - Create illustrations, diagrams, or infographics
- **Diagram rendering** - Use `--render-diagrams` flag or generate via Kroki
- **Tables** - Structure data clearly with `insert-table` and styling
- **Charts/visualizations** - Generate and insert as images

```bash
# Insert image into document
uvx gws-cli docs insert-image $DOC_ID "https://example.com/image.png" --index 50

# Or use diagram rendering during conversion
uvx gws-cli convert md-to-doc report.md -t "Report" --render-diagrams
```

### Create an engaging presentation (manual approach recommended)
```bash
# 1. Create presentation
uvx gws-cli slides create "Presentation Title"

# 2. Add slides with layouts (TITLE, TITLE_AND_BODY, SECTION_HEADER, etc.)
uvx gws-cli slides add-slide $PRES_ID --layout TITLE_AND_BODY

# 3. Read to get element IDs
uvx gws-cli slides read $PRES_ID

# 4. Insert text into elements
uvx gws-cli slides insert-text $PRES_ID $ELEMENT_ID "Your content"

# 5. Apply styling
uvx gws-cli slides set-background $PRES_ID $SLIDE_ID --color "#1A365D"
uvx gws-cli slides format-text $PRES_ID $ELEMENT_ID --bold --font-size 24
```

### Slide content limits (see [SKILL-advanced.md](SKILL-advanced.md) for design best practices)
- Maximum 6 bullet points per slide
- Maximum 6 words per bullet
- Under 40 words total per slide
- One idea per slide

### Enhance presentations with visuals
Great presentations use **images, diagrams, charts, and infographics** to communicate ideas effectively. Use all available tools:
- **Image generation** (DALL-E, etc.) - Create custom illustrations, icons, or backgrounds
- **Diagram tools** (Mermaid, PlantUML) - Render flowcharts, architecture diagrams, timelines
- **Charts from data** - Visualize metrics and trends
- **Screenshots/mockups** - Show products, interfaces, or examples

Insert visuals with:
```bash
uvx gws-cli slides insert-image $PRES_ID $SLIDE_ID "https://example.com/image.png" \
    --x 100 --y 100 --width 400 --height 300
```

### Send professional emails
```bash
# Simple email (short body as argument)
uvx gws-cli gmail send "recipient@example.com" "Subject" "Short message body"

# Multi-line email with heredoc (--stdin reads from pipe)
cat <<'EOF' | uvx gws-cli gmail send "recipient@example.com" "Meeting Follow-up" --stdin
Hi Team,

Following up on today's meeting. Key action items:

1. Review the proposal by Friday
2. Submit feedback via the shared doc
3. Schedule follow-up for next week

Best regards
EOF

# Plain text email (use --plain)
cat <<'EOF' | uvx gws-cli gmail send "recipient@example.com" "Status Update" --plain --stdin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andmarios/google-workspace-skill](https://github.com/andmarios/google-workspace-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
