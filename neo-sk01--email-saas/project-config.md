---
trigger: always_on
description: Comprehensive reference for Taskmaster MCP tools and CLI commands.
---

# Taskmaster Tool & Command Reference

This document provides a detailed reference for interacting with Taskmaster, covering both the recommended MCP tools, suitable for integrations like Cursor, and the corresponding `task-master` CLI commands, designed for direct user interaction or fallback.

**Note:** For interacting with Taskmaster programmatically or via integrated tools, using the **MCP tools is strongly recommended** due to better performance, structured data, and error handling. The CLI commands serve as a user-friendly alternative and fallback. 

**Important:** Several MCP tools involve AI processing... The AI-powered tools include `parse_prd`, `analyze_project_complexity`, `update_subtask`, `update_task`, `update`, `expand_all`, `expand_task`, and `add_task`.

---

## Initialization & Setup

### 1. Initialize Project (`init`)

*   **MCP Tool:** `initialize_project`
*   **CLI Command:** `task-master init [options]`
*   **Description:** `Set up the basic Taskmaster file structure and configuration in the current directory for a new project.`
*   **Key CLI Options:**
    *   `--name <name>`: `Set the name for your project in Taskmaster's configuration.`
    *   `--description <text>`: `Provide a brief description for your project.`
    *   `--version <version>`: `Set the initial version for your project, e.g., '0.1.0'.`
    *   `-y, --yes`: `Initialize Taskmaster quickly using default settings without interactive prompts.`
*   **Usage:** Run this once at the beginning of a new project.
*   **MCP Variant Description:** `Set up the basic Taskmaster file structure and configuration in the current directory for a new project by running the 'task-master init' command.`
*   **Key MCP Parameters/Options:**
    *   `projectName`: `Set the name for your project.` (CLI: `--name <name>`)
    *   `projectDescription`: `Provide a brief description for your project.` (CLI: `--description <text>`)
    *   `projectVersion`: `Set the initial version for your project, e.g., '0.1.0'.` (CLI: `--version <version>`)
    *   `authorName`: `Author name.` (CLI: `--author <author>`)
    *   `skipInstall`: `Skip installing dependencies. Default is false.` (CLI: `--skip-install`)
    *   `addAliases`: `Add shell aliases tm and taskmaster. Default is false.` (CLI: `--aliases`)
    *   `yes`: `Skip prompts and use defaults/provided arguments. Default is false.` (CLI: `-y, --yes`)
*   **Usage:** Run this once at the beginning of a new project, typically via an integrated tool like Cursor. Operates on the current working directory of the MCP server. 
*   **Important:** Once complete, you *MUST* parse a prd in order to generate tasks. There will be no tasks files until then. The next step after initializing should be to create a PRD using the example PRD in .taskmaster/templates/example_prd.txt. 

### 2. Parse PRD (`parse_prd`)

*   **MCP Tool:** `parse_prd`
*   **CLI Command:** `task-master parse-prd [file] [options]`
*   **Description:** `Parse a Product Requirements Document, PRD, or text file with Taskmaster to automatically generate an initial set of tasks in tasks.json.`
*   **Key Parameters/Options:**
    *   `input`: `Path to your PRD or requirements text file that Taskmaster should parse for tasks.` (CLI: `[file]` positional or `-i, --input <file>`)
    *   `output`: `Specify where Taskmaster should save the generated 'tasks.json' file. Defaults to '.taskmaster/tasks/tasks.json'.` (CLI: `-o, --output <file>`)
    *   `numTasks`: `Approximate number of top-level tasks Taskmaster should aim to generate from the document.` (CLI: `-n, --num-tasks <number>`)
    *   `force`: `Use this to allow Taskmaster to overwrite an existing 'tasks.json' without asking for confirmation.` (CLI: `-f, --force`)
*   **Usage:** Useful for bootstrapping a project from an existing requirements document.
*   **Notes:** Task Master will strictly adhere to any specific requirements mentioned in the PRD, such as libraries, database schemas, frameworks, tech stacks, etc., while filling in any gaps where the PRD isn't fully specified. Tasks are designed to provide the most direct implementation path while avoiding over-engineering.
*   **Important:** This MCP tool makes AI calls and can take up to a minute to complete. Please inform users to hang tight while the operation is in progress. If the user does not have a PRD, suggest discussing their idea and then use the example PRD in `.taskmaster/templates/example_prd.txt` as a template for creating the PRD based on their idea, for use with `parse-prd`.

---

## AI Model Configuration

### 2. Manage Models (`models`)
*   **MCP Tool:** `models`
*   **CLI Command:** `task-master models [options]`
*   **Description:** `View the current AI model configuration or set specific models for different roles (main, research, fallback). Allows setting custom model IDs for Ollama and OpenRouter.`
*   **Key MCP Parameters/Options:**
    *   `setMain <model_id>`: `Set the primary model ID for task generation/updates.` (CLI: `--set-main <model_id>`)
    *   `setResearch <model_id>`: `Set the model ID for research-backed operations.` (CLI: `--set-research <model_id>`)
    *   `setFallback <model_id>`: `Set the model ID to use if the primary fails.` (CLI: `--set-fallback <model_id>`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Neo-sk01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
