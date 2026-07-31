---
trigger: always_on
description: The **microsoft/community-content** repository provides prepared "content-in-a-box" presentation materials for user groups, meetups, and community events. This repository contains PowerPoint presentations, supporting documentation, demo resources, and marketing materials for technical talks on Microsoft technologies.
---

# AGENTS.md

## Project Overview

The **microsoft/community-content** repository provides prepared "content-in-a-box" presentation materials for user groups, meetups, and community events. This repository contains PowerPoint presentations, supporting documentation, demo resources, and marketing materials for technical talks on Microsoft technologies.

**Current Focus**: Season of AI - MCP (Model Context Protocol) - October 2025 to January 2026

**Key Technologies**: Documentation repository (Markdown, PowerPoint, Word documents)

**Repository Structure**:
- `Season-of-AI_MCP/` - Active season content (presentations, marketing kits, organizer kits)
- `archive/` - Previous seasons' content (Seasons 1-4: Azure AI, Copilots, Best of Ignite, Season of Agents)
- `assets/` - Images and visual resources
- Root documentation: README.md, CODE_OF_CONDUCT.md, SECURITY.md, LICENSE files

## Content Organization

### Active Content
- **Location**: `Season-of-AI_MCP/`
- **Contains**: PowerPoint presentations, explainer decks, marketing materials, organizer kits
- **Current Topic**: Model Context Protocol (MCP) - "Let's Learn MCP" session materials

### Archived Content
- **Location**: `archive/`
- **Seasons**:
  - S1: Azure AI topics
  - S2: Copilots and GitHub Copilot topics
  - S3: Best of Ignite highlights
  - S4: Season of Agents

### File Types
- `.pptx` - PowerPoint presentations (primary content format)
- `.md` - Markdown documentation and session descriptions
- `.docx` - Word documents with supporting materials
- `.png`, `.jpg` - Images and screenshots

## Development Workflow

### Adding New Content

1. **Determine Content Location**:
   - Active season content goes in `Season-of-AI_MCP/`
   - Archived content should remain in `archive/` subdirectories
   - Supporting assets go in `assets/`

2. **Content Standards**:
   - PowerPoint presentations should include speaker notes
   - Include README.md with session descriptions and demo links
   - Reference external demo repositories with aka.ms links
   - Maintain consistent branding and formatting

3. **File Naming**:
   - Use descriptive, kebab-case names for markdown files
   - PowerPoint files may use title case with spaces
   - Avoid special characters except hyphens and underscores

### Repository Conventions

- **No Build Process**: This is a documentation repository with no compilation or build steps
- **No Dependencies**: No package.json, requirements.txt, or dependency management files
- **No Testing Framework**: Content is manually reviewed
- **Version Control**: Use Git for tracking changes to presentations and documentation

## Content Guidelines

### Presentation Materials

- **Format**: PowerPoint (.pptx) is the primary format
- **Duration**: Sessions designed for 30-45 minutes plus Q&A
- **Speaker Notes**: Include detailed notes for presenters
- **Customization**: Content can be remixed and adapted by speakers
- **Branding**: Maintain Microsoft Season of AI branding

### Documentation

- **README Files**: Each content folder should have a README.md describing available sessions
- **Session Descriptions**: Include:
  - Session title and overview
  - Target audience level (beginner, intermediate, advanced)
  - Links to supporting demo code repositories
  - Links to presentation slides
  - Additional resources and materials

### Supporting Materials

- **Marketing Kits**: Located in `Season-of-AI_MCP/Marketing Kit/`
- **Organizer Kits**: Located in `Season-of-AI_MCP/Organizer Kit/`
- **QR Codes**: Include attendee survey QR codes in presentations
- **Assets**: Store reusable images and graphics in `assets/`

## External Resources

### Demo Code Repositories

Demo code is maintained in separate repositories and linked via aka.ms URLs:
- C# demos: https://aka.ms/letslearnmcp-csharp
- Java demos: https://aka.ms/letslearnmcp-java
- JavaScript demos: https://aka.ms/letslearnmcp-javascript
- Python demos: https://aka.ms/letslearnmcp-python

### Event Registration

- Event registration: https://aka.ms/soai/mcp/registerevent
- Developer community: https://aka.ms/soai/mcp/devcom
- Attendee survey: https://aka.ms/soai/mcp/attendeesurvey
- FAQs: https://aka.ms/soai/mcp/faq

## Contributing

### Pull Request Guidelines

1. **Title Format**: Use clear, descriptive titles
   - Example: "Add new MCP session materials"
   - Example: "Update Season 5 marketing kit"

2. **Changes to Active Content**:
   - Update `Season-of-AI_MCP/` for current season materials
   - Update README.md to reflect new or modified content
   - Ensure all links are functional and use aka.ms shortcuts where appropriate

3. **Archiving Content**:
   - When a season ends, content moves to `archive/`
   - Maintain README files in archived directories
   - Update root README.md to reflect current season

4. **Required Checks**:
   - Verify all markdown files render correctly
   - Test all hyperlinks (aka.ms links and external URLs)
   - Review PowerPoint presentations for completeness
   - Ensure branding and formatting are consistent


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/community-content](https://github.com/microsoft/community-content) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
