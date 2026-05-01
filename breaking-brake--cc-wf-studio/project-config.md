---
trigger: always_on
description: Auto-generated from all feature plans. Last updated: 2025-11-01
---

# cc-wf-studio Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-11-01

## Language

- GitHub Issues and Pull Requests (titles, bodies, and comments) MUST be written in English.
- This applies regardless of the conversation language used with Claude.

## Active Technologies
- ローカルファイルシステム (`.vscode/workflows/*.json`, `.claude/skills/*.md`, `.claude/commands/*.md`) (001-cc-wf-studio)
- TypeScript 5.3 (VSCode Extension Host), React 18.2 (Webview UI) (001-node-types-extension)
- ローカルファイルシステム (`.vscode/workflows/*.json`) (001-node-types-extension)
- TypeScript 5.3.0 (001-skill-node)
- File system (SKILL.md files in `~/.claude/skills/` and `.claude/skills/`), workflow JSON files in `.vscode/workflows/` (001-skill-node)
- TypeScript 5.3.0 (VSCode Extension Host), TypeScript/React 18.2 (Webview UI) + VSCode Extension API 1.80.0+, React 18.2, React Flow (visual canvas), Zustand (state management), child_process (Claude Code CLI execution) (001-mcp-node)
- Workflow JSON files in `.vscode/workflows/` directory, Claude Code MCP configuration (user/project/enterprise scopes) (001-mcp-node)
- TypeScript 5.3.0 (VSCode Extension Host), TypeScript/React 18.2 (Webview UI) + VSCode Extension API 1.80.0+, React 18.2, React Flow (visual canvas), Zustand (state management), existing MCP SDK client services (001-mcp-natural-language-mode)
- Workflow JSON files in `.vscode/workflows/` directory (extends existing McpNodeData structure) (001-mcp-natural-language-mode)
- TypeScript 5.3 (VSCode Extension Host), React 18.2 (Webview UI), @slack/web-api 7.x, Node.js http (OAuth callback server), VSCode Secret Storage (001-slack-workflow-sharing)
- Workflow JSON files in `.vscode/workflows/` directory, Slack message attachments (workflow storage), VSCode Secret Storage (OAuth tokens) (001-slack-workflow-sharing)

- TypeScript 5.x (VSCode Extension Host), React 18.x (Webview UI) (001-cc-wf-studio)

## Project Structure

```text
src/
tests/
resources/
  workflow-schema.json   # ワークフロースキーマ定義（ソース）
  workflow-schema.toon   # AI向け最適化フォーマット（ビルド時に .json から自動生成。手動編集しない）
```

### Schema Files
- `resources/workflow-schema.json` がスキーマのソースファイル
- `resources/workflow-schema.toon` は `workflow-schema.json` からビルド時に自動生成される。**直接編集しないこと**

## Development Workflow & Commands

### Commit Message Guidelines

**IMPORTANT: Keep commit messages simple for squash merge workflow.**

#### Format
```
<type>: <subject>

<optional body with bullet points>
```

#### Example
```
fix: add missing MCP node definition to workflow schema

- Added 'mcp' to supportedNodeTypes
- Added complete MCP node type definition with field constraints
- Fixes MCP_INVALID_PARAMETERS and MCP_INVALID_MODE validation errors
```

#### Rules
- **Subject**: 50 characters max, imperative mood, no period
- **Body**: 3-5 bullet points max, "what" changed only
- **Details**: Put "why" and "how" in PR description, NOT commit message

#### Types
- `feat:` - New feature (minor version bump)
- `fix:` - Bug fix (patch version bump)
- `improvement:` - Minor enhancement to existing feature (patch version bump)
- `docs:` - Documentation only
- `refactor:` - Code refactoring
- `chore:` - Build/tooling changes

#### What to Avoid
❌ Long explanations (Problem/Solution/Impact sections)
❌ Multiple paragraphs
❌ Code blocks
❌ Test results with checkboxes

✅ Simple 3-5 line summary of changes

### Code Quality Checks (Required Before Commit/PR)

**Always run these commands in the following order after code modifications:**

```bash
npm run check   # Run all Biome checks (lint + format, with auto-fix)
npm run build   # Build extension and webview (verify compilation)
```

### Command Execution Timing

#### During Development
1. **After code modification**:
   ```bash
   npm run check
   ```
   - Runs lint + format with auto-fix in a single command

2. **Before manual E2E testing**:
   ```bash
   npm run build
   ```
   - Compiles TypeScript and builds extension
   - Required for testing changes in VSCode

3. **Before git commit**:
   ```bash
   npm run check
   ```
   - Ensures all code quality standards are met
   - Prevents committing code with linting/formatting issues

#### Testing
- **Unit/Integration tests**: Not required (manual E2E testing only)
- **Manual E2E testing**: Required for all feature changes and bug fixes
  - Run `npm run build` first
  - Test in VSCode Extension Development Host

## Version Update Procedure

**IMPORTANT: Version updates and releases are fully automated via Semantic Release and GitHub Actions.**

### Automated Release Process

This project uses **Semantic Release** with **GitHub Actions** for fully automated versioning, changelog generation, and publishing.

#### Release Workflow (`.github/workflows/release.yml`)

**Trigger**: Push to `production` branch

**Automated Steps**:
1. **Semantic Release** analyzes commit messages and determines version bump
2. **Version Update**: Updates `package.json`, `src/webview/package.json`, `src/webview/package-lock.json`
3. **Changelog Generation**: Automatically updates `CHANGELOG.md`
4. **Git Commit**: Creates release commit with message `chore(release): ${version} [skip ci]`
5. **GitHub Release**: Creates GitHub release with release notes
6. **VSIX Build**: Builds and packages the extension

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [breaking-brake/cc-wf-studio](https://github.com/breaking-brake/cc-wf-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
