---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 📁 File Organization Guidelines

**⚠️ IMPORTANT - Temporary Documentation Files**:
- **All temporary markdown files** created without explicit user request **MUST** be saved to `/plans/temp/`
- Examples: exploration notes, research summaries, draft documentation, intermediate analysis
- Permanent documentation (user-requested plans, design docs) goes in `/plans/`
- The `/plans/temp/` directory is git-ignored and not tracked in version control

## 🤖 Task Agent Usage Guidelines

**IMPORTANT**: Use specialized Task agents liberally for exploration, planning, and research tasks.

### When to Use Task Agents

**Explore Agent** (use `subagent_type=Explore`):
- Understanding codebase structure and architecture
- Finding where features are implemented across multiple files
- Exploring error handling patterns, API endpoints, or design patterns
- Questions like "How does X work?", "Where is Y handled?", "What's the structure of Z?"
- Set thoroughness: `quick` (basic), `medium` (moderate), or `very thorough` (comprehensive)

**Plan Agent** (use `subagent_type=Plan`):
- Breaking down complex feature implementations
- Designing multi-step refactoring approaches
- Planning architectural changes or migrations

**General-Purpose Agent** (use `subagent_type=general-purpose`):
- Multi-step tasks requiring multiple tool invocations
- Documentation lookups via WebSearch/WebFetch
- Complex searches across many files with multiple rounds

### Documentation Lookup Pattern

When encountering unfamiliar libraries, frameworks, or patterns:

1. **Use Task agent with WebSearch/WebFetch** to look up official documentation
2. **Search for**: Rails 8 features, Playwright APIs, Python FastAPI patterns, etc.
3. **Example**: "Look up Rails 8 Turbo Stream documentation to understand real-time updates"

### Examples

```typescript
// ❌ Don't: Manually grep/glob through unfamiliar codebase
grep -r "webhook" .

// ✅ Do: Use Explore agent
Task(subagent_type="Explore",
     prompt="Find all webhook implementations and explain how webhooks work in this Rails+Python microservices architecture",
     thoroughness="medium")

// ❌ Don't: Guess how to implement without research
// Implement feature based on assumptions

// ✅ Do: Use general-purpose agent for documentation lookup
Task(subagent_type="general-purpose",
     prompt="Research Rails 8 ActionCable best practices for WebSocket broadcasting, then summarize the recommended pattern for real-time updates")
```

## Project Overview

Meme Search is a self-hosted AI-powered meme search engine with a microservices architecture:
- **Rails Application** (`meme_search/meme_search_app`) - Main web app on port 3000
- **Python Image-to-Text Service** (`meme_search/image_to_text_generator`) - AI inference service on port 8000
- **PostgreSQL with pgvector** - Database with vector similarity search

## Environment Setup

**Tool Versions** (managed via [mise](https://mise.jdx.dev/)):
- Ruby: 3.4.2 | Python: 3.12 | Node.js: 20 | PostgreSQL: 17 (Docker)

**Quick Setup**:
```bash
brew install mise
eval "$(mise activate zsh)"  # Add to shell config
cd /path/to/meme-search && mise trust && mise install
mise doctor  # Verify setup
```

**Run Commands**: `mise exec -- <command>` or let mise auto-activate when you `cd` into the project

## Development Commands

### Quick Reference

**Unified CI Tests** (from project root):
```bash
npm test                     # All tests (Rails + Python + E2E) ~5-10 min
npm run test:ci:skip-e2e     # Skip E2E for faster feedback ~3-5 min
npm run test:rails           # Rails only
npm run test:python          # Python only
```

**Rails** (`meme_search/meme_search_app`):
```bash
./bin/dev                    # Start dev server
bash run_tests.sh            # All Rails tests
bin/rails test test/models   # Model tests
COVERAGE=true bin/rails test # With coverage
rubocop app && brakeman      # Lint + security scan
```

**Python** (`meme_search/image_to_text_generator`):
```bash
bash run_tests.sh            # All Python tests (lint + integration + unit)
pytest tests/unit/           # Unit tests (88 tests, 81.52% coverage)
ruff check app/              # Lint
```

**Docker**:
```bash
docker compose up                                      # Production (pre-built images)
docker compose -f docker-compose-local-build.yml up   # Local build
```

### Docker E2E Tests

**Purpose**: Validate full microservices stack (Rails + Python + PostgreSQL) in production-like containers. For **local validation before major releases**, not CI/CD (10-15 min builds). See `playwright-docker/README.md` for details.

```bash
npm run test:e2e:docker              # Full test (setup + run + teardown)
npm run test:e2e:docker:setup        # Build + start services
npm run test:e2e:docker:ui           # Interactive UI mode
```

**Docker E2E** (6/7 passing): Tests Docker builds, isolated ports (3001, 5433, 8000)
**CI E2E** (16/16 passing): Tests local services, fast, runs in GitHub Actions

### Playwright E2E Tests (CI)

**16/16 tests passing** (100% migrated from Capybara). Uses Page Object Model pattern. **See `playwright/README.md` for comprehensive docs**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neonwatty/meme-search](https://github.com/neonwatty/meme-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
