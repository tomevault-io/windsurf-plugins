---
trigger: always_on
description: CO_OP_TRANSLATOR_METADATA:
---

<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "913da05fee7fb78699c0447cf6d8aa10",
  "translation_date": "2025-11-18T18:03:10+00:00",
  "source_file": "AGENTS.md",
  "language_code": "pcm"
}
-->
# AGENTS.md

## Project Overview

**Security-101** na beginner-friendly cybersecurity curriculum wey Microsoft create. Dis project na documentation-based learning resource wey dey teach basic cybersecurity concepts through structured modules. E no dey tied to any vendor and e dey designed make person fit complete am for small-small lessons (30-60 minutes each).

**Key Technologies:**
- Markdown for content
- Docsify for static site generation
- GitHub Pages for hosting
- Co-op Translator for multi-language support (50+ languages)
- GitHub Actions for CI/CD

**Architecture:**
- Educational content dey arranged inside 8 main modules, each one get sub-lessons
- Static HTML site wey Docsify dey render Markdown content
- Automated translation workflow wey dey use Azure AI services
- No need for build tools or package managers for core content

## Repository Structure

```
/
├── README.md                    # Main entry point with curriculum overview
├── index.html                   # Docsify site entry point
├── [1-8].[1-4] *.md            # Curriculum modules and lessons
├── CODE_OF_CONDUCT.md          # Community guidelines
├── SECURITY.md                 # Security policy
├── SUPPORT.md                  # Support information
├── LICENSE                     # MIT License
├── images/                     # Image assets for lessons
├── translated_images/          # Translated image assets
├── translations/               # Translated versions (50+ languages)
└── .github/
    ├── workflows/
    │   ├── co-op-translator.yml    # Automated translation workflow
    │   ├── deploy.yaml             # GitHub Pages deployment
    │   └── jekyll-gh-pages.yml     # Jekyll site generation
    └── ISSUE_TEMPLATE/             # Issue templates
```

## Setup Commands

Dis na documentation project wey no need any dependencies to install. To work with the content:

```bash
# Clone the repository
git clone https://github.com/microsoft/Security-101.git
cd Security-101

# View content locally - any Markdown viewer works
# OR serve with a simple HTTP server to use Docsify rendering
python -m http.server 8000
# Then visit http://localhost:8000 in your browser
```

## Development Workflow

### Viewing Content Locally

Dis project dey use Docsify for rendering. To preview changes:

```bash
# Option 1: Use Python's built-in HTTP server
python -m http.server 8000

# Option 2: Use Node.js http-server (if available)
npx http-server -p 8000

# Option 3: View Markdown files directly in any Markdown editor
```

### Content Structure

Modules dey numbered one after the other:
- **Module 1:** Basic security concepts (1.1-1.7)
- **Module 2:** Identity & access management (2.1-2.4)
- **Module 3:** Network security (3.1-3.4)
- **Module 4:** Security operations (4.1-4.4)
- **Module 5:** Application security (5.1-5.3)
- **Module 6:** Infrastructure security (6.1-6.3)
- **Module 7:** Data security (7.1-7.3)
- **Module 8:** AI security (8.1-8.4)

Each module dey end with quiz file (e.g., "1.7 End of module quiz.md").

### Making Content Changes

1. Edit Markdown files directly for root directory
2. Follow the naming style wey dey already: `[module].[lesson] [Title].md`
3. Update README.md table if you dey add/remove modules
4. Add images to `/images/` directory
5. Reference images using relative paths: `![Description](../../translated_images/filename.8c8067c683396b6f17b14be6dc5b8b323f661a863de696f36ec51e4813657b57.pcm.png)`

## Translation Workflow

**Automated Translation:**
- Translations dey handled automatically by the Co-op Translator GitHub Action
- When you push changes go `main` branch, the workflow dey translate content to 50+ languages
- Translated files dey stored inside `/translations/[language_code]/`
- Translation metadata dey preserved inside YAML frontmatter

**Supported Languages:** Arabic, Bengali, Bulgarian, Burmese, Chinese (Simplified, Traditional), Croatian, Czech, Danish, Dutch, Estonian, Finnish, French, German, Greek, Hebrew, Hindi, Hungarian, Indonesian, Italian, Japanese, Korean, Lithuanian, Malay, Marathi, Nepali, Norwegian, Persian, Polish, Portuguese, Punjabi, Romanian, Russian, Serbian, Slovak, Slovenian, Spanish, Swahili, Swedish, Tagalog, Tamil, Thai, Turkish, Ukrainian, Urdu, Vietnamese, and more.

**No dey manually edit translation files** - dem go overwrite am with the automated workflow.

## Code Style Guidelines

### Markdown Conventions

- Use standard Markdown syntax
- Headings: Use `#` for main title, `##` for sections, `###` for subsections
- Lists: Use `-` or `*` for unordered lists, `1.` for ordered lists
- Links: Use descriptive text with full GitHub URLs for cross-references
- Images: Store for `/images/` directory, use descriptive alt text
- Code blocks: Use triple backticks with language identifier when e dey necessary

### Content Guidelines

- Make lessons short and straight to the point (30-60 minute read time)
- Use clear, beginner-friendly language
- No dey use vendor-specific tool instructions (curriculum no dey tied to any vendor)
- Add learning objectives for the start of each module

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Security-101](https://github.com/microsoft/Security-101) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
