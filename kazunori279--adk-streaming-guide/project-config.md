---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a comprehensive technical guide for building real-time, bidirectional streaming AI applications using Google's Agent Development Kit (ADK). The repository contains detailed documentation covering ADK's streaming architecture and a working demo application showcasing bidirectional communication with Gemini models.

## Project Structure

```text
adk-streaming-guide/
├── .github/                       # GitHub configuration
│   ├── workflows/                # Automated workflows
│   │   ├── adk-version-monitor.yml    # Monitor ADK releases
│   │   └── claude-code-reviewer.yml   # Automated doc reviews
│   ├── current_adk_version.txt   # Tracked ADK version
│   └── WORKFLOWS.md              # Workflow documentation
├── docs/                          # Multi-part documentation guide
│   ├── part1.md            # Introduction to ADK Gemini Live API Toolkit
│   ├── part2.md  # Unified message processing
│   ├── part3.md         # Event handling with run_live()
│   ├── part4.md       # RunConfig configuration
│   └── part5.md  # Multimodal features
├── reviews/                       # Documentation review reports
├── src/bidi-demo/                # Working demo application
│   ├── app/
│   │   ├── main.py              # FastAPI WebSocket server
│   │   ├── static/              # Frontend HTML/JS/CSS
│   │   └── .env                 # Environment configuration
│   └── pyproject.toml           # Python dependencies
├── tests/e2e/                    # End-to-end tests with Chrome DevTools
├── STYLES.md                     # Documentation and code style guide
└── AGENTS.md                     # Claude Code agent configuration
```

## Key Architecture Concepts

This guide covers ADK's bidirectional streaming architecture, which consists of four key phases:

1. **Application Initialization** (once at startup): Create `Agent`, `SessionService`, and `Runner`
2. **Session Initialization** (per user connection): Get/create `Session`, create `RunConfig` and `LiveRequestQueue`, start `run_live()` event loop
3. **Bidi-streaming** (active communication): Concurrent upstream (client → queue) and downstream (events → client) tasks
4. **Termination**: Close `LiveRequestQueue`, disconnect from Live API session

The upstream/downstream concurrent task pattern is fundamental to all streaming applications in this codebase.

## Documentation Standards

**CRITICAL**: All documentation must follow the comprehensive style guidelines in `STYLES.md`.

## Demo Application

The `src/bidi-demo/` directory contains a working FastAPI application demonstrating ADK bidirectional streaming.

For setup instructions, running the server, and feature details, see [`src/bidi-demo/README.md`](src/bidi-demo/README.md).

### Deploy the demo application to adk-samples repo

To deploy the demo application to the adjacent adk-samples repo, copy all files of `src/bidi-demo` to `../adk-samples/python/agents/bidi-demo` except for the files included in `.gitignore`.

## GitHub Actions Workflows

This repository includes automated workflows for maintaining documentation compatibility with ADK updates:

### ADK Version Monitor

- **Schedule**: Runs every 12 hours to check for new ADK releases on PyPI
- **What it does**: When a new version is detected, creates a parent issue and 5 sub-issues (one per documentation part)
- **Location**: `.github/workflows/adk-version-monitor.yml`

### Claude Code Reviewer

- **Trigger**: Automatically responds to issues with the `adk-version-update` label that mention `@claude`
- **What it does**: Uses the `adk-reviewer` agent to analyze documentation for compatibility issues and posts findings
- **Location**: `.github/workflows/claude-code-reviewer.yml`

### Setup Required

To enable these workflows:

1. Install the [Claude GitHub App](https://github.com/apps/claude-code) on this repository
2. Add `ANTHROPIC_API_KEY` to repository secrets (Settings → Secrets and variables → Actions)
3. The workflows will automatically run on schedule and when triggered by issue creation

See `.github/WORKFLOWS.md` for complete setup instructions and workflow details.

## Claude Code Skills

This repository provides specialized knowledge through skill configuration files in `.claude/skills/`:

- **`bidi`** (`.claude/skills/bidi/SKILL.md`): Expert in ADK bidirectional streaming documentation and implementation
- **`google-adk`** (`.claude/skills/google-adk/SKILL.md`): Agent Development Kit (ADK) expertise for Python SDK and API reference
- **`gemini-live-api`** (`.claude/skills/gemini-live-api/SKILL.md`): Google Gemini Live API documentation and guides
- **`vertexai-live-api`** (`.claude/skills/vertexai-live-api/SKILL.md`): Google Cloud Vertex AI Live API documentation
- **`code-lint`** (`.claude/skills/code-lint/SKILL.md`): Python code linter and formatter using black, isort, and flake8
- **`docs-lint`** (`.claude/skills/docs-lint/SKILL.md`): Documentation reviewer that checks links, source code references, and style consistency

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kazunori279/adk-streaming-guide](https://github.com/kazunori279/adk-streaming-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
