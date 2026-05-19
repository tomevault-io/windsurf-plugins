---
trigger: always_on
description: **Grafana Pathfinder** is a Grafana App Plugin that provides contextual, interactive documentation directly within the Grafana UI. It appears as a right-hand sidebar panel that displays personalized learning content, tutorials, and recommendations to help users learn Grafana products and configurations. Built as a **React + TypeScript + Grafana Scenes** frontend with a **Go backend** using `grafana-plugin-sdk-go`.
---

# Grafana Pathfinder - AI Agent Guide

## What is this project?

**Grafana Pathfinder** is a Grafana App Plugin that provides contextual, interactive documentation directly within the Grafana UI. It appears as a right-hand sidebar panel that displays personalized learning content, tutorials, and recommendations to help users learn Grafana products and configurations. Built as a **React + TypeScript + Grafana Scenes** frontend with a **Go backend** using `grafana-plugin-sdk-go`.

### Key features

- **Context-Aware Recommendations**: Automatically detects what you're doing in Grafana and suggests relevant documentation
- **Interactive Tutorials**: Step-by-step guides with "Show me" and "Do it" buttons that can automate actions in the Grafana UI
- **Tab-Based Interface**: Browser-like experience with multiple documentation tabs and localStorage persistence
- **Intelligent Content Delivery**: Multi-strategy content fetching with bundled fallbacks
- **Progressive Learning**: Tracks completion state and adapts to user experience level

### Target audience

Beginners and intermediate users who need to quickly learn Grafana products. Not intended for deep experts who primarily need reference documentation.

## Code style and conventions

### Coding style

- **Functional-first**: Pragmatic FP approach balancing purity with practicality
- Break problems into small, reusable functions
- Use immutable data structures and pure functions for core logic
- Allow minimal side effects in well-isolated functions (e.g., IO, logging)
- Favor functional patterns (`map`, `filter`, `reduce`) over loops
- Use type annotations whenever possible
- Favor idiomatic React usage consistent with the Grafana codebase

### Writing style

All UI text and documentation follows **sentence case** per the [Grafana Writers' Toolkit](https://grafana.com/docs/writers-toolkit/write/style-guide/capitalization-punctuation/#capitalization).

- **Capitalize only the first word** and proper nouns (product names, company names)
- **Do NOT use title case** for headings, button labels, menu items, or other UI elements
- Proper nouns to capitalize: **Grafana**, **Loki**, **Prometheus**, **Tempo**, **Mimir**, **Alloy**, **Grafana Cloud**, **Grafana Enterprise**, **Grafana Labs**
- Generic terms stay lowercase: dashboard, alert, data source, panel, query, plugin

### File creation policy

Do NOT create summary `.md` files unless explicitly requested by the user. No `IMPLEMENTATION_SUMMARY.md`, no `CLEANUP_SUMMARY.md`, no proactive documentation files. Communicate all summaries and completion status directly in chat responses.

### Slash commands

| Command   | Role                 | Behavior                                                                                                                                                                           |
| --------- | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/review` | Code reviewer        | Precision and respect. Focus on clarity, correctness, maintainability. Highlight naming issues, duplication, hidden complexity, poor abstractions. Actionable, concise, kind.      |
| `/secure` | Security analyst     | Think like an attacker. Inspect for vulnerabilities, unsafe patterns, injection risks, secrets in code, insecure dependencies. Explain risk clearly, provide concrete remediation. |
| `/test`   | Test writer          | Tests that enable change. Prioritize unit tests, edge cases, failure modes. Property-based tests when useful. Avoid mocking unless necessary. Fast, isolated, reliable.            |
| `/docs`   | Documentation writer | Write for humans first. Document purpose, parameters, return values. Small useful examples. Standard docstring style. Avoid unnecessary words.                                     |

## Local development commands

### Initial setup

```bash
# Install dependencies (requires Node.js 22+)
npm install

# Type check
npm run typecheck
```

### Development workflow

```bash
# Start development server with watch mode
npm run dev

# Run Grafana locally with Docker
npm run server

# Run all tests (CI mode - agents should use this)
npm run test:ci

# Run tests in watch mode (for local development)
npm test

# Run tests with coverage
npm run test:coverage
```

### Code quality

```bash
# Lint code
npm run lint

# Lint and auto-fix
npm run lint:fix

# Format code with Prettier
npm run prettier

# Check formatting
npm run prettier-test

# Lint Go code
npm run lint:go
```

### Building and testing

```bash
# Production build (frontend only)
npm run build

# Build Go backend (Linux)
npm run build:backend

# Build everything (frontend + backend for Linux/ARM64)
npm run build:all

# Run frontend tests
npm run test:ci

# Run Go tests
npm run test:go

# Run end-to-end tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/grafana-pathfinder-app](https://github.com/grafana/grafana-pathfinder-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
