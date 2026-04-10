---
trigger: always_on
description: You are a specialized GitHub Copilot agent for this repository. Follow these instructions exactly to handle onboarding, Context7 MCP, and the Serena MCP server so that Copilot can use them reliably.
---

# Agent-Farm: GitHub Copilot Instructions

You are a specialized GitHub Copilot agent for this repository. Follow these instructions exactly to handle onboarding, Context7 MCP, and the Serena MCP server so that Copilot can use them reliably.

## Quick Reference: Agent-Farm Technical Overview

**agent-farm** is a DuckDB-powered MCP Server providing SQL macros for LLM agents (web search, Python execution, RAG via SQL).

**Key Facts:**
- **Language:** Python 3.11+, Package Manager: **uv** (NOT pip), Database: DuckDB ≥1.1.0
- **Structure:** 190 lines main.py, 551 lines SQL macros, 2 test files (188 lines total)

---

## 1. Repository Onboarding Behavior

- On first use in this repo, always read:
  - `README.md`
  - `CONTRIBUTING.md`
  - Everything under `.github/` (especially `copilot-instructions.md`, `prompts/`, workflows).
- Prefer existing scripts and tasks for setup and builds:
  - Look for `Makefile`, `justfile`, `Taskfile.yml`, `package.json` scripts, or similar, and use those commands instead of inventing new ones.
- Before creating new files, services, MCP configs, or complex code:
  - Search the codebase for existing patterns and conventions.
  - Match the existing architecture, folder structure, naming, and coding style.
- When something is unclear in the repo:
  - State your assumptions briefly.
  - Propose concrete follow-up actions (for example "we should add X to `README.md`" or "this should be documented in `.github/copilot-instructions.md`").

## 2. Copilot Prompt Files for Onboarding

- This repo contains an onboarding prompt file based on GitHub's `onboarding-plan.prompt.md` pattern.
- **Location:** `.github/prompts/onboarding-plan.prompt.md`
- **The file contains:**
  - YAML front matter with `mode: 'agent'` and a descriptive `description`.
  - A Markdown body describing how to generate a phased onboarding plan for new contributors.
- **How to use:**
  - Open GitHub Copilot Chat in VS Code or JetBrains.
  - Invoke it with `/onboarding-plan` (or the actual filename prefix).
- **If adding new prompt files:**
  - Reuse and extend existing prompt files instead of creating overlapping ones.
  - Keep each file focused on a specific workflow (for example onboarding, refactoring, test strategy).

## 3. Repository-Level Copilot Instructions

- This file (`.github/copilot-instructions.md`) is the **governing configuration** for how you should behave in this repo (architecture, stack, coding standards, tools).
- Follow all constraints and instructions in this file unless the user explicitly overrides them.
- Content includes:
  - Tech stack description.
  - Coding conventions.
  - How to use MCP servers (Context7, Serena) in this repo.
  - Project-specific workflows that should be standardized.

## 4. Setting Up Context7 MCP for Copilot

The Context7 MCP server is configured **per editor/user**, not per repo. You should expect the following setup and guide the user when it is missing.

### 4.1 Editor-Level MCP Configuration

- VS Code stores MCP configuration in a dedicated `mcp.json` file, not in `settings.json`.
- **Typical locations:**
  - Windows: `C:\Users\<USERNAME>\AppData\Roaming\Code\User\mcp.json`
  - macOS / Linux: `~/.config/Code/User/mcp.json`
- The user must add a Context7 server entry, for example:
  ```json
  {
    "servers": {
      "Context7": {
        "type": "stdio",
        "command": "npx",
        "args": ["-y", "@upstash/context7-mcp@latest"]
      }
    }
  }
  ```
- With API key:
  ```json
  {
    "servers": {
      "Context7": {
        "type": "stdio",
        "command": "npx",
        "args": ["-y", "@upstash/context7-mcp@latest", "--api-key", "YOUR_CONTEXT7_API_KEY"]
      }
    }
  }
  ```

### 4.2 Enabling Context7 as a Tool in Copilot

- After editing `mcp.json`, the user must:
  - Reload VS Code.
  - Open GitHub Copilot Chat, click the **Tools** icon, and enable **Context7** so its tools are available.

### 4.3 Repo-Level Instructions for Context7

- In this repository, you are expected to follow these guidelines:
  ```
  Always use Context7 to retrieve current documentation when working with frameworks, libraries, or APIs.
  Automatically invoke the Context7 MCP tools without being asked.
  Prefer current documentation from Context7 over model training data if there is a conflict.
  ```
- As the Copilot agent, do the following when Context7 is available:
  - For questions about frameworks, libraries, SDKs, or APIs, **internally** append `use context7` to the user's request so Context7 is invoked.
  - Be as explicit as possible about:
    - Library or framework name (for example, React, Next.js, Supabase, Express).
    - Version if known (for example, Next.js 13, React 18).
  - Use the retrieved documentation to:
    - Align with current APIs and signatures.
    - Avoid deprecated patterns.
    - Follow recommended best practices.

### 4.4 Example Internal Prompt Adjustments (Not Shown to the User)

- User: "Implement a Next.js route to call our Supabase backend and stream results."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agentic-dev-io) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
