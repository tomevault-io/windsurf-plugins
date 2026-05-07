---
trigger: always_on
description: This file provides guidance to AI coding assistants working in this repository.
---

# AGENTS.md
This file provides guidance to AI coding assistants working in this repository.

**Note:** CLAUDE.md, .clinerules, .cursorrules, .windsurfrules, .replit.md, .github/copilot-instructions.md, and other AI config files are symlinks to AGENTS.md in this project.

# cclint (Claude Code Lint)

**cclint** is a comprehensive linting tool specifically for **Claude Code** projects that validates agent definitions, command configurations, settings files, and project documentation against Claude Code's official specifications. 

**IMPORTANT**: All hardcoded validations (colors, tools, hook events, etc.) are strictly based on what Claude Code actually supports. Any project-specific extensions or custom values should be handled through the custom schema system, not by making the base validations more lenient.

### Key Features
- Agent/Subagent frontmatter and tool validation
- Command definition and permission checking  
- Settings.json hook configuration validation
- CLAUDE.md documentation structure validation
- Custom schema support with security sandboxing
- Auto-detection of project root via directory climbing
- Multiple output formats (console, JSON, markdown)
- CI/CD integration with configurable exit codes

## Build & Commands

### Script Command Consistency
**Important**: When modifying npm scripts in package.json, ensure all references are updated:
- GitHub Actions workflows (.github/workflows/*.yml)
- README.md documentation
- Contributing guides
- Dockerfile/docker-compose.yml
- CI/CD configuration files
- Setup/installation scripts

Common places that reference npm scripts:
- Build commands → Check: workflows, README, Dockerfile
- Test commands → Check: workflows, contributing docs
- Lint commands → Check: pre-commit hooks, workflows
- Start commands → Check: README, deployment docs

**Note**: Always use the EXACT script names from package.json, not assumed names

## Build & Development Commands

### Essential Commands
```bash
# Build the project
npm run build

# Run tests (comprehensive test suite - 153+ tests)  
npm test

# Development watch mode
npm run dev

# Lint TypeScript code
npm run lint

# Format code with Prettier
npm run format

# Install CLI globally for testing
npm install -g .
```

### Testing Strategy
- **Framework**: Vitest 3.2.4 with comprehensive test coverage
- **Test Files**: `*.test.ts` in `src/` directory structure
- **Coverage**: 153+ tests covering all linter components
- **Security Testing**: Includes path traversal, code injection, and timeout protection tests
- **Integration Tests**: End-to-end CLI testing with real configuration files

### Testing Philosophy
**When tests fail, fix the code, not the test.**

Key principles:
- **Tests should be meaningful** - Avoid tests that always pass regardless of behavior
- **Test actual functionality** - Call the functions being tested, don't just check side effects
- **Failing tests are valuable** - They reveal bugs or missing features
- **Fix the root cause** - When a test fails, fix the underlying issue, don't hide the test
- **Test edge cases** - Tests that reveal limitations help improve the code
- **Document test purpose** - Each test should include a comment explaining why it exists and what it validates

### Quality Assurance
- **TypeScript**: Strict mode enabled, no `any` types allowed
- **ESLint**: @typescript-eslint rules for code quality
- **Prettier**: Consistent code formatting
- **Vitest**: Comprehensive unit and integration testing
- **Security**: Path sanitization and code execution protection

## Architecture & Code Style

### Project Structure
```
src/
├── cli.ts              # Main CLI entry point with Commander.js
├── lib/
│   ├── config.ts       # Secure configuration loading with sandboxing
│   ├── utils.ts        # Utilities with path sanitization
│   └── file-scanner.ts # Parallel file discovery with glob patterns
├── linters/
│   ├── base.ts         # Base linter class with shared functionality
│   ├── agents.ts       # Agent/subagent frontmatter validation
│   ├── commands.ts     # Command definition validation
│   ├── settings.ts     # Settings.json hook validation
│   └── claude-md.ts    # Documentation structure validation
├── schemas/
│   └── index.ts        # Zod schemas for all file types
└── types/
    └── index.ts        # TypeScript type definitions
```

## Directory Structure & File Organization

### Reports Directory
ALL project reports and documentation should be saved to the `reports/` directory:

```
cclint/
├── reports/              # All project reports and documentation
│   └── *.md             # Various report types
├── temp/                # Temporary files and debugging
└── [other directories]
```

### Report Generation Guidelines
**Important**: ALL reports should be saved to the `reports/` directory with descriptive names:

**Implementation Reports:**
- Phase validation: `PHASE_X_VALIDATION_REPORT.md`
- Implementation summaries: `IMPLEMENTATION_SUMMARY_[FEATURE].md`
- Feature completion: `FEATURE_[NAME]_REPORT.md`

**Testing & Analysis Reports:**
- Test results: `TEST_RESULTS_[DATE].md`
- Coverage reports: `COVERAGE_REPORT_[DATE].md`
- Performance analysis: `PERFORMANCE_ANALYSIS_[SCENARIO].md`
- Security scans: `SECURITY_SCAN_[DATE].md`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlrannaberg/cclint](https://github.com/carlrannaberg/cclint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
