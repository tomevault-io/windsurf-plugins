---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

Node.js CLI tool for managing Claude Code components (agents, commands, MCPs, hooks, settings) with a static website for browsing and installing components. The project includes Vercel API endpoints for download tracking and Discord integration.

## Essential Commands

```bash
# Development
npm install                    # Install dependencies
npm test                       # Run tests
npm version patch|minor|major  # Bump version
npm publish                    # Publish to npm

# Component catalog
python scripts/generate_components_json.py  # Update docs/components.json

# API testing
cd api && npm test             # Test API endpoints before deploy
vercel --prod                  # Deploy to production
```

## Security Guidelines

### ⛔ CRITICAL: NEVER Hardcode Secrets or IDs

**NEVER write API keys, tokens, passwords, project IDs, org IDs, or any identifier in code.** This includes Vercel project/org IDs, Supabase URLs, Discord IDs, database connection strings, and any other infrastructure identifier. ALL must go in `.env`.

```javascript
// ❌ WRONG
const API_KEY = "AIzaSy...";

// ✅ CORRECT
const API_KEY = process.env.GOOGLE_API_KEY;
```

**When creating scripts with API keys:**
1. Use `process.env` (Node.js) or `os.environ.get()` (Python)
2. Load from `.env` file using `dotenv`
3. Add variable to `.env.example` with placeholder
4. Verify `.env` is in `.gitignore`

**If you accidentally commit a secret:**
1. Revoke the key IMMEDIATELY
2. Generate new key
3. Update `.env`
4. Old key is compromised forever (git history)

## Component System

### Component Types

**Agents** (600+) - AI specialists for development tasks
**Commands** (200+) - Custom slash commands for workflows
**MCPs** (55+) - External service integrations
**Settings** (60+) - Claude Code configuration files
**Hooks** (39+) - Automation triggers
**Templates** (14+) - Complete project configurations

### Installation Patterns

```bash
# Single component
npx claude-code-templates@latest --agent frontend-developer
npx claude-code-templates@latest --command setup-testing
npx claude-code-templates@latest --hook automation/simple-notifications

# Batch installation
npx claude-code-templates@latest --agent security-auditor --command security-audit --setting read-only-mode

# Interactive mode
npx claude-code-templates@latest
```

### Component Development

#### Adding New Components

**CRITICAL: Use the component-reviewer agent for ALL component changes**

When adding or modifying components, you MUST use the `component-reviewer` subagent to validate the component before committing:

```
Use the component-reviewer agent to review [component-path]
```

**Component Creation Workflow:**

1. Create component file in `cli-tool/components/{type}/{category}/{name}.md`
2. Use descriptive hyphenated names (kebab-case)
3. Include clear descriptions and usage examples
4. **REVIEW with component-reviewer agent** (validates format, security, naming)
5. Fix any issues identified by the reviewer
6. Run `python scripts/generate_components_json.py` to update catalog

**The component-reviewer agent checks:**
- ✅ Valid YAML frontmatter and required fields
- ✅ Proper kebab-case naming conventions
- ✅ No hardcoded secrets (API keys, tokens, passwords)
- ✅ Relative paths only (no absolute paths)
- ✅ Supporting files exist (for hooks with scripts)
- ✅ Clear, specific descriptions
- ✅ Correct category placement
- ✅ Security best practices

**Example Usage:**
```
# After creating a new agent
Use the component-reviewer agent to review cli-tool/components/agents/development-team/react-expert.md

# Before committing hook changes
Use the component-reviewer agent to review cli-tool/components/hooks/git/prevent-force-push.json

# For PR reviews with multiple components
Use the component-reviewer agent to review all modified components in cli-tool/components/
```

The agent will provide prioritized feedback:
- **❌ Critical Issues**: Must fix before merge (security, missing fields)
- **⚠️ Warnings**: Should fix (clarity, best practices)
- **📋 Suggestions**: Nice to have improvements

#### Statuslines with Python Scripts

Statuslines can reference Python scripts that are auto-downloaded to `.claude/scripts/`:

```javascript
// In src/index.js:installIndividualSetting()
if (settingName.includes('statusline/')) {
  const pythonFileName = settingName.split('/')[1] + '.py';
  const pythonUrl = githubUrl.replace('.json', '.py');
  additionalFiles['.claude/scripts/' + pythonFileName] = {
    content: pythonContent,
    executable: true
  };
}
```

### Publishing Workflow

```bash
# 1. Update component catalog
python scripts/generate_components_json.py

# 2. Run tests
npm test

# 3. Check current npm version and align local version
npm view claude-code-templates version  # check latest on registry
# Edit package.json version to be one patch above the registry version

# 4. Commit version bump and push
git add package.json && git commit -m "chore: Bump version to X.Y.Z"
git push origin main

# 5. Publish to npm (requires granular access token with "Bypass 2FA" enabled)
npm config set //registry.npmjs.org/:_authToken=YOUR_GRANULAR_TOKEN
npm publish

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davila7/claude-code-templates](https://github.com/davila7/claude-code-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
