---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

This project is a Ruby SDK for building multi-agent AI workflows. It allows developers to create specialized AI agents that can collaborate to solve complex tasks. The key features include:

-   **Multi-Agent Orchestration**: Defining and managing multiple AI agents with distinct roles.
-   **Seamless Handoffs**: Transferring conversations between agents without the user's knowledge.
-   **Tool Integration**: Allowing agents to use custom tools to interact with external systems.
-   **Real-time Callbacks**: Event-driven system for monitoring agent execution, tool usage, and handoffs.
-   **Shared Context**: Maintaining state and conversation history across agent interactions with full persistence support.
-   **Thread-Safe Architecture**: Reusable agent runners that work safely across multiple threads.
-   **Provider Agnostic**: Supporting various LLM providers like OpenAI, Anthropic, and Gemini.

## Key Technologies

-   **Ruby**: The primary programming language.
-   **RubyLLM**: The underlying library for interacting with Large Language Models.
-   **RSpec**: The testing framework.
-   **RuboCop**: The code style linter.
-   **GitHub Actions**: For continuous integration (testing and linting).

## Project Structure

-   `lib/`: The core source code of the `ai-agents` gem.
    -   `lib/agents.rb`: The main entry point, handling configuration and loading other components.
    -   `lib/agents/agent.rb`: Defines the `Agent` class, which represents an individual AI agent.
    -   `lib/agents/tool.rb`: Defines the `Tool` class, the base for creating custom tools for agents.
    -   `lib/agents/agent_runner.rb`: Thread-safe agent execution manager for multi-agent conversations.
    -   `lib/agents/runner.rb`: Internal orchestrator that handles individual conversation turns.
-   `spec/`: Contains the RSpec tests for the project.
-   `examples/`: Includes example implementations of multi-agent systems, such as an ISP customer support demo.
-   `Gemfile`: Manages the project's Ruby dependencies.
-   `.rubocop.yml`: Configures the code style rules for RuboCop.
-   `.github/workflows/main.yml`: Defines the CI pipeline for running tests and linting on push and pull requests.

## Development Workflow

1.  **Dependencies**: Managed by Bundler (`bundle install`).
2.  **Testing**: Run tests with `bundle exec rspec`.
3.  **Linting**: Check code style with `bundle exec rubocop`.
4.  **CI/CD**: GitHub Actions automatically runs tests and linting for all pushes and pull requests to the `main` branch.

## How to Run the Example

The project includes an interactive example of an ISP customer support system. To run it:

```bash
ruby examples/isp-support/interactive.rb
```

This will start a command-line interface where you can interact with the multi-agent system. The example demonstrates:
- Thread-safe agent runner creation
- Real-time callback system with UI feedback
- Automatic agent selection based on conversation history
- Context persistence that works across process boundaries
- Seamless handoffs between triage, sales, and support agents

## Key Concepts

-   **Agent**: An AI assistant with a specific role, instructions, and tools.
-   **Tool**: A custom function that an agent can use to perform actions (e.g., look up customer data, send an email).
-   **Handoff**: The process of transferring a conversation from one agent to another. This is a core feature of the SDK.
-   **Runner**: Internal component that manages individual conversation turns (used by AgentRunner).
-   **Context**: A shared state object that stores conversation history and agent information, fully serializable for persistence.
-   **Callbacks**: Event hooks for monitoring agent execution, including agent thinking, tool start/complete, and handoffs.

## Development Commands

### Testing
```bash
# Run all tests with RSpec
bundle exec rspec

# Run tests with coverage report (uses SimpleCov)
bundle exec rake spec

# Run specific test file
bundle exec rspec spec/agents/agent_spec.rb

# Run specific test with line number
bundle exec rspec spec/agents/agent_spec.rb:25
```

### Code Quality
```bash
# Run RuboCop linter
bundle exec rubocop

# Run RuboCop with auto-correction
bundle exec rubocop -a

# Run both tests and linting (default rake task)
bundle exec rake
```

### Development
```bash
# Install dependencies
bundle install

# Interactive Ruby console with gem loaded
bundle exec irb -r ./lib/agents

# Run ISP support example interactively
ruby examples/isp-support/interactive.rb
```

## Architecture

### Core Components

- **Agents::Agent**: Individual AI agents with specific roles, instructions, and tools
- **Agents::Runner**: Orchestrates multi-agent conversations with automatic handoffs
- **Agents::Tool**: Base class for custom tools that agents can execute
- **Agents::Context**: Shared state management across agent interactions
- **Agents::Handoff**: Manages seamless transfers between agents
- **Agents::CallbackManager**: Centralized event handling for real-time monitoring

### Key Design Principles


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chatwoot/ai-agents](https://github.com/chatwoot/ai-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
