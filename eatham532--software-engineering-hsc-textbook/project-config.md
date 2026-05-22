---
trigger: always_on
description: This is an **educational MkDocs site** for NSW HSC Software Engineering, structured as a hierarchical textbook with interactive elements. The project is **90% AI-generated content** with human editing for accuracy.
---

# Copilot Instructions for NSW HSC Software Engineering Textbook

## Project Architecture Overview

This is an **educational MkDocs site** for NSW HSC Software Engineering, structured as a hierarchical textbook with interactive elements. The project is **90% AI-generated content** with human editing for accuracy.

### Core Structure Pattern

```

docs/Year{11|12}/{ModuleName}/Chapter-{XX}-{Name}/Section-{YY}-{Name}/
├── index.md    # Main content with PlantUML diagrams, code examples
└── quiz.md     # Interactive assessments (6-10 questions each)

```

**Critical**: All navigation is controlled by `docs/.nav.yml` via mkdocs-awesome-nav. **Never** add manual `nav:` sections to `mkdocs.yml`.

### CRITICAL: Length Management for AI Contributors

Before writing substantial content, AI contributors MUST:

1. **Estimate content length** based on learning objectives and code examples required

2. **Check if content will exceed reasonable response limits** (typically ~1000-2000 lines)

3. **If content will be large, inform the user BEFORE starting** and propose breaking into manageable chunks

4. **Plan chunk boundaries** logically (by major topics, not arbitrary cuts)

5. **Update todo lists** to reflect multi-part approach

### Content Creation Standards

**ModulePlan.md Pattern**: Each module has a plan defining learning outcomes and section structure:

- Outcomes map to NSW syllabus codes (e.g., `SE-12-06`)

- Author deliverables: `index.md` + `quiz.md` per section

- Folder naming: `Chapter-{Zero-padded}-{Kebab-case}` / `Section-{Zero-padded}-{Kebab-case}`

**Section Content Requirements**:

- Educational explanations with practical Python examples (beginner-first, one idea at a time)

- PlantUML diagrams for system modeling (wrapped by custom `kroki_wrapper.py`)

- Interactive quizzes using `!!! quiz` admonition blocks

- Syllabus-aligned terminology and learning outcomes

- **Python only** - avoid external packages unless essential

## Development Workflows

### Local Development

```powershell
# Primary development command (use this, not raw mkdocs serve)
.\scripts\serve.ps1

```

This script sets `PYTHONPATH` to repo root for custom Python extensions and activates the venv.

**Before running Python code**: Always activate the project virtual environment with `.venv\Scripts\activate`

### Build & Validation

```bash
python scripts/smoke_test.py  # Validates build + diagram integration
python scripts/fix_quiz_format.py  # Standardizes quiz formatting across Year12
python scripts/simple_md_lint.py  # Basic Markdown linting and formatting

```

### Python Environment

- **uv** for dependency management (`uv sync` to install)

- Python 3.13+ target (but 3.x compatible)

- Custom markdown extensions in project root (`kroki_wrapper.py`)

- Typical development sequence:

  1. Activate virtual environment: `.venv\Scripts\activate`

  2. Install dependencies: `uv pip install -e .` (or `pip install -e .`)

  3. Serve locally: `mkdocs serve`

## Custom Integrations

### Diagram System

- **kroki_wrapper.py**: Post-processes Kroki SVGs to add modal expand buttons

- PlantUML blocks become clickable diagrams with `🔍 View Larger` functionality

- Diagrams use `diagram-container` CSS class with modal JavaScript

### Interactive Quiz System

- **Custom admonition**: `!!! quiz "Quiz Title"`

- **Format**: Ordered list questions with unordered list options

- **Markers**: `{data-correct}` in option text (auto-converted to attributes)

- **Multi-select**: Detected when multiple options marked correct

- **JavaScript**: `quiz.js` handles interaction, grading, visual feedback

- **IMPORTANT**: Use Markdown-only format, NO HTML divs, scripts, or complex structures

### Quiz Formatting Standards

```markdown
!!! quiz "Section X.Y Quiz: Topic"
    
    1. Question text here?
        - Option A
        - Option B {data-correct}
        - Option C
        - Option D

```

**Quiz Debugging**: If answers show "Try again" when correct:

1. Hard-refresh (Ctrl+F5) to ensure latest `quiz.js` loads

2. Set `window.__quiz_debug = true` in browser console for debug logs

3. Use browser console snippet from AI_README.md to inspect quiz state

## Content Structure & Authoring Standards

### Section Template (index.md)

Required structure:

- **H1**: Section title (short and numbered, e.g., "1.1 Software development steps")

- **H2**: Why it matters

- **H2**: Concepts

  - **H3**: Concept name (short)

  - **H3**: Guided example

- **H2**: Try it (or "Practice" for non-coding sections)

- **H2**: Recap

### Authoring Style Rules

- **Syllabus outcomes**: NEVER add "Outcomes: SE-XX-XX" lines to section files (belongs in ModulePlan.md only)

- **Cross-references**: Include "See also [section]" to connect concepts across modules

- **Paragraphs**: Keep short (1-3 sentences)

- **Lists**: Use spaces not tabs for proper indentation

- **Voice**: Third-person neutral, Australian spelling, plain language

- **Examples**: Progressive - start with REPL snippets, grow to small scripts

### PlantUML Diagrams


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eatham532/Software-Engineering-HSC-Textbook](https://github.com/Eatham532/Software-Engineering-HSC-Textbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
