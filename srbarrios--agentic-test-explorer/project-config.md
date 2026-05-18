---
trigger: always_on
description: enables semantic search in long-term memory (e.g. `embedding_model: google-genai:models/embedding-001`,
---

# AGENTS.md

## Project Overview
This repository is a product-agnostic Agentic Test Explorer Proof-of-Concept (PoC). It performs
automated, autonomous exploratory QA against any web application configured by the user. It runs
on an async Python runtime utilizing a LangGraph supervisor-worker swarm architecture, Playwright
for browser automation, and Claude (default) or Gemini models. Application-specific details
(URL, credentials, auth selectors, MCP servers, Skills) are supplied through `config.yaml`,
`.env`, and `mcp_servers.json` — none are baked into the codebase.

The framework also supports **PR-driven test generation**: given a GitHub Pull Request URL, it
extracts the code diff (preferring the **GitHub MCP server** at
`https://api.githubcopilot.com/mcp/`, falling back to the `gh` CLI) and uses an LLM to
auto-generate targeted mission YAML covering the UI areas impacted by the changes.

## Architecture & Swarm Pattern
The system employs a Supervisor-Worker pattern implemented with LangGraph, where a Supervisor
node routes tasks with structured enum outputs to specialized agents and loops workers back to
itself until a `FINISH` state is reached.

### Record-and-Translate Browser Engine
A core architectural principle is the **brain/hands separation** implemented in
`src/agentic_explorer/tools/browser/engine.py`:

* **Brain** — LangGraph agents emit strict JSON intents (e.g.,
  `{"action":"click","selector":"[data-test-subj='submitButton']"}`).
* **Hands** — The deterministic engine parses, validates, and executes each command with
  Playwright.
* **Action Tape** — Every command is appended to a per-thread, immutable
  `action_tape.jsonl` log stored in `report_<thread_id>/`.
* **Reproduction** — When a bug is found, `generate_reproduction_spec` translates the Action
  Tape into a runnable `reproduction_*.spec.ts` Playwright test file.

Supported JSON actions: `navigate`, `click`, `fill`, `press`, `select_option`, `hover`,
`wait_for`, `scroll`, `extract_text`, `snapshot`.

### Agent Types
`main.py` automatically compiles either a standard or advanced graph based on `thread_id`
keywords (`accessibility`, `a11y`, `data_heavy`, `data-heavy`, `impatient`, `returning`,
`explorer`, `chaos`, `autonomous` → advanced; everything else → standard).

### PR-Driven Scenario Generation
`src/agentic_explorer/pr_analyzer.py` provides a pipeline that sits **before** mission
execution:
1. `parse_pr_url()` — extracts `(owner, repo, number)` from a GitHub PR URL.
2. `fetch_pr_data()` — **MCP-first with gh fallback**:
   - Looks for a `"github"` entry in `mcp_servers.json` (supports both `mcpServers`/`transport`
     and `servers`/`type` formats).
   - If found, connects via `MultiServerMCPClient` and calls `get_pull_request`,
     `get_pull_request_files`, and `get_pull_request_diff` MCP tools concurrently.
   - If the MCP server is not configured, unreachable, or missing required tools, falls back
     to concurrent `gh` CLI subprocess calls.
   - Truncates diffs exceeding 100K chars.
3. `generate_missions_from_pr()` — assembles the PR data and app context into a structured
   prompt, calls the configured LLM (Claude or Gemini), parses the YAML response, validates
   the mission structure, and retries on parse failures or transient API errors.

The generated missions follow the standard YAML format and are routed to agents via the same
`thread_id`-keyword mechanism. Thread IDs use the convention `pr_{number}_{agenttype}_{nn}`.

* **Standard QA Agents** (`src/agentic_explorer/orchestration/standard_graph.py`): Three agents available for routing by the supervisor:
  * `new_user_agent` — tests onboarding flows, discoverability, default states, and empty states.
  * `power_user_agent` — uses keyboard shortcuts, bulk operations, advanced filters, edge-case workflows.
  * `adversarial_user_agent` — deliberately tries to break things (invalid inputs, back-button abuse).
* **Advanced Testing Agents** (`src/agentic_explorer/orchestration/advanced_graph.py`):
  * `accessibility_user_agent` — validates WCAG compliance, screen reader navigation, keyboard-only interaction.
  * `data_heavy_user_agent` — uploads large files, creates thousands of records, uses long strings.
  * `impatient_user_agent` — cancels operations mid-flight, refreshes during submissions, clicks buttons multiple times.
  * `returning_user_agent` — scenarios for returning users with stale sessions, cached pages, outdated bookmarks.
  * `explorer_agent` — autonomous chaos exploration; uses the full Record-and-Translate engine.

### State Management
* **Persistent Memory**: State is persisted via an SQLite checkpointer (`agent_memory.sqlite`),
  keyed by the `thread_id`. A companion `AsyncSqliteStore` (sharing the same SQLite file,
  optionally configured with an embedding index for semantic search) provides cross-session
  memory. LLM-driven memory operations (procedural reflection, agent observations) are
  powered by the **Langmem SDK**.
* **Mission Isolation**: Each mission has a unique `thread_id` to isolate its checkpoints;
  reusing a thread ID resumes the prior context. Use `--clear-checkpoints` to reset
  checkpoints while preserving learned memory, `--clear-learned` for the inverse, or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srbarrios/agentic-test-explorer](https://github.com/srbarrios/agentic-test-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
