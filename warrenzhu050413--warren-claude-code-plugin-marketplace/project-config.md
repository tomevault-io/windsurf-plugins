---
trigger: always_on
description: **VERIFICATION_HASH:** `9f2e4a8c6d1b5730`
---

# Claude Code Plugin Development Guide

**VERIFICATION_HASH:** `9f2e4a8c6d1b5730`

A concise guide to plugin development for Claude Code, focused on structure and best practices.

---

## Quick Reference

### Essential Documentation
- **[Plugin Reference](https://docs.claude.com/en/docs/claude-code/plugins-reference.md)** - Complete plugin structure
- **[Plugin Marketplaces](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces.md)** - Marketplace setup
- **[Plugins Overview](https://docs.claude.com/en/docs/claude-code/plugins.md)** - How plugins work
- **[Hooks Reference](https://docs.claude.com/en/docs/claude-code/hooks.md)** - Hook configuration and events

---

## Plugin Structure

### Minimal Plugin

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # REQUIRED: Plugin manifest
├── commands/                # Optional: Slash commands
│   └── my-command.md
├── snippets/                # Optional: Context snippets
│   └── my-snippet.md
└── README.md                # Recommended
```

### Plugin Manifest (`.claude-plugin/plugin.json`)

```json
{
  "name": "my-plugin",           // REQUIRED: kebab-case
  "version": "1.0.0",            // REQUIRED: semver
  "description": "Brief desc",   // REQUIRED
  "author": {
    "name": "Your Name",
    "email": "you@example.com"
  },
  "commands": "./commands",      // Optional: relative path
  "snippets": "./snippets"       // Optional: relative path
}
```

**Key Rules:**
- All paths MUST be relative: `./commands`, `./snippets`
- Never use absolute paths
- Plugin name MUST be kebab-case
- Version MUST follow semver (X.Y.Z)

---

## Marketplace Structure

### Marketplace Manifest (`.claude-plugin/marketplace.json`)

```json
{
  "name": "marketplace-name",
  "owner": {
    "name": "Owner Name",
    "email": "owner@example.com"
  },
  "plugins": [
    {
      "name": "plugin-name",
      "version": "1.0.0",
      "description": "Brief description",
      "source": "./plugin-directory"
    }
  ]
}
```

**Installation:**
```bash
# Add marketplace
/plugin marketplace add username/repo

# Install plugin
/plugin install plugin-name@marketplace-name
```

---

## Command Files

### Structure

```markdown
---
description: Brief description (shows in /help)
---

# Command Title

## Arguments

Input: `$ARGUMENTS`

## Instructions

1. **Parse arguments**:
   - Check for flags: `-f`, `--force`
   - Extract main input from `$ARGUMENTS`
   - Validate input

2. **Ask for confirmation** (unless `-f` or `--force`):
   - Show what will change
   - Wait for yes/no
   - Proceed only if yes

3. **Execute action**:
   - Perform operation
   - Handle errors gracefully
   - Confirm completion

## Example Usage

```
/my-command example input
/my-command --force another example
```

## Notes

- Important notes here
```

**Key Points:**
- **MUST have YAML frontmatter** with `description`
- **ALWAYS use `$ARGUMENTS`** for user input
- **Ask for confirmation** for create/update/delete (unless forced)
- **No confirmation** for read-only operations

---

## Snippet Files

### Structure

Snippets provide contextual instructions to Claude through the `UserPromptSubmit` hook. They follow a similar structure to commands but serve as continuous context rather than one-time actions.

```markdown
---
description: Brief description of what this snippet provides
SNIPPET_NAME: unique-identifier
ANNOUNCE_USAGE: true
---

# Snippet Title

**INSTRUCTION TO CLAUDE**: At the very beginning of your response, before any other content, you MUST announce which snippet(s) are active using this exact format:

📎 **Active Context**: snippet-name

If multiple snippets are detected, combine them:

📎 **Active Contexts**: snippet1, snippet2, snippet3

---

**VERIFICATION_HASH:** `unique-hash-here`

[Main instructions for Claude...]

## Section 1: Core Instructions

[Detailed instructions...]

## Section 2: Additional Guidance

[More instructions...]
```

**Key Points:**
- **YAML frontmatter** with `description`, `SNIPPET_NAME`, and `ANNOUNCE_USAGE`
- **Announcement block** at the top (tells Claude to announce active contexts)
- **Verification hash** for content integrity tracking
- **Clear section structure** for organization
- **Instructions are directives** to Claude, not conversational

---

## Template Pattern for Complex Snippets

For snippets that require external files (templates, examples), use this pattern:

### Directory Structure

```
my-plugin/
├── snippets/
│   └── my-snippet.md          # Main snippet with instructions
└── templates/
    └── my-template/
        ├── base-template.html  # Base template file
        └── examples.md         # Usage examples
```

### Template-Based Snippet Structure

```markdown
---
description: Snippet description
SNIPPET_NAME: my-snippet
ANNOUNCE_USAGE: true
---

# My Snippet Title

**INSTRUCTION TO CLAUDE**: Announcement block...

---

**VERIFICATION_HASH:** `hash-here`

## Primary Purpose

Main instructions for Claude...

## File Handling Instructions

1. **ALWAYS** create output directory: `mkdir -p output/`
2. Write to: `output/{description}.ext`
3. Open file after writing

## Template System

**Base Template:** `${CLAUDE_PLUGIN_ROOT}/templates/my-template/base-template.html`
- Contains all structure and styling
- Ready to use - just add content

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WarrenZhu050413) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
