---
trigger: always_on
description: This is the **GitHub Copilot for Eclipse** plugin - an Eclipse IDE extension that brings AI-powered code completions, chat assistance, and intelligent coding features to Eclipse users.
---

# GitHub Copilot for Eclipse - Copilot Instructions

## Project Overview

This is the **GitHub Copilot for Eclipse** plugin - an Eclipse IDE extension that brings AI-powered code completions, chat assistance, and intelligent coding features to Eclipse users.

### Key Features
- **Code Completions**: Real-time AI-powered code suggestions using ghost text and inline completions
- **Chat Interface**: Conversational AI assistance with agent mode, custom chat modes, and context-aware responses
- **Agent Tools**: Multi-step autonomous coding with tools for file operations, error analysis, and terminal execution
- **Language Server Integration**: LSP4E-based connection to GitHub Copilot language server
- **Authentication**: GitHub OAuth integration for Copilot subscription management
- **Multi-Platform Support**: Windows, macOS (x64/aarch64), and Linux (x64/aarch64)

### Tech Stack
- **Java**: Primary language (Java 17+, with Eclipse 4.31+ requiring Java 21+)
- **Eclipse RCP/OSGi**: Plugin architecture using Eclipse platform APIs
- **Maven/Tycho**: Build system for Eclipse plugin development
- **LSP4E**: Language Server Protocol integration for Eclipse
- **Node.js/JavaScript**: Copilot language server (agent binaries bundled per platform)
- **JDT**: Java Development Tools integration for Java-specific features
- **SWT**: Standard Widget Toolkit for UI components

## Validating Changes

You MUST verify compilation and code quality before declaring work complete!

1. **ALWAYS** run Maven build to check for compilation errors:
   ```shell
   .\mvnw clean verify
   ```

2. **CHECK** Checkstyle compliance (Google Java Style with custom rules):
   ```shell
   .\mvnw checkstyle:check
   ```

3. **RUN** tests before submitting changes:
   ```shell
   .\mvnw test
   ```

4. **FIX** all compilation errors, Checkstyle violations, and test failures before moving forward

### Maven Build Structure
- Root POM coordinates multi-module build with Tycho
- Each bundle/feature/test has its own POM inheriting from root
- Checkstyle validation runs during `verify` phase
- Target platform defined in `base.target`, `target-terminal.target`, `target-tm-terminal.target`

### Code Change Review Trigger

**When** code changes are complete (after creating, editing, or refactoring code files), automatically verify that the changes follow project best practices:

1. Review changes against the **Best Practices Summary** section above
2. Verify **Code Style Preferences** are followed (formatting, naming, imports)
3. **When modifying a function's behavior** (error handling, return type, async pattern) → check all callers using `list_code_usages` or `grep_search` to ensure they handle the new behavior correctly
4. Run build verification:
   - `.\mvnw checkstyle:check` to verify Checkstyle compliance
   - `.\mvnw clean verify` to ensure compilation succeeds
   - `.\mvnw test` to verify tests pass

**Action**: After completing code changes, review against this checklist and fix any violations before declaring work complete.

## Project Architecture

### Bundle Structure (OSGi Modules)

The project follows Eclipse plugin conventions with multiple OSGi bundles:

#### Core Bundles

**`com.microsoft.copilot.eclipse.core`** - Core functionality and language server integration
- **Package**: `com.microsoft.copilot.eclipse.core`
- **Purpose**: LSP client, authentication, chat/completion logic, persistence, utilities
- **Key Components**:
  - `lsp/` - Language server protocol client implementation
  - `chat/` - Chat modes (built-in, custom), conversation management
  - `completion/` - Code completion provider and job scheduling
  - `persistence/` - Conversation history storage with Gson serialization
  - `format/` - Language-specific formatting readers (Java, CDT)
  - `logger/` - Logging infrastructure with Eclipse console and telemetry handlers
  - `events/` - Event management for chat and auth status
  - `utils/` - Core utilities, file operations, resource management

**`com.microsoft.copilot.eclipse.ui`** - User interface and editor integration
- **Package**: `com.microsoft.copilot.eclipse.ui`
- **Purpose**: Chat view, completion UI, editor integration, agent tools
- **Key Components**:
  - `chat/` - Chat view widgets, message rendering, input handling
  - `completion/` - Ghost text rendering, code mining, completion managers
  - `chat/tools/` - Agent tools (file operations, debugging, terminal, errors)
  - `chat/services/` - Tool service manager, chat service coordination
  - `editors/` - Editor lifecycle management and integration
  - `handlers/` - Command handlers for Copilot actions
  - `preferences/` - Settings UI and preference pages
  - `quickstart/` - Onboarding and feature introduction
  - `swt/` - Custom SWT widgets (markdown rendering, code blocks)

**`com.microsoft.copilot.eclipse.ui.jobs`** - Copilot Jobs view integration
- **Package**: `com.microsoft.copilot.eclipse.ui.jobs`
- **Purpose**: Copilot Jobs view integration

**`com.microsoft.copilot.eclipse.branding`** - Product branding and about dialog
- **Package**: N/A (resources only)
- **Purpose**: Icons, about.ini, about.properties


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/copilot-for-eclipse](https://github.com/microsoft/copilot-for-eclipse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
