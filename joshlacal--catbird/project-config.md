---
trigger: always_on
description: This guide provides prescriptive, production-focused instructions for AI agents (GitHub Copilot, Aider, Cursor, etc.) working in this repository. **All workflows leverage MCP (Model Context Protocol) servers for intelligent automation.**
---

# AGENTS.md

This guide provides prescriptive, production-focused instructions for AI agents (GitHub Copilot, Aider, Cursor, etc.) working in this repository. **All workflows leverage MCP (Model Context Protocol) servers for intelligent automation.**

## Critical Development Principles

### Efficiency & Workflow
- **NO timeline estimates**: Don't predict how long things will take - timelines are consistently inaccurate
- **NO dates in documentation**: Avoid date-based references that become stale immediately
- **BUILD FREELY**: Builds take ~20 seconds on M4 Max - just do it
- **Verify with real builds**: Use XcodeBuildMCP for actual compilation, not just syntax checks
- **Full verification loop**: Build → Run → describe_ui → Screenshot → Test
- **Use MCP servers extensively**: Always verify against Apple docs via MCP, use xcodebuild-mcp, leverage all available MCP tools
- **Work continuously**: No artificial session boundaries - complete tasks fully without stopping prematurely
- **Maximize parallelism**: Use parallel tool calls aggressively for all independent operations
- **Parallel agents**: Consider using `parallel-agents.py` for truly independent concurrent tasks

### Documentation Management
- **Session notes directory**: Place temporary fix documentation in `docs/session-notes/` (gitignored, auto-cleaned)
- **Keep root clean**: Never clutter main directory with ad-hoc fix documentation files
- **Permanent docs only**: Only commit documentation with lasting value to repository root
- **Clear old fixes**: Session notes should be reviewed and either promoted to permanent docs or deleted

### Context Management
- **No context anxiety**: Don't underestimate capacity - you can accomplish more in a session than you think
- **Rolling context**: Application maintains context automatically across work - don't artificially limit scope
- **Work depth over breadth**: Complete tasks thoroughly rather than stopping prematurely due to perceived limits
- **Trust your capabilities**: The system handles context management - focus on completing work

### Communication Style
- **Minimal explanations**: Execute efficiently without verbose preambles or postambles
- **Skip confirmations**: Don't repeatedly ask for permission on straightforward, low-risk tasks
- **Direct action**: Prefer immediate execution over prolonged discussion for simple operations
- **Reduce safety theater**: Balance caution with productivity - not everything needs multiple confirmations

## Parallel Agents System

For truly independent tasks that can run concurrently, use the parallel agents system to spawn multiple Copilot CLI instances:

```bash
# Example: Run multiple independent tasks simultaneously
./parallel-agents.py quick \
  "Check Swift syntax in Core/" \
  "Build iOS target" \
  "Run tests" \
  --approval='--allow-all-tools'
```

**Use parallel agents for:**
- Multi-platform builds (iOS + macOS + tests simultaneously)
- Independent code quality checks (syntax + lint + security scans)
- Parallel feature development across different modules
- Concurrent documentation generation
- Large-scale refactoring across independent files

See `PARALLEL_AGENTS_README.md` for complete documentation.

## Project Overview

Catbird is a **PRODUCTION-READY** cross-platform client for Bluesky built with SwiftUI and modern Swift 6 patterns, supporting both iOS and macOS. This is a release-ready application where all code must be production-quality with no placeholders, fallbacks, or temporary implementations. It uses the Petrel library for AT Protocol communication.

### Project Components
- **Catbird**: Cross-platform app with SwiftUI interface for Bluesky (iOS and macOS)
- **Petrel**: Swift library providing AT Protocol networking and data models (auto-generated from Lexicon JSON files)
- **CatbirdNotificationWidget**: iOS widget extension for notifications
- **CatbirdFeedWidget**: Feed widget extension (iOS only, in development)

### Platform Support
- **iOS 26.0+**: Full featured mobile client with UIKit optimizations and Liquid Glass design (minimum iOS 18.0+ for legacy support)
- **macOS Tahoe 26.0+**: Native macOS client with SwiftUI-based feed implementation and Liquid Glass (minimum macOS 13.0+ for legacy support)
- **Shared Codebase**: ~95% code sharing between platforms using conditional compilation

### iOS 26 Naming Convention
Apple introduced a major change to OS naming at WWDC 2025, moving from sequential numbering to year-based naming:
- **iOS 26** represents the 2025-2026 release cycle (September 2025 - September 2026)
- What would have been iOS 19 is now iOS 26, reflecting the year it will be current
- This naming convention applies to all Apple platforms: iOS 26, iPadOS 26, macOS Tahoe 26, watchOS 26, tvOS 26, visionOS 26
- Provides consistency across all Apple operating systems and aligns with industry conventions (like automotive model years)

### Project Structure
```
/Catbird/
├── App/                    # App entry point
├── Core/                   # Core infrastructure
│   ├── Extensions/         # Swift extensions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshlacal/Catbird](https://github.com/joshlacal/Catbird) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
