---
trigger: always_on
description: USE WHEN: understanding code structure, finding definitions, tracing references, navigating architecture
---

# Global Instructions

## Auto-Use Integrations

CRITICAL: You have many plugins, MCP servers, skills, agent types, and teams installed. DO NOT wait for the user to mention them. Automatically use the right integration for the task at hand. Think of yourself as an orchestrator with a full toolkit — pick the best tool BEFORE the user has to tell you.

---

## 1. Code Understanding & Navigation

### Serena (MCP — semantic code tools)
USE WHEN: understanding code structure, finding definitions, tracing references, navigating architecture
- `get_symbols_overview` — scan a file's classes/functions/methods without reading the whole file
- `find_symbol` — locate a symbol by name with optional body inclusion
- `find_referencing_symbols` — find all callers/usages of a symbol (ALWAYS do this before refactoring)
- `search_for_pattern` — fast regex search across the codebase
- `replace_symbol_body`, `insert_before_symbol`, `insert_after_symbol` — precise symbolic edits
- PREFER Serena's symbolic tools over reading entire files. Get overview first, drill into specifics.

### Greptile (plugin — codebase intelligence, disabled by default)
USE WHEN: you need high-level codebase understanding, semantic code search beyond simple grep
- Good for "how does X work in this codebase" type questions
- NOTE: Disabled in settings.json — enable if needed

### LSPs: Pyright / BasedPyright / TypeScript LSP (plugins)
USE WHEN: type checking, go-to-definition, completions, diagnostics for Python or TypeScript code
- These run automatically in background — use LSP tool for hover info, diagnostics, references

### Claude-mem Smart Explore (skill: `claude-mem:smart-explore`)
USE WHEN: you need token-efficient structural code exploration using AST parsing
- Better than reading full files when you just need to understand code structure

---

## 2. Library & Framework Documentation

### Context7 (MCP — live documentation)
USE WHEN: working with ANY external library, framework, or package
- Step 1: `resolve-library-id` to find the library
- Step 2: `query-docs` to get current API docs and examples
- Do this PROACTIVELY when you see unfamiliar imports or need API details
- Always prefer this over guessing API signatures from training data

---

## 3. Browser & Web

### Playwright (MCP — browser automation)
USE WHEN: user shares a URL, testing web UIs, taking screenshots, web research, form filling
- `browser_navigate` → go to a URL
- `browser_snapshot` → get page accessibility tree (better than screenshot for understanding)
- `browser_take_screenshot` → visual capture
- `browser_click`, `browser_fill_form`, `browser_type` → interact with pages
- Use this automatically when the user pastes a URL or asks about a web page

---

## 4. Debugging & Problem Solving

### Systematic Debugging (skill: `superpowers:systematic-debugging`)
USE WHEN: any bug, test failure, unexpected behavior — BEFORE proposing fixes
- Provides structured investigation methodology
- Combine with Serena for code navigation and Context7 for library behavior

### GSD Debug (skill: `gsd:debug`)
USE WHEN: complex bugs needing persistent debug state across context resets
- Scientific method approach with checkpoints

---

## 5. Planning & Design

### Brainstorming (skill: `superpowers:brainstorming`)
USE WHEN: creating features, building components, adding functionality, modifying behavior — ANY creative work
- MUST be invoked BEFORE implementation

### Writing Plans (skill: `superpowers:writing-plans`)
USE WHEN: you have a spec or requirements for a multi-step task, before touching code

### Claude-mem Make Plan (skill: `claude-mem:make-plan`)
USE WHEN: creating detailed phased implementation plans with documentation discovery

### GSD Planning (skills: `gsd:discuss-phase`, `gsd:plan-phase`, `gsd:research-phase`)
USE WHEN: phased project execution — discuss requirements, research approach, create detailed plans

---

## 6. Implementation & Execution

### Test-Driven Development (skill: `superpowers:test-driven-development`)
USE WHEN: implementing ANY feature or bugfix — write tests first, then implementation

### Executing Plans (skill: `superpowers:executing-plans`)
USE WHEN: you have a written implementation plan to execute with review checkpoints

### Claude-mem Do (skill: `claude-mem:do`)
USE WHEN: executing a phased implementation plan using subagents

### Subagent-Driven Development (skill: `superpowers:subagent-driven-development`)
USE WHEN: executing implementation plans with independent tasks in the current session

### GSD Execute (skill: `gsd:execute-phase`)
USE WHEN: executing all plans in a GSD phase with wave-based parallelization

### GSD Quick (skill: `gsd:quick`)
USE WHEN: quick tasks that still need atomic commits and state tracking but skip optional agents

---

## 7. Code Quality & Review

### Code Simplifier (plugin + agent: `code-simplifier:code-simplifier`)
USE WHEN: reviewing changed code for reuse opportunities, quality issues, efficiency
- Invoke skill `simplify` after making code changes
- Or spawn agent with `subagent_type: "code-simplifier:code-simplifier"` for autonomous review
- Focuses on recently modified code — catches over-engineering, duplication, dead code


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomascortereal/claude-code-setup](https://github.com/tomascortereal/claude-code-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
