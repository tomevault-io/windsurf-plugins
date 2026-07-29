---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HostedGPT is a Ruby on Rails 8.0 app providing a multi-provider conversational AI UI (OpenAI, Anthropic, Groq, Google Gemini, and others via OpenAI-compatible APIs). Users bring their own API keys, create assistants bound to language models, chat with streaming responses, switch models/providers mid-conversation, attach images/files, and use tool/function calling where the provider supports it.

## Commands

### Running locally
- Outside Docker: `bin/dev` starts Puma, SolidQueue worker, and tailwindcss watcher (via Overmind/Procfile.dev). Do **not** run `db:setup` directly — it won't configure encryption; `bin/dev` handles `db:prepare` correctly.
- With Docker: `docker compose up --build`.
- `just start` / `just bash` / `just overmind` / `just teardown` are Docker convenience wrappers (see `justfile`).

### Tests
- Full suite: `bin/rails test` and `bin/rails test:system` (system tests need a real browser and do not run inside Docker; they run automatically on PRs).
- Single file: `bin/rails test test/models/message_test.rb`
- Single test by line: `bin/rails test test/models/message_test.rb:42`
- Docker equivalent: `docker compose run base rails test`
- Load sample data (users/conversations/etc.) into dev DB: `bin/rails db:fixtures:load`

### Models & assistants seed data
- `models.yml` and `assistants.yml` declaratively define the language models / assistants provisioned for each user on registration (`User::Registerable`).
- Re-import after editing `models.yml`: `rails models:import` (also runs automatically on `db:prepare` / server restart).
- Export current models: `rails models:export[tmp/models.json]`.
- Similar `rails assistants:import` / `assistants:export[path]` tasks exist for `assistants.yml`.

### Linting
- RuboCop is configured (`.rubocop.yml`, target Ruby 3.3, Rails cops enabled).

## Architecture

### AI provider abstraction
- `AIBackend::<Provider>` classes in `app/services/ai_backend/` (`open_ai.rb`, `anthropic.rb`, `gemini.rb`) implement a shared interface for talking to each provider — the base module holds `tools.rb` (function-calling helpers) and `utilities.rb` (shared logic).
- `APIService#driver` (enum: `openai`/`anthropic`/`gemini`; Groq rides the `openai` driver against a different base URL) selects the backend class via `APIService#ai_backend`.
- `APIService#effective_token` falls back to a shared default key (`Setting.default_*_key`) when `Feature.default_llm_keys?` is enabled and the user hasn't set their own token.
- Adding a new provider: add an `APIService.driver` enum value + base URL constant, implement `AIBackend::<NewProvider>` mirroring an existing backend's method names (`set_client_config`, `client_method_name`, `test_execute`), add a deterministic test client stub under `test/support/test_client/`, and add default models to `models.yml`.

### Message generation flow
- `MessagesController` creates the user's message, then enqueues `GetNextAIMessageJob` to fetch the assistant's reply asynchronously (SolidQueue, optionally run in-process via `RUN_SOLID_QUEUE_IN_PUMA=true`).
- The job calls `ai_backend.new(...).stream_next_conversation_message`, accumulating chunks onto `message.content_text` and broadcasting partial updates over ActionCable (Turbo Streams) roughly every 100ms; the message is only persisted once the stream finishes.
- Generation can be cancelled mid-stream (`User#last_cancelled_message_id`); the job checks this and raises `ResponseCancelled` to stop cleanly.
- Tool/function calls come back as `content_tool_calls` on the message (not streamed incrementally). `GetNextAIMessageJob#call_tools_before_wrapping_up` executes each tool call via `AIBackend#get_tool_messages_by_calling`, persists a message per tool result, and enqueues a follow-up `GetNextAIMessageJob` so the assistant can respond to the tool output (tools implemented in `app/services/toolbox/`, e.g. `gmail.rb`, `google_tasks.rb`, `image.rb`, `memory.rb`, `open_weather.rb`).
- Provider errors are translated into user-facing chat messages (invalid key, billing/quota, blank response, connection failure) rather than raised to the user; unexpected errors retry up to 3 attempts with backoff before giving up.

### Message versioning / branching
- Conversations support editing a message and regenerating replies without losing history. Each `Message` has an `index` (position in the conversation) and a `version`; editing/regenerating creates a new `version` starting at that index, and `branched` + `branched_from_version` track where a version forked from another. See `app/models/message/version.rb` and `conversations.yml` fixtures (`:versioned` example) for the exact scheme.
- Use `conversation.messages.for_conversation_version(version_or_:latest)` rather than querying `messages` directly — it resolves the correct set of messages across branches/versions via a real SQL join, not just an association scope.

### Identity model: Person / User / Client / Credential

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AllYourBot/hostedgpt](https://github.com/AllYourBot/hostedgpt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
