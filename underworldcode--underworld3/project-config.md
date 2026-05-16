---
trigger: always_on
description: > **Note**: Human-readable developer documentation is in `docs/developer/` (Sphinx/MyST format).
---

# Underworld3 AI Assistant Context

> **Note**: Human-readable developer documentation is in `docs/developer/` (Sphinx/MyST format).
> For development history and completed migrations, see `docs/developer/ai-notes/historical-notes.md`

---

## Extended AI Context (Optional)

You can configure additional AI instruction files that live outside the Underworld repository. This is useful for:
- Personal coding style preferences
- Project planning and coordination systems
- Private instructions not shared with other contributors

**Setup**: If the environment variable `UW_AI_TOOLS_PATH` is set (colon-separated directories, like `PATH`), check each directory for `.md` files containing additional context. Configure this via `./uw setup` or manually in your shell profile.

**At conversation start**: If `UW_AI_TOOLS_PATH` is set, look for `underworld.md` (or other relevant files) in those directories. If found, report relevant Active/Bugs items briefly. If the variable is unset or files aren't found, proceed normally with repo-local context only.

### Responding to External Planning Items

When completing a task from an external planning file, add an annotation directly below the item:

```markdown
<!-- PROJECT RESPONSE (YYYY-MM-DD underworld3):
Brief summary of what was done.
Reference to any files created/modified.
-->
```

### Adding New Items to External Planning

If you discover bugs, identify new tasks, or have items that should be tracked in the external planning file:
- **Don't create local TODO files or add to CLAUDE.md**
- **Do add to the external planning file** under the appropriate section (Bugs, Active, Nice to Have)
- Use the project tag: `<!-- project:underworld3/subsystem -->`

### Inline TODO Comments in Code

Use inline `TODO` comments to mark **problem locations** in the source code. These provide self-documentation and help future developers (human or AI) find the relevant code quickly.

**Format:**
```python
# TODO(BUG): Brief description of the issue
# More context if needed
# See planning file: underworld.md (section, date)
```

**When to use:**
- Mark the exact location of a known bug
- Flag code that needs enhancement or refactoring
- Note incomplete implementations

**Example:**
```python
# TODO(BUG): add_natural_bc() causes PETSc error 73
# The Stokes solver works; issue is specific to scalar Poisson setup.
# See planning file: underworld.md (Bugs section, 2026-01-19)
self.natural_bcs = []
```

This complements the planning file — the planning file tracks *what* needs doing, inline TODOs mark *where* in the code.

### What Not to Do

- Don't rewrite strategic paragraphs — they contain cross-project context
- Don't move items between sections (Active → Done) — planning-claude handles that
- Don't restructure the document

If something needs more than an annotation or simple addition, mention it in conversation for the user to handle.

---

## Pending Release Actions

**⚠️ REMINDER: Tag v3.0.0 when `uw3-release-candidate` merges to `main`**

The AI-friendly codebase with improved documentation, patterns, and tooling should be released as Underworld3 version 3.0.0. After merging the release candidate branch to main:

```bash
git tag -a v3.0.0 -m "Underworld3 Release 3.0.0"
git push origin v3.0.0
```

See `docs/developer/guides/version-management.md` for details.

---

## Documentation Requests

**⚠️ MANDATORY - READ BEFORE WRITING ANY DOCUMENTATION ⚠️**

- **ALL documentation MUST go in `docs/` directory** - NO exceptions
- **NEVER create .md files in the repository root, src/, tests/, or anywhere else**
- **NEVER create planning/design documents outside `docs/developer/design/`**
- If you're tempted to create a file like `SOME-FEATURE-NOTES.md` in the repo root - **DON'T**. Put it in `docs/developer/` instead.
- This applies to: design docs, how-to guides, technical notes, implementation plans, reviews, audits - EVERYTHING goes in `docs/`

**Where to put documentation:**

| Content Type | Location |
|--------------|----------|
| System documentation (meshing, solvers, swarms) | `docs/developer/subsystems/` |
| Architecture and design decisions | `docs/developer/design/` |
| How-to guides and best practices | `docs/developer/guides/` |
| User tutorials | `docs/beginner/tutorials/` |
| Advanced user guides | `docs/advanced/` |

**Format** - Use MyST Markdown (`.md` files) compatible with Sphinx:
- Standard markdown with MyST extensions
- Use ```` ```python ```` for code blocks (not `{python}`)
- Use `{note}`, `{warning}`, `{tip}` for admonitions
- Math: `$inline$` and `$$display$$`

**Style** - Concise, helpful, standalone:
- Self-contained explanations (don't assume reader has context)
- Include practical code examples
- Link to related documentation where appropriate
- Focus on "why" and "how to use", not just "what"
- Follow the notebook style guide for tutorials

**Integration** - Link into the documentation system:
- Add to appropriate toctree in parent `index.md`
- Cross-reference related docs with `:doc:` or relative links
- Build and verify: `pixi run docs-build`

**Style references**:
- Notebook writing: `docs/developer/guides/notebook-style-guide.md`
- Code patterns: `docs/developer/UW3_Style_and_Patterns_Guide.md`

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [underworldcode/underworld3](https://github.com/underworldcode/underworld3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
