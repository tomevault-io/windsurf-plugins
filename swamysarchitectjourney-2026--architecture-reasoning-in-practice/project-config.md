---
trigger: always_on
description: enables:
---

# Markdown Authoring Standards

**Version**: 1.0  
**Last Updated**: January 1, 2026

---

## 🎨 Markdown Authoring Standards

### Character Encoding (CRITICAL)

- ✅ Use UTF-8 encoding for all files
- ❌ NEVER use (replacement character) - breaks previews
- ✅ If preview broken, search for replacement character first
- ✅ Validate UTF-8 integrity after content changes

### Core Markdown Rules

- Spaces only (no hard tabs) - MD010
- 2-space indentation for nested lists - MD007
- Language specified for code fences - MD040
- Blank lines around headings/lists/fences - MD022/MD031/MD032
- Wrap filenames/paths/code in backticks
- Line length ~120 chars (tables/URLs may exceed)

### Code Fences

Supported languages: `text`, `mermaid`, `bash`, `powershell`, `json`, `yaml`, `python`, `typescript`, `javascript`, `csharp`

### Diagrams

- Mermaid-first (primary visualization method)
- Include ASCII fallback for compatibility and plain-text viewing
- Consistent styling across repository
- NEVER embed copyrighted figures

---

## 🔍 YAML Frontmatter Validation (CRITICAL)

### Required Structure

All educational content files MUST have valid YAML frontmatter with proper structure:

```yaml
---
learning_level: "Beginner" | "Intermediate" | "Advanced"
prerequisites: ["required knowledge", "prior concepts"]
estimated_time: "25 minutes"
learning_objectives:
  - "Specific, measurable outcome 1"
  - "Specific, measurable outcome 2"
related_topics:
  prerequisites: []
  builds_upon: []
  enables:
    - 01_Next-File.md
    - 02_Another-File.md
  cross_refs: []
---
```

### Common Pitfalls & Prevention

#### ❌ **CRITICAL ERROR: Template Placeholder Corruption**

**Problem Pattern:**

```yaml
# WRONG - Missing 'enables:' key and placeholder not replaced
  $101_Next-File.md
    - 02_Another-File.md
```

**Root Cause:**

- Template placeholders like `$101_`, `$102_`, `$103_` were used but never replaced
- Missing `enables:` key in `related_topics` section
- Placeholder pattern `$10X_` indicates template variable that wasn't substituted

**Correct Format:**

```yaml
# CORRECT - Proper YAML structure
related_topics:
  enables:
    - 01_Next-File.md
    - 02_Another-File.md
```

#### Prevention Checklist

Before creating or modifying YAML frontmatter:

1. ✅ **Always include `enables:` key** in `related_topics` section
2. ✅ **Never use placeholder patterns** like `$101_`, `$102_`, `$103_` in final files
3. ✅ **Replace all template variables** before saving files
4. ✅ **Validate YAML syntax** - ensure proper indentation and list format
5. ✅ **Use proper list format** - all items under `enables:` must start with `-`
6. ✅ **File prefixes must match** - `01_`, `02_`, `03_` not `$101_`, `$102_`, `$103_`

#### Validation Rules

**Required Keys:**

- `learning_level` - Must be present
- `prerequisites` - Must be present (can be empty array)
- `estimated_time` - Must be present
- `learning_objectives` - Must be present (array with at least one item)
- `related_topics` - Must be present with sub-keys:
  - `prerequisites` - Array (can be empty)
  - `builds_upon` - Array (can be empty)
  - `enables` - Array (can be empty, but key must exist)
  - `cross_refs` - Array (can be empty)

**Forbidden Patterns:**

- ❌ `$10X_` - Template placeholder pattern (must be replaced)
- ❌ Missing `enables:` key in `related_topics`
- ❌ Top-level list items without parent key
- ❌ Inconsistent indentation (must use 2 spaces)

#### Automated Validation

Run validation scripts before committing:

- `.\tools\psscripts\Review-EducationalContent.ps1` - Checks metadata compliance
- `.\tools\psscripts\Comprehensive-WorkspaceReview.ps1` - Full workspace validation
- `.\tools\psscripts\Validate-FileReferences.ps1` - Validates all file references

#### Example of Correct Structure

```yaml
---
learning_level: "Beginner"
prerequisites: ["Basic programming knowledge"]
estimated_time: "25 minutes"
learning_objectives:
  - "Understand core concepts"
  - "Apply principles in practice"
related_topics:
  prerequisites: []
  builds_upon: []
  enables:
    - 01_Next-Topic-Part1.md
    - 01_Next-Topic-Part2.md
    - 02_Advanced-Topic.md
  cross_refs:
    - ../../02_AI-and-ML/
---
```

---

## 🔗 File Reference Validation (CRITICAL)

### Required Practices

**CRITICAL**: All file references in YAML frontmatter and markdown content MUST point to existing files.

### Common Broken Reference Patterns

#### ❌ **ERROR: Missing Part Suffixes**

**Problem Pattern:**

```yaml
# WRONG - File doesn't exist
prerequisites:
  - 01_OOP-Introduction-Part1.md
```

**Root Cause:**

- Files were split into multiple parts (e.g., `-Part1-A.md`, `-Part1-B.md`)
- References were not updated to match actual file names
- Splitting process created new files but didn't update all references

**Correct Format:**

```yaml
# CORRECT - References actual file (using simplified naming)
prerequisites:
  - 01_OOP-Introduction-Part1-A.md
```

#### ❌ **ERROR: Non-Existent Part Files**

**Problem Pattern:**

```yaml
# WRONG - Part2.md doesn't exist, only Part2-A.md exists
enables:
  - 03_Encapsulation-Part2.md
```

**Root Cause:**

- Files were split multiple times creating nested part structures
- References use simplified names that don't match actual file structure
- Need to check actual file names before referencing

**Correct Format:**

```yaml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SwamysArchitectJourney-2026/architecture-reasoning-in-practice](https://github.com/SwamysArchitectJourney-2026/architecture-reasoning-in-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
