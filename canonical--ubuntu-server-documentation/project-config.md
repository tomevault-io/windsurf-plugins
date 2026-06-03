---
trigger: always_on
description: This is the **Ubuntu Server documentation** – a Sphinx-based documentation project using the [Canonical Sphinx Docs Starter Pack](https://github.com/canonical/sphinx-docs-starter-pack). Documentation targets the latest Ubuntu LTS release, calling out version-specific differences when they exist.
---

# Ubuntu Server Documentation – Copilot Instructions

## Project Overview

This is the **Ubuntu Server documentation** – a Sphinx-based documentation project using the [Canonical Sphinx Docs Starter Pack](https://github.com/canonical/sphinx-docs-starter-pack). Documentation targets the latest Ubuntu LTS release, calling out version-specific differences when they exist.

## Architecture & Structure

### Diátaxis Framework

Content is organized using the [Diátaxis framework](https://diataxis.fr/):
- Tutorials (`tutorial/`) – Getting started guides (e.g., `basic-installation.md`)
- How-to Guides (`how-to/`) – Task-oriented guides organized by topic (installation, security, networking, virtualisation, etc.)
- Explanations (`explanation/`) – Conceptual overviews and background information
- Reference (`reference/`) – Technical specifications, glossaries, system requirements
- `contributing/` – Contributor documentation

### File Types

- **Markdown (`.md`)** – Primary format for content pages with MyST syntax support

### Key Configuration Files

- `conf.py` – Project-specific settings (edit here for customization)
- `Makefile` – Build system
- `.readthedocs.yaml` – Read the Docs build configuration

## Development Workflow

### Local Setup

```bash
git clone git@github.com:canonical/ubuntu-server-documentation.git
cd ubuntu-server-documentation
sudo apt update
sudo apt install make python3 python3-venv python3-pip
make install
```

### Build & Preview

```bash
make run          # Build, watch, and serve at http://127.0.0.1:8000
make html         # Build only
make serve        # Serve only
make clean-doc    # Clean built files
```

### Testing & Quality Checks

```bash
make spelling     # Spell check (uses .custom_wordlist.txt for exceptions)
make linkcheck    # Verify all links
make woke         # Check inclusive language
make pa11y        # Accessibility testing
```

Always run `make spelling` and `make linkcheck` before submitting PRs.

## Writing Conventions

### Style & Language

- Follow the [Canonical documentation style guide](https://docs.ubuntu.com/styleguide/en)
- Use **US English** (`en-US`)
- Acronyms: Expand on first use, e.g., "Yet Another Markup Language (YAML)"
- Add technical terms to `reference/glossary.rst` and spelling exceptions to `.custom_wordlist.txt`

### Versioning Strategy

This project does **not** use separate branches per Ubuntu release. Instead:
- Default content applies to all supported LTS releases
- Version-specific content uses **note admonitions**:
  ```markdown
  :::{note}
  For Ubuntu 24.04 LTS (Noble) onwards, use the new method...
  :::
  ```
- Major version differences use **MyST tabs** with sync keywords (e.g., `24.04`, `22.04`)
- Order tabs from **newest to oldest** Ubuntu release

### Cross-References

- Use reference labels in `.md` files: `(slug-name)=`
- Link with {ref}`my-label`
- First mentions of packages/tools should link to official docs or manpages
- Use semantic markup: {kbd}`Ctrl`, {manpage}`dpkg(1)`, {term}`DAC`
- Manpage links auto-generate URLs (no hardcoding needed)

### Markdown Elements

- **Headings**: Use proper hierarchy (`#`, `##`, `###`, `####`); don't skip levels
- **Lists**: Use `1.` for all numbered items (auto-renumbers)
- **Code blocks**: Specify language for syntax highlighting
- **MyST extensions**: `colon_fence`, `deflist`, `linkify`, `substitution`

### Sphinx Roles & Semantic Markup

Every technical entity must use a specific MyST role:
- **Keyboard keys:** Must use {kbd}`Key` (e.g., {kbd}`Enter`).
- **Command-line tools:** Use {manpage}`tool(section)` for the first mention (e.g., {manpage}`ls(1)`).
- **Glossary terms:** Use {term}`term` for items defined in the glossary.
- **Internal Links:** Use {ref}`label-name` instead of standard markdown links for internal cross-references.

## Critical Patterns

### Redirects

When renaming/moving/deleting files, **always add redirects**:

**Internal redirects** (in `redirects.txt`):
```
old/path/to/file/ new/path/to/file/
```

**External redirects** (in `conf.py` under `redirects = {}`):
```python
redirects = {
    "how-to/containers/lxc-containers": "https://linuxcontainers.org/lxc/documentation/",
}
```

### Custom Wordlist

Add valid technical terms/acronyms to `.custom_wordlist.txt` (alphabetically sorted) rather than wrapping in backticks, unless you want monospaced rendering.

### PR Requirements

- Link PRs to issues with `Fixes: #<issue-number>` in description
- Use [Conventional Comments](https://conventionalcomments.org/) for feedback
- Include manual testing for code/commands
- Preview builds available via Read the Docs check on PR

## Common Tasks

### Adding New Content

1. Determine Diátaxis category (tutorial/how-to/explanation/reference)
2. Create `.md` file in appropriate subdirectory
3. Add to corresponding `index.md` or section landing page
4. Use reference labels: `(my-label)=` at top of Markdown files
5. Test: `make run` and verify navigation

### Updating Links

- Prefer reputable sources (official upstream docs, not blog posts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [canonical/ubuntu-server-documentation](https://github.com/canonical/ubuntu-server-documentation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
