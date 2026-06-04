---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code Boost is a TypeScript-based CLI tool that provides intelligent auto-approval hooks for Claude Code. It enhances developer productivity by automatically approving safe development operations while blocking genuinely destructive commands.

## Architecture

The project follows a simple CLI architecture:

- **Entry Point**: `src/index.ts` - Uses Commander.js for CLI structure
- **Main Command**: `src/commands/auto-approve-tools.ts` - Core logic for tool approval decisions
- **Hook Integration**: Processes Claude Code PreToolUse hooks via stdin/stdout JSON communication  
- **AI-Powered Decisions**: Falls back to Claude API for complex approval decisions
- **Type Safety**: `src/types/hook-schemas.ts` - Zod schemas for input/output validation
- **Security Model**: Two-tier approval system:
  - Fast approval for unambiguously safe tools (Read, LS, Glob, etc.)
  - AI-powered analysis for complex operations using `prompts/system-prompt.md`

## Development Commands

### Build and Test
```bash
npm run build          # TypeScript compilation to dist/
npm run type-check     # Type checking without compilation
npm run test           # Run test suite with Vitest
npm run test:watch     # Watch mode for tests
npm run test:env       # Run tests with .env.local file
```

### Linting and Formatting
```bash
npm run lint           # ESLint checking
npm run lint:fix       # Auto-fix linting issues
npm run prettier       # Format code with Prettier
npm run prettier:check # Check code formatting
```

### Development
```bash
npm run dev            # Run CLI with tsx (development mode)
npm run prepublishOnly # Full build + test + lint pipeline
```

### CLI Testing
```bash
# Test the CLI locally (requires ANTHROPIC_API_KEY env var or config.json)
echo '{"session_id":"test","transcript_path":"/tmp/test","tool_name":"Read","tool_input":{"file_path":"/test"}}' | ANTHROPIC_API_KEY=your_key npm run dev auto-approve-tools

# Test with API authentication
echo '{"session_id":"test","transcript_path":"/tmp/test","tool_name":"Read","tool_input":{"file_path":"/test"}}' | npm run dev auto-approve-tools

# Install CCB hook to Claude Code settings
npm run build && node dist/index.js install --user        # Install to user settings
npm run build && node dist/index.js install --project     # Install to project settings
npm run build && node dist/index.js install --project-local # Install to project local settings
```

### Release Process

The project includes automated release scripts for consistent versioning and changelog management:

```bash
# Generate changelog entry for current version
npm run changelog

# Full release process (interactive)
npm run release patch    # Patch version (0.3.0 -> 0.3.1)
npm run release minor    # Minor version (0.3.0 -> 0.4.0) 
npm run release major    # Major version (0.3.0 -> 1.0.0)
npm run release 1.0.0    # Specific version

# Manual changelog generation
tsx scripts/generate-changelog-entry.ts [version]
```

The release script will:
1. Run tests, linting, and build
2. Update package.json version
3. Generate changelog entry from git commits
4. Create git commit and tag
5. Optionally push to remote and publish to npm

## Key Implementation Details

### Hook Processing Flow
1. Reads JSON input from stdin (Claude Code hook format)
2. Parses with Zod schema validation
3. Checks fast approval list first (read-only operations)
4. Falls back to AI analysis via:
   - beyondthehype.dev API proxy (recommended)
   - OpenAI-compatible endpoints (OpenAI, OpenRouter, etc.)
   - Anthropic API (via OpenAI SDK compatibility layer)
5. Returns JSON decision: `{"decision": "approve|block|undefined", "reason": "..."}`

### LLM Client Architecture
- **Unified OpenAI SDK**: Uses OpenAI SDK for all LLM providers including Anthropic
- **Multi-provider support**: Handles different authentication methods seamlessly
- **Anthropic compatibility**: Uses Anthropic's OpenAI-compatible endpoint at `https://api.anthropic.com/v1`
- **Automatic model selection**: Chooses appropriate model based on authentication method
- **Structured output**: Handles JSON schema requirements across different providers

### Security Philosophy
- **Permissive by default** - Approves standard development operations
- **Context-aware** - Uses project knowledge to inform decisions
- **Destructive-only blocking** - Only blocks genuinely harmful operations (rm -rf /, system wipes)
- **Developer trust** - Assumes most operations are legitimate development work

### Testing Strategy
- Unit tests for approval logic
- Integration tests with actual Claude API calls
- Mock scenarios for edge cases
- Environment variable support for API testing

## Configuration

### Environment Variables

- `CCB_CONFIG_DIR` - Configuration directory for CCB (defaults to `$HOME/.ccb`)
- `ANTHROPIC_API_KEY` - API key for Anthropic Claude API integration (can also be set in config.json)

### Configuration

CCB uses a `config.json` file located in the CCB configuration directory (`$CCB_CONFIG_DIR` or `$HOME/.ccb`). The config schema includes:

```json
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yifanzz/claude-code-boost](https://github.com/yifanzz/claude-code-boost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
