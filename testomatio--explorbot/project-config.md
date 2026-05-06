---
trigger: always_on
description: Manages multi-turn AI interactions:
---

# Explorbot - Claude Assistant Documentation

Explorbot is a Bun application that performs automated exploratory testing of web applications using AI. It combines intelligent web navigation with automatic failure recovery to test web applications without pre-written test scripts.

## Project Goal

Build a fully autonomous web testing system that can interact with any web page without human intervention.

**Main Goal**: Explorbot should work for hours on different pages of a web application, inventing and testing scenarios autonomously.

### Core Principles

- **General-purpose, not site-specific** — Explorbot must not be fine-tuned for any specific website. All solutions should be universal.
- **Rely on common web patterns** — CRUD interfaces, ARIA roles, URL conventions, React patterns, semantic HTML. These are the building blocks.
- **Never hardcode locators** — If an element can't be found, solve it through better strategies, not by adding specific selectors to source code.
- **State-based navigation** — All pages have states (URL + headings) used as anchor points for understanding where we are.
- **Adaptive scenarios** — Guess test scenarios from UI. If a scenario doesn't work, adapt and try alternatives.

### What This Means in Practice

When implementing features or fixing bugs:

1. Solutions must work across different websites, not just the one being tested
2. Prefer ARIA selectors and semantic locators over CSS/XPath
3. Use state hashing and history to detect loops and progress
4. Let AI invent scenarios based on what it sees, not predefined scripts
5. Build recovery mechanisms that learn from failures

## Code Style

**Do not write comments unless explicitly specified**

Instead of if/else try to use premature exit from loop

Example:

```js
// bad example
if (!isValid()) {
  //...
} else {
  // ...
}

// good example
if (!isValid()) {
  // ...
  return;
}
```

Safely use ?. operator, instead of multiple && && checks
Do not use try/catch inside try/catch

When updating do the smallest change possible
Avoid repetitive code patterns
Avoid ternary operators!
Never use `...(condition ? { key: value } : {})` spread pattern — use a plain `if` statement instead
Avoid creating extra functions that were not explicitly set
Use dedent when formatting prompts
Use `mdq()` from `src/utils/markdown-query.ts` for all markdown manipulation (find sections, replace tables, extract text). Never do manual line-splitting/counting on markdown.
Put types into the end of file
Introduce new types and interfaces only for module-to-module collaboration, do not add types when dealing inside one module

DUPLICATING CODE IS A SIN YOU WILL BURN IN ROBOT HELL FOR THAT! Always look if this code was already written and doesn't need to be reintroduced again
RUN code deduplication agent after each major change

## Prompts & Rules — General, Not Example-Driven

Prompts, tool descriptions, and rules must be GENERAL. They describe how things work, not what just went wrong.

- **NEVER hardcode examples from a debug session, user message, or recent bug into a prompt, tool description, or rule.** If the user reports "the AI did X with input Y", the fix is never to add Y as a counter-example in the prompt. Describe the underlying principle instead.
- **NEVER add programmatic validators that target a specific failure the user just reported.** Do not bypass AI judgment with hardcoded regexes, keyword lists, or pattern matchers derived from the bug. Fix the prompt so the model understands the distinction generally.
- Examples in prompts must illustrate the general shape of correct usage (taken from the tool's purpose), never the specific wrong input a user mentioned.
- When a user reports model confusion between two tools/commands, sharpen each tool's description to state its purpose and boundaries in general terms — do not enumerate the user's failing example.

Applies to: `src/ai/rules.ts`, every `tool({ description })` / `inputSchema` describe text, system prompts in agent files, any dedent-block feeding an AI call.

## Separation of Concerns

Follow separation of concerns principle when implementing new features:

- logic for AI agents should be inside agent classes
- shared logic for html/aria should be added to corresponding files in util/ dir
- TUI and tsx should contain only logic of TUI interaction, all business logic must be moved to corresponding agents
- tools only contain tool definitions, result parsing, etc
- CLI commands in `bin/explorbot-cli.ts` must delegate to command classes from `src/commands/` — never duplicate command logic inline in the CLI
- All reusable command logic MUST live in a command class in `src/commands/` — the CLI handler should only do minimal setup (create ExplorBot, call command.execute(), stop) with zero business logic
- avoid using `And` in a function name, if you use it probably you need 2 functions

## Architecture Overview

```
ExplorBot (DI Container)
    ├── AIProvider ─────────────> Conversation
    │
    ├── Explorer ───────────────> CodeceptJS ──> Playwright
    │       │
    │       └── StateManager
    │              ├── KnowledgeTracker
    │              └── ExperienceTracker
    │
    └── Agents (via createAgent factory)
            ├── Researcher
            ├── Navigator
            ├── Planner

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [testomatio/explorbot](https://github.com/testomatio/explorbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
