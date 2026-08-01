---
trigger: always_on
description: Generates complete solutions for a chapter/section including:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based static website that provides solutions to exercises and problems from "Introduction to Algorithms" (3rd edition) by Cormen, Leiserson, Rivest, and Stein (CLRS). The site is deployed to GitHub Pages at https://atekihcan.github.io/CLRS/.

## Development Commands

### Building and Testing Locally

```bash
# Install dependencies (first time only)
bundle install

# Start development server with live reload (recommended)
./serve.sh
# OR manually:
bundle exec jekyll serve --livereload

# Build the site for production (output to _site/)
./build.sh
# OR manually:
bundle exec jekyll build
```

**Live Reload Features:**
- Automatically rebuilds site when files change
- Automatically refreshes browser on changes
- No need to manually rebuild or refresh
- Server runs at: http://127.0.0.1:4000/CLRS/

### Deployment

The site automatically deploys via GitHub Actions on every push to the master branch. See `.github/workflows/github-pages.yml` for the workflow configuration.

### Git Workflow for New Solutions

**Always work in a feature branch:**

```bash
# Create a new branch for the chapter/section
git checkout -b solutions/chapter-X-section-Y

# After generating solutions, commit
git add .
git commit -m "Add solutions for Chapter X, Section Y"

# Push the branch
git push origin solutions/chapter-X-section-Y

# Create a PR for review (or merge locally after testing)
```

**Branch Naming Convention:**
- Single section: `solutions/chapter-5-section-1`
- Multiple sections: `solutions/chapter-5-sections-1-2-3`
- Entire chapter: `solutions/chapter-5`
- Bug fixes: `fix/chapter-5-typo`

## Architecture

### Content Structure

**Solution Files** (`_solutions/`)
- Organized by chapter number (e.g., `01/`, `02/`, `03/`, etc.)
- Naming convention:
  - Exercises: `E[chapter].[section]-[number].md` (e.g., `E01.01-01.md`)
  - Problems: `P[chapter]-[number].md` (e.g., `P01-01.md`)
  - Appendix: `EA.[section]-[number].md` (e.g., `EA.01-01.md`)
- Each solution is a markdown file with YAML front matter containing:
  - `title`: Exercise/problem identifier
  - `published`: Original publication date
  - `modified`: Last modification date
  - `description`: The exercise/problem statement

**Table of Contents** (`_data/toc.json`)
- Structured JSON defining the book's chapter and section organization
- Contains metadata about exercise counts per section
- Used to generate navigation and ensure coverage

### Templates and Includes

**Layouts** (`_layouts/`)
- `base.html`: Main layout template
- `post.html`: Solution page layout (extends base)

**Includes** (`_includes/`)
- `code/`: Python code snippets for interactive execution (using Skulpt)
  - Organized by chapter (e.g., `code/01/`, `code/02/`)
  - Embedded in solutions for demonstration
- `graph/`: HTML files containing graph visualizations
- `analytics.html`: Google Analytics integration
- `comment.html`: Disqus comments integration
- Standard partials: `head.html`, `header.html`, `footer.html`, `aside.html`

### Assets

**Static Resources** (`assets/`)
- `css/`: Stylesheets (normalize, syntax highlighting, custom site styles)
- `js/`: JavaScript including Skulpt (in-browser Python execution)
- `img/`: Solution diagrams and illustrations organized by chapter
- `katex/`: LaTeX math rendering library

### Jekyll Configuration

Key settings in `_config.yml`:
- Uses kramdown with GFM (GitHub Flavored Markdown)
- Solutions are a Jekyll collection with custom permalink structure
- Base URL: `/CLRS`
- Plugins: `jemoji`, `jekyll-redirect-from`

## Solution Writing Guidelines

### Core Philosophy

**Intuition First, Mathematics Second**
- Start with conceptual understanding and practical examples
- Build intuition before diving into formal proofs
- Use plain language explanations accessible to learners
- Mathematical rigor comes after understanding "why"

### Content Structure

When creating solutions, follow this order:

1. **Problem Statement** - Quote the exact problem from the book
2. **Intuitive Explanation** - Explain the "why" in plain language with concrete examples
3. **Approach/Strategy** - Describe the solution approach
4. **Detailed Solution** - Step-by-step walkthrough
5. **Mathematical Analysis** - Formal proofs, complexity analysis (if needed)
6. **Implementation** - Code examples (if applicable)
7. **Edge Cases/Insights** - Use aside boxes for additional observations

### Writing Style

**Natural Flow - No Artificial Subheadings:**
- **DO NOT use subheadings like "Intuition:", "Recurrence:", "Solution:"**
- **DO NOT use em-dashes (—)** - use commas, parentheses, or split into separate sentences
- Let the text flow naturally in paragraph form
- Start each method/part with intuitive explanation before equations
- Use concrete analogies (e.g., dictionary, cooking, traffic, bank etc.)
- Section headers for multi-part problems: "### A.", "### B." (not "### Part (a):")

**Mathematical Formatting:**
- Use `\begin{align*}...\end{align*}` for multi-step equations
- Example:
  ```latex
  $$\begin{align*}
  f(n) &= \Theta(n) \\
       &= \Theta(n^{\log_2 2}) \\
       &= \Theta(n^{\log_b a})
  \end{align*}$$
  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Atekihcan/CLRS](https://github.com/Atekihcan/CLRS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
