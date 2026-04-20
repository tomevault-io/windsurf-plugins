---
trigger: always_on
description: This skill automates the conversion between Claude Code skills and Gemini CLI extensions, enabling true cross-platform AI tool development.
---

# skill-porter - Gemini CLI Extension

Converts Claude Code skills to Gemini CLI extensions and vice versa. Use when the user wants to make a skill cross-platform compatible, port a skill between platforms, or create a universal extension that works on both Claude Code and Gemini CLI.

## Quick Start

After installation, you can use this extension by asking questions or giving commands naturally.


# Skill Porter - Cross-Platform Skill Converter

This skill automates the conversion between Claude Code skills and Gemini CLI extensions, enabling true cross-platform AI tool development.

## Core Capabilities

### Bidirectional Conversion

Convert skills and extensions between platforms while preserving functionality:

**Example requests:**
- "Convert this Claude skill to work with Gemini CLI"
- "Make my Gemini extension compatible with Claude Code"
- "Create a universal version of this skill that works on both platforms"
- "Port the database-helper skill to Gemini"

### Smart Platform Detection

Automatically analyzes directory structure to determine source platform:

**Detection criteria:**
- Claude: Presence of `SKILL.md` with YAML frontmatter or `.claude-plugin/marketplace.json`
- Gemini: Presence of `gemini-extension.json` or `GEMINI.md` context file
- Universal: Has both platform configurations

**Example requests:**
- "What platform is this skill built for?"
- "Analyze this extension and tell me what needs to be converted"
- "Is this a Claude skill or Gemini extension?"

### Metadata Transformation

Intelligently converts between platform-specific formats:

**Conversions handled:**
- YAML frontmatter ↔ JSON manifest
- `allowed-tools` (whitelist) ↔ `excludeTools` (blacklist)
- Environment variables ↔ settings schema
- MCP server configuration paths
- Platform-specific documentation formats

**Example requests:**
- "Convert the metadata from this Claude skill to Gemini format"
- "Transform the allowed-tools list to Gemini's exclude pattern"
- "Generate a settings schema from these environment variables"

### MCP Server Preservation

Maintains Model Context Protocol server configurations across platforms:

**Example requests:**
- "Ensure the MCP server config works on both platforms"
- "Update the MCP server paths for Gemini's ${extensionPath} variable"
- "Validate that the MCP configuration is compatible"

### Validation & Quality Checks

Ensures converted output meets platform requirements:

**Validation checks:**
- Required files present (SKILL.md, gemini-extension.json, etc.)
- Valid YAML/JSON syntax
- Correct frontmatter structure
- MCP server paths resolve correctly
- Tool restrictions are valid
- Settings schema is complete

**Example requests:**
- "Validate this converted skill"
- "Check if this Gemini extension meets all requirements"
- "Is this conversion ready to install?"

## Conversion Process

When you request a conversion, I will:

1. **Analyze** the source directory structure
2. **Detect** which platform it's built for
3. **Extract** metadata, MCP configuration, and documentation
4. **Transform** the data to target platform format
5. **Generate** required files for target platform
6. **Validate** output meets all requirements
7. **Report** what was converted and any manual steps needed

## Platform Differences Handled

### File Structure
- **Claude**: `SKILL.md` + `.claude-plugin/marketplace.json`
- **Gemini**: `GEMINI.md` + `gemini-extension.json`
- **Universal**: Both sets of files + shared documentation

### Metadata Format
- **Claude**: YAML frontmatter in SKILL.md
- **Gemini**: JSON manifest file
- **Conversion**: Bidirectional YAML ↔ JSON transformation

### Tool Restrictions
- **Claude**: `allowed-tools` (whitelist - only listed tools permitted)
- **Gemini**: `excludeTools` (blacklist - listed patterns blocked)
- **Conversion**: Logic inversion with intelligent mapping

### Configuration
- **Claude**: Environment variables (user sets before running)
- **Gemini**: Settings schema (prompted during installation)
- **Conversion**: Infer schema from env vars or generate env var docs

### MCP Server Paths
- **Claude**: Relative paths from skill directory
- **Gemini**: Uses `${extensionPath}` variable substitution
- **Conversion**: Automatic path transformation

## Usage Examples

### Convert Claude Skill to Gemini

```
User: "I have a Claude skill at ./database-helper. Convert it to Gemini."
Claude: I'll analyze the skill and convert it to Gemini format...
- Detected Claude skill (found SKILL.md with frontmatter)
- Extracted metadata and MCP configuration
- Generated gemini-extension.json and GEMINI.md
- Transformed allowed-tools to excludeTools
- Created settings schema from environment variables
✓ Conversion complete! Ready to install with: gemini extensions install ./database-helper
```

### Convert Gemini Extension to Claude

```
User: "Port my Gemini extension to Claude Code"
Claude: I'll convert this Gemini extension to a Claude skill...
- Detected Gemini extension (found gemini-extension.json)
- Extracted manifest and context file
- Generated SKILL.md with YAML frontmatter
- Created .claude-plugin/marketplace.json
- Converted excludeTools to allowed-tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jduncan-rva/skill-porter](https://github.com/jduncan-rva/skill-porter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
