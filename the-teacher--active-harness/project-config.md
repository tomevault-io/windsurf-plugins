---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**ActiveHarness** is a Ruby gem (`active_harness`) for building production-grade LLM pipelines. It wraps individual LLM calls with retry logic, fallback chains, observability hooks, cost tracking, streaming, and consensus-based decisions. Works in plain Ruby and Rails. Current version: `0.2.x`.

## Commands

```bash
# Install dependencies (gem has no Gemfile — use bundler in consuming apps)
bundle install

# Run all tests
bundle exec rake test

# Build gem artifact
make build

# Bump patch version, sync, and release to RubyGems
make up

# Bump minor version
make up/minor

# Bump major version
make up/major
```

Tests live in `test/active_harness/` and use Minitest. There is no linter. Ruby 2.6+ compatibility is required — no endless `def`, no numbered block params.

## Architecture

### Three Primary Abstractions

**`ActiveHarness::Agent`** (`lib/active_harness/agent.rb` + `lib/active_harness/agent/`)
Single LLM call with a model chain, system prompt, hooks, retry policy, and optional streaming. Call pattern: `MyAgent.call(input: "...", context: {}, params: {}, memory: nil, models: nil, token: nil, stream: nil)`. Returns `self`; result is on `agent.result` (a `Result` struct). Can also be instantiated and called inline: `agent.call("new input")`.

**`ActiveHarness::Tribunal`** (`lib/active_harness/tribunal.rb` + subdirs)
Runs multiple agents in parallel via `concurrent-ruby`'s `Concurrent::Future`. Computes a verdict from all successful results using either a `:unanimous`/`:majority` built-in strategy or a custom `process { |results| ... }` block. Tolerates partial failure via `may_fail: N`. Returns `self`; verdict is on `tribunal.verdict`.

**`ActiveHarness::Pipeline`** (`lib/active_harness/pipeline.rb` + subdirs)
Sequential chain of agents and tribunals as named steps. Each step can have a `stop_if` lambda; when it fires, `pipeline.stopped?` becomes true and subsequent steps are skipped. Step results accumulate in `pipeline.step_results` and are also forwarded as `context[step_name]` to downstream steps. Each step's output replaces the payload only when `step.transform?` is true (i.e., a `use` agent rather than a tribunal that only produces a verdict).

### Supporting Components

**Prompt classes** (`app/ai/prompts/` in Rails) — Plain Ruby objects with `#call` (or `#text`) that return strings. Agent injects `@input`, `@context`, `@params`, `@memory`, `@context_window`, and `@config` into prompt instances before calling them. Also accepts inline strings or lambdas.

**`ActiveHarness::Memory`** (`lib/active_harness/memory.rb`) — Conversation history storage. Must be subclassed (`Memory::JsonFile`, `Memory::Postgresql`, `Memory::Sqlite`). Records turns automatically after each successful agent/pipeline call. **Memory is never auto-injected into LLM messages** — always inject manually via a hook, prompt class, or `before_call` block.

**`ActiveHarness::Result`** — `Struct` wrapping: `input`, `output`, `processed`, `system_prompt`, `model` (a `ModelInfo` struct with `.name`, `.provider`, `.temperature`, `.context_window`, `.pricing`), `model_list`, `attempts`, `execution_time`, `usage` (a `UsageInfo` struct with `.tokens.{input,output,total}` and `.cost.{input,output,total}`). For `format :json` agents, `processed` is a parsed Ruby Hash/Array; for `:text` it's the raw string.

**Providers** (`lib/active_harness/providers/`) — One file per provider, all subclassing `Providers::Base`. Supported symbols: `:openai`, `:anthropic`, `:gemini`, `:groq`, `:openrouter`, `:xai`, `:deepseek`, `:mistral`, `:ollama`, `:perplexity`, `:gpustack`, `:azure`, `:bedrock`, `:vertexai`, `:custom`. To add a provider: subclass `Providers::Base`, implement `#call`, register in `agent/providers.rb` `PROVIDERS` hash.

**`ActiveHarness::Pricing`** — Pulls pricing from `models.dev` API (cached 24h in `tmp/active_harness/pricing.json`), falls back to bundled `lib/active_harness/data/models.json`.

### Hooks System

All three abstractions share `Core::HookRunner`. Hooks are arrays — multiple registrations accumulate, all fire in order. Blocks run via `instance_exec` so they have access to agent/pipeline/tribunal instance variables.

Agent hooks: `:setup`, `:before_call`, `:after_call`, `:before_system_prompt`, `:after_system_prompt`, `:before_parse`, `:after_parse`, `:parse_error`, `:retry`, `:failure`.
Rails-style aliases: `before :call`, `after :call`, `callback :setup`, etc.

Pipeline hooks: `:before_step`, `:after_step`, `:stopped`, `:complete`. Can be scoped to a specific step: `on :before_step, :translate do |payload| ... end`.

Tribunal hooks: `:before_call`, `:after_call`, `:before_agent`, `:after_agent`, `:agent_error`, `:before_verdict`, `:after_verdict`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-teacher/active_harness](https://github.com/the-teacher/active_harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
