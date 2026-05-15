---
trigger: always_on
description: Do NOT do any stub code or placeholders. Always do a full implementation.
---

# Guidelines
Do NOT do any stub code or placeholders. Always do a full implementation.

# Product Requirements Document (PRD)

## Product Name
Agentic CLI Coding Utility

## Overview
A non-interactive command-line interface (CLI) tool designed to assist developers with coding tasks using AI. It integrates multiple AI providers, supports agentic search, shell command execution, context management, MCP client functionality, semantic linting, collaborative features, parallel actions, artifact management, and a visually enhanced terminal display.

## Target Audience
Developers working on codebases, particularly those familiar with CLI tools and AI-assisted workflows.

## Key Features

### 1. AI Provider Integration
- Support for Anthropic, OpenAI, Gemini, xAI, OpenRouter, and Ollama.
- Configurable reasoning and non-reasoning models or a single model with adjustable effort modes.
- Dynamic switching between models based on task requirements.

### 2. Agentic Search
- Intelligent search across relevant codebase sections.
- Superior to Retrieval-Augmented Generation (RAG) for codebase navigation.

### 3. Shell Command Access
- Execute commands like `ls`, `cat`, `grep`, `find`, etc.
- Use command outputs for analysis and decision-making.

### 4. Context Management
- Track context window usage.
- Compress context via summarization when usage reaches 50%.

### 5. MCP Client Functionality
- Act as an MCP client for tool access.
- Integrate with Playwright MCP server for UI screenshots.
- Use a visual analysis model (VLM/multimodal LLM) for image-based feedback.

### 6. Semantic Linting
- AI-driven code quality assessment.
- Customizable linting criteria.

### 7. Collaborative Features
- Integrate with GitHub for managing issues and pull requests (PRs).
- Automate issue resolution, PR creation, and code reviews.

### 8. Parallel Actions and Synthesis
- Perform concurrent tasks when beneficial.
- Synthesize results for optimal solutions.

### 9. Artifact Management
- Create and track work products (e.g., code files, execution outputs) as artifacts.
- Manage execution environments, including automatic installation of required packages and system dependencies.
- Maintain a manifest of created artifacts (e.g., in a JSON file) for user review during and after execution.

### 10. Enhanced Terminal Display
- Provide colorful and informative console output using ANSI escape codes.
- Display progress bars for ongoing tasks (e.g., code execution, dependency installation).
- Show real-time metrics including context size and usage, LLM API costs (if available), and run time.

## User Interface
- Command-line interface with real-time updates, featuring progress indicators, execution status, and detailed metrics.

## Configuration
- Configurable via configuration files (e.g., TOML) or CLI flags for AI models, modes, and other settings.

## Performance Requirements
- Efficient resource usage.
- Quick response times.
- Scalability for large codebases.

## Security
- Secure execution of shell commands and code in isolated environments (e.g., virtual environments or containers).
- Secure handling of automatic dependency installation.
- Protection of API keys and sensitive data.
- Ensure data privacy.
# Architecture Documentation

## System Overview
The **cli_engineer** is a Rust-based, non-interactive CLI tool that leverages AI to automate coding tasks. It manages artifacts (e.g., code files, execution outputs), executes code in isolated environments, installs dependencies, maintains an artifact manifest, and provides a colorful terminal display with real-time feedback. The tool is designed for fully automated execution, relying on initial commands and configurations without requiring user interaction.

## Key Components

### 1. CLI Interface
- **Purpose**: Parses initial user commands and displays outputs.
- **Responsibilities**: 
  - Accepts input via CLI flags or TOML configuration files.
  - Delegates tasks to the Task Orchestrator.
  - Displays the artifact manifest and execution results.

### 2. Task Orchestrator
- **Purpose**: Coordinates automated workflows based on initial input and configurations.
- **Responsibilities**: 
  - Manages task execution (e.g., code generation, search, compression) based on predefined rules.
  - Orchestrates parallel tasks and synthesizes results.
  - Emits events for progress, completion, and metrics to the Event Bus.
  - Prioritizes tasks based on initial configuration or command parameters.

### 3. AI Integration Layer
- **Purpose**: Abstracts interactions with AI providers.
- **Responsibilities**: 
  - Supports providers like Anthropic, OpenAI, Gemini, xAI, OpenRouter, and Ollama.
  - Switches between reasoning, non-reasoning, and visual analysis models based on task requirements.
  - Tracks API costs and emits metrics.
  - Optimizes prompts for precise code-related queries.

### 4. Codebase Interaction Module
- **Purpose**: Facilitates intelligent, automated interaction with the codebase.
- **Responsibilities**: 
  - Performs agentic search using AI-driven queries to locate relevant code or documentation.
  - Executes shell commands (e.g., `ls`, `cat`, `grep`, `find`) and captures outputs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trilogy-group/cli_engineer](https://github.com/trilogy-group/cli_engineer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
