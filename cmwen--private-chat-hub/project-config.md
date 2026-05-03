---
trigger: always_on
description: This document provides guidance for AI agents and automated tools working with this Android Flutter repository. It serves as the primary rules file for both [OpenCode](https://opencode.ai) and GitHub Copilot.
---

# Agent Configuration and Instructions

This document provides guidance for AI agents and automated tools working with this Android Flutter repository. It serves as the primary rules file for both [OpenCode](https://opencode.ai) and GitHub Copilot.

## Quick Reference for AI Agents

**First-Time Users**: Direct them to [GETTING_STARTED.md](GETTING_STARTED.md) for complete setup guide.

**Customization**: Use [APP_CUSTOMIZATION.md](APP_CUSTOMIZATION.md) for comprehensive customization checklist with AI prompts.

**Important Files to Update When Renaming App**:
- `pubspec.yaml` (name, description)
- `lib/main.dart` (imports, title)
- `test/widget_test.dart` (imports)
- `android/app/build.gradle.kts` (namespace, applicationId)
- `android/app/src/main/AndroidManifest.xml` (label)

## Repository Overview

This is **Private Chat Hub** - a privacy-first Android app for chatting with self-hosted AI models via Ollama. Built with Flutter, it features an optimized build system (Java 17, parallel builds, caching), comprehensive CI/CD, and AI-powered development workflow.

## AI Agent Tooling

This project supports two AI agent platforms with parallel configurations:

| Platform | Config | Agents | Skills/Commands |
|----------|--------|--------|-----------------|
| **OpenCode** | `opencode.json` + `.opencode/` | `.opencode/agents/*.md` | `.opencode/skills/`, `.opencode/commands/` (in opencode.json) |
| **GitHub Copilot** | `.github/agents/*.agent.md` | `.github/agents/` | `.github/skills/` |

### OpenCode Setup

OpenCode reads this `AGENTS.md` file automatically as project rules. Additional configuration:

- **Config**: `opencode.json` - formatter, custom commands, instruction files
- **Agents**: `.opencode/agents/` - 6 specialized agents (flutter-developer, architect, researcher, product-owner, experience-designer, doc-writer)
- **Skills**: `.opencode/skills/` - 4 skills (build-fix, android-debug, ci-debug, icon-generation)
- **Commands**: `/test`, `/build`, `/analyze`, `/fix`, `/clean`, `/check`

Switch agents with **Tab** in the TUI. Use `@agent-name` to invoke subagents. Use `/command` for quick actions.

### GitHub Copilot Agents

6 specialized agents in `.github/agents/`:

| Agent | Purpose |
|-------|---------|
| **@product-owner** | Define features, requirements, user stories |
| **@experience-designer** | Design UX, user flows, wireframes |
| **@architect** | Plan architecture, technical decisions |
| **@researcher** | Research packages, best practices |
| **@flutter-developer** | Implement features, write tests, debug |
| **@doc-writer** | Create documentation, guides |

### Skills (Both Platforms)

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| **build-fix** | Diagnose build failures | Gradle errors, dependency conflicts, compilation issues |
| **android-debug** | Debug Android app issues | App crashes, device issues, performance problems |
| **ci-debug** | Fix GitHub Actions failures | Workflow failures, CI-specific build errors |
| **icon-generation** | Generate app icons and launcher assets | Creating UI or launcher icons |

**OpenCode location**: `.opencode/skills/` | **Copilot location**: `.github/skills/`

## Key Technologies

- **Framework**: Flutter 3.10.1+
- **Language**: Dart 3.10.1+
- **Build System**: Java 17, Gradle 8.0+ with parallel builds and caching
- **Platform**: Android
- **Package Manager**: pub (pubspec.yaml)
- **Testing**: flutter_test, Widget testing, Integration testing
- **Linting**: flutter_lints 6.0.0
- **CI/CD**: GitHub Actions with caching and concurrency control

## Project Structure

```
├── lib/                    # Dart source code
│   └── main.dart           # App entry point
├── test/                   # Unit and widget tests
├── android/                # Android platform files
├── docs/                   # AI prompting guides
├── scripts/                # Automation scripts
│   ├── signing/            # Signing-related scripts
│   │   ├── generate-keystore.sh    # Generate keystore locally
│   │   └── persist-credentials.sh  # Persist auto-generated credentials
│   ├── setup/              # Setup scripts
│   └── release/            # Release scripts
├── .opencode/              # OpenCode AI agent configuration
│   ├── agents/             # Specialized agents (flutter-developer, architect, etc.)
│   └── skills/             # Reusable skills (build-fix, android-debug, etc.)
├── opencode.json           # OpenCode config (formatter, commands, instructions)
├── .github/                # CI/CD workflows and Copilot agents
│   ├── workflows/          # GitHub Actions
│   ├── actions/            # Custom composite actions
│   │   └── setup-signing/  # Auto-generate or use existing keystore
│   ├── agents/             # Copilot Chat agents
│   ├── skills/             # Copilot agent skills (task-specific workflows)
│   └── prompts/            # Copilot custom prompts (legacy)
├── audit/                  # LLM-generated implementation summaries and logs
└── pubspec.yaml            # Dependencies and project config
```

## AI Customization Points


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cmwen/private-chat-hub](https://github.com/cmwen/private-chat-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
