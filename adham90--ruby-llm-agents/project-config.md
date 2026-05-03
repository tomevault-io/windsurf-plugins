---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RubyLLM::Agents is a Rails engine gem for building, managing, and monitoring LLM-powered AI agents. It provides a DSL for agent configuration, a middleware pipeline for execution, automatic tracking with cost analytics, and a mountable dashboard UI.

**Requirements:** Ruby >= 3.1, Rails >= 7.0, RubyLLM >= 1.12.0

## Common Commands

```bash
bundle exec rspec                          # Run full test suite (~3700+ specs)
bundle exec rspec spec/agents/routing_spec.rb  # Run a single spec file
bundle exec rspec spec/agents/ -e "parses"     # Run specs matching description
bundle exec standardrb                     # Lint (StandardRB, targets Ruby 3.1)
bundle exec standardrb --fix              # Auto-fix lint issues
bundle exec rake                           # Run both specs and linter (default task)
RUN_INTEGRATION=1 bundle exec rspec        # Include integration tests (skipped by default)
```

**Pre-commit hook:** A git pre-commit hook runs `standardrb --no-fix` and blocks commits on lint failures. Fix with `bundle exec standardrb --fix` before committing.

**CI:** Runs lint on Ruby 3.4, tests on Ruby 3.2/3.3/3.4.

---

## Core Design Principles

### Deep Modules Philosophy

> From John Ousterhout's *A Philosophy of Software Design*: **The best modules are deep — they hide significant complexity behind a simple interface.**

```
Module Depth = (Complexity Hidden) / (Interface Exposed)
```

A **deep module** has a small public API that hides substantial logic. A **shallow module** exposes an interface almost as complex as its implementation. **Do not create shallow modules.**

### Decision Framework: When to Create a New Abstraction

Before creating a new class, module, concern, or middleware, answer:

1. **What complexity does it hide?** If "not much" or "it just delegates," don't create it.
2. **Is the interface simpler than the implementation?** If the public API has as many concepts as the internals, it's shallow.
3. **Does it have a reason to change independently?** If it always changes in lockstep with another module, merge them.
4. **Can I name it with a specific noun/verb?** Vague names (`Manager`, `Handler`, `Processor`, `Utils`) usually signal shallow design.

### Deep vs. Shallow in This Gem

| Layer | Deep | Shallow |
|-------|------|---------|
| **Middleware** | `Reliability` — hides retries, exponential backoff, fallback model switching, circuit breaker state machine | A middleware that just logs and delegates |
| **DSL Module** | `Dsl::Reliability` — simple `on_failure { retries 3 }` hides complex config normalization and validation | A DSL module that wraps a single `attr_accessor` |
| **Concern** | `Execution::Analytics` — hides aggregate queries, trend calculations, grouping logic | `Execution::Timestamps` — just reformats `created_at` |
| **Infrastructure** | `BudgetTracker` — facade hiding query/record/forecast/alert subsystems | A class that just calls `update_column` |
| **Pipeline::Context** | Explicit data carrier — all middleware reads/writes named attributes | Would be shallow if it were just a hash wrapper |
| **Controller** | Intentionally thin — delegates to models. Controllers *should* be shallow in this gem |
| **Job** | `ExecutionLoggerJob` — thin wrapper calling deep model logic (correct; jobs are infrastructure glue) |

### Anti-Patterns to Reject

**Pass-through methods:**
```ruby
# BAD: Wraps a call without hiding anything
class AgentExecutor
  def run(agent, params)
    agent.call(**params)
  end
end
# Just call agent.call(**params) directly.
```

**Premature extraction:**
```ruby
# BAD: Concern used by only one class, hides nothing meaningful
module Pipeline::Middleware::Logging
  def log_start = Rails.logger.info("Starting")
end
# Inline it. Extract when a second middleware needs it.
```

**Needless indirection:**
```ruby
# BAD: Repository pattern over ActiveRecord
class ExecutionRepository
  def find(id) = Execution.find(id)
  def all = Execution.all
end
# ActiveRecord IS the repository. This adds a layer that hides nothing.
```

**Config objects for simple cases:**
```ruby
# BAD: Over-engineered wrapper
class RetryConfig
  attr_reader :max, :backoff
  def initialize(max: 3, backoff: :linear) = ...
end
# Use keyword arguments or constants until complexity warrants it.
```

---

## Architecture

### Class Hierarchy

```
RubyLLM::Agents::BaseAgent        # Core: middleware pipeline, DSL, execute
  └── RubyLLM::Agents::Base       # Adds before_call/after_call callbacks
        └── ApplicationAgent      # User's base class in host app
              └── ConcreteAgent   # User's agent
```

Specialized types (`Embedder`, `Speaker`, `Transcriber`, image agents) also inherit from `BaseAgent` but have their own execution logic.

### Middleware Pipeline

Agent execution flows through a middleware stack assembled by `Pipeline::Builder`:

1. **Tenant** → resolves tenant context
2. **Budget** → checks spending limits, records costs after
3. **Cache** → returns cached result or stores new one
4. **Instrumentation** → logs execution to DB via `ExecutionLoggerJob`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adham90/ruby_llm-agents](https://github.com/adham90/ruby_llm-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
