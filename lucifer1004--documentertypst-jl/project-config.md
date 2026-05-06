---
trigger: always_on
description: **For AI Agents and Contributors**: This document describes the architecture, data flow, and code organization of DocumenterTypst.jl.
---

# AGENTS.md

**For AI Agents and Contributors**: This document describes the architecture, data flow, and code organization of DocumenterTypst.jl.

---

## Critical Rules

1. **Never break backward compatibility**: Any change that breaks existing user documentation builds is a bug.
2. **All user-visible changes require a changelog entry** in `CHANGELOG.md` under "Unreleased" section (unless marked "Skip Changelog").
3. **Code style**: Follow Julia standard conventions. Use `just format` (Runic for Julia, markdownlint-cli2 for Markdown) before committing.
4. **Test coverage**: Add tests for all new Markdown node types or Typst output features.
5. **Detect VCS correctly**: Always check for version control system before suggesting workflows (see "VCS Detection" below).

---

## VCS Detection (For AI Agents)

**Before suggesting any Git/Jujutsu commands**, detect the project's VCS:

### Detection Strategy

```bash
# Check for hidden VCS directories (list_dir tool doesn't show hidden files!)
ls -a | grep -E "^\.(git|jj|hg|svn)"
```

### Workflow Selection

| VCS Detected          | Workflow to Use                      | Example Commands                                   |
| --------------------- | ------------------------------------ | -------------------------------------------------- |
| `.jj` (Jujutsu)       | Jujutsu change-based                 | `jj new -m "msg"`, `jj describe`, `jj git push`    |
| `.git` only           | Git branch-based                     | `git checkout -b branch`, `git commit`, `git push` |
| Both `.jj` and `.git` | **Prefer Jujutsu** (explicit choice) | Jujutsu commands + note Git interop available      |

### Critical Notes

- **List tools may hide dotfiles**: Always use `ls -a` or equivalent terminal command
- **Jujutsu often coexists with Git**: If both present, prefer Jujutsu workflow but mention Git compatibility
- **This project uses**: Jujutsu (`.jj` present) with Git interop (`.git` also present)

---

## What This Project Does

**Input**: Documenter.jl's AST (Abstract Syntax Tree) from Markdown documentation
**Output**: Professional PDF via Typst typesetting system

**Core Value**: Compile large documentation projects to PDF in < 60 seconds vs several minutes with LaTeX.

---

## Architecture Overview

### Data Flow

```
User Markdown Files
  ↓
Documenter.jl (parsing, cross-references, etc.)
  ↓
Documenter.Document (unified AST with MarkdownAST nodes)
  ↓
TypstWriter.render_typst() (convert AST to Typst code)
  ↓
.typ file (Typst markup language)
  ↓
Typst Compiler (via Typst_jll / native)
  ↓
PDF Output
```

### Key Design Decision

DocumenterTypst is a **Documenter plugin**, not a standalone tool. It:

- Reuses Documenter's Markdown parsing, cross-reference resolution, and page management
- Only handles the "AST → Typst → PDF" conversion
- Follows the same plugin architecture as DocumenterVitepress.jl

---

## Code Organization

### Entry Point

- **`src/DocumenterTypst.jl`**: Module definition, exports `Typst` format struct. Minimal glue code.

### Core Logic

- **`src/TypstWriter.jl`**: The entire conversion engine
  - `Typst` struct: Configuration (platform, version, typst executable path)
  - `render_typst()`: Main orchestrator, generates `.typ` file and compiles to PDF
  - `convert()` methods: AST node → Typst code translation (50+ node types supported)
  - Helper functions: Label generation, escaping, path handling

### Typst Template

- **`assets/documenter.typ`**: Default Typst template defining PDF styling
  - Page layout (margins, headers, footers)
  - Typography (fonts, colors, spacing)
  - Admonition boxes (info, warning, danger, etc.)
  - Code block highlighting
  - Table of contents generation

**User Customization**: Users can override styles by providing `src/assets/custom.typ` in their docs project.

### Tests

- **`test/runtests.jl`**: Unit tests for all MarkdownAST node conversions
- **`test/integration/`**: Integration tests that compile full documentation projects
  - **`fixtures/`**: Test fixtures for different scenarios

### Documentation

- **`docs/src/manual/`**: User guides (getting started, configuration, styling, math support, troubleshooting)
- **`docs/src/examples/`**: Complete examples (advanced features, migration from LaTeX)
- **`docs/src/api/`**: API reference

---

## Key Data Structures

### `Typst` (Format Configuration)

```julia
struct Typst <: Documenter.Writer
    platform::String   # "typst" | "native" | "none"
    version::String    # Semantic version for filename (e.g., "1.0.0")
    typst::Union{Nothing, String, Cmd}  # Custom path to typst executable
    optimize_pdf::Bool  # Whether to run PDF optimization (via pdfcpu)
    use_system_fonts::Bool  # Allow system font lookup
    font_paths::Vector{String}  # Additional font directories
end
```

### Documenter's AST Nodes

DocumenterTypst handles 50+ MarkdownAST node types. Critical ones:

- **Structural**: `Document`, `Heading`, `Paragraph`, `List`, `BlockQuote`
- **Inline**: `Text`, `Code`, `Strong`, `Emph`, `Link`, `Image`
- **Advanced**: `CodeBlock`, `Table`, `Math`, `Admonition`, `Footnote`
- **Documenter-specific**: `@docs` blocks, `@example` blocks, cross-references

### Conversion Strategy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucifer1004/DocumenterTypst.jl](https://github.com/lucifer1004/DocumenterTypst.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
