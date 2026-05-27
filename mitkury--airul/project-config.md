---
trigger: always_on
description: Generates AI context files:
---

This is a context for AI editor/agent about the project. It's generated with a tool Airul (https://github.com/mitkury/airul) out of 3 sources. Feel free to edit .airul.json to change the sources and configure editors. Run `airul gen` to update the context after making changes to .airul.json or the sources. Remember to update TODO-AI.md after major changes in the project, keeping track of completed tasks and new developments.

# From TODO-AI.md:

# AI Workspace

## Active Task
✅ **COMPLETED**: Write tests for document ordering fix

## Status
✅ Completed

## Context & Progress
- Created: 2025-02-13
- I (AI) will maintain this document as we work together
- My current focus: Understanding and working on the active task

## Task History
- ✅ **COMPLETED**: Add ability to generate AGENTS.md for Codex @https://platform.openai.com/docs/codex/overview#using_agents_md
  - Added `codex` field to all relevant TypeScript interfaces
  - Updated generator to create AGENTS.md when codex is enabled
  - Added `--codex` flag to CLI commands (init, generate, new)
  - Updated README documentation to include Codex in supported editors table
  - Added comprehensive test coverage for Codex functionality
  - All tests passing (46/46)
- ✅ **COMPLETED**: Fix document ordering in generated files
  - Fixed issue where files were appearing in alphabetical order instead of source array order
  - Updated `expandAndDeduplicate` function to properly handle explicit files vs glob patterns
  - Explicit files maintain their order in the sources array
  - Glob patterns (like `docs/*.md`) get alphabetical ordering within their group
  - Fixed bug where `0` was being treated as falsy in the sorting logic
  - All tests passing (46/46)
- ✅ **COMPLETED**: Write tests for document ordering fix
  - Added test case to verify explicit files maintain their order in sources array
  - Added test case to verify glob patterns are sorted alphabetically within their group
  - Both new tests are passing and verify the ordering functionality works correctly
  - All generator tests passing (16/16)

## Notes
- I'll update this file to track our progress and maintain context
- I'll keep sections concise but informative
- I'll update status and add key decisions/changes
- I'll add new tasks as they come up

## Next Steps
- Consider adding support for other AI editors as needed
- Monitor usage and feedback for the Codex integration
- Consider adding more customization options for AGENTS.md format if needed
---

# From README.md:

# Airul

Airul is for generating context for AI agents (AGENTS.md, CLAUDE.md, etc) from your docs. So you can link several text files, including description of your project, user docs and other things that seems useful for agents and then generate one global context file referenced by your AI agent.

## How to use

### Starting a new project

```bash
# Install as a CLI tool
npm install -g airul

# Create a new project and open in Cursor
airul new my-project "Create a React app with authentication" --cursor

# This will:
# 1. Create my-project directory
# 2. Initialize git repository
# 3. Create initial documentation
# 4. Generate AI context files
# 5. Open in Cursor (and other editors if specified)
```

### Core Commands

#### `airul init`
Initializes a new project:
- Creates `.airul.json` config if it doesn't exist
- Creates initial `TODO-AI.md` if it doesn't exist
- Generates rules from the new configuration
- Initializes git repository (if not already initialized)

#### `airul gen`
Generates AI context files:
- If `.airul.json` exists: generates rules using the existing configuration
- If `.airul.json` doesn't exist: runs `init` first, then generates rules
- Always uses the most recent content from your source files

### Adding to existing project

```bash
# Install as a CLI tool
npm install -g airul

# Initialize airul in your project
airul init 

# This will:
# 1. Add airul as dev dependency
# 2. Create .airul.json config
# 3. Create initial documentation
# 4. Generate AI context files
```

### Keeping context updated

After making changes to your project, you have these options to update the AI context:

```bash
# Run this after making changes to your documentation
airul gen
```

You can run this command directly if you installed Airul globally, or use `npx airul gen` if installed as a dev dependency.

For automatic updates, add this to your package.json:
```json
{
  "devDependencies": {
    "airul": "latest"
  },
  "scripts": {
    "prestart": "airul gen",
    "prebuild": "airul gen"
  }
}
```

This way, your AI context will always be updated before running or building your project.

All approaches will update context when you:
- Add/modify documentation
- Install new dependencies
- Change project structure

## Supported Editors

Airul supports multiple AI-powered editors. You can enable them during initialization or in existing projects:

| Editor | Output File | Flag to Enable | Configuration |
|--------|-------------|----------------|--------------|
| Cursor | `AGENTS.md` | `--cursor` | `"cursor": true` |
| GitHub Copilot | `.github/copilot-instructions.md` | `--copilot` | `"copilot": true` |
| Windsurf | `.windsurfrules` | `--windsurf` | `"windsurf": true` |
| Claude | `CLAUDE.md` | `--claude` | `"claude": true` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitkury/airul](https://github.com/mitkury/airul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
