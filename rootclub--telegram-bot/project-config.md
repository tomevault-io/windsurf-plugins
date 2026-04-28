---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a PHP-based Telegram bot that handles group chat interactions with various features including:
- AI-powered responses using Ollama API (Gemma 4) with modular intent dispatcher
- Order management system ("pappatoie" - food ordering)
- Image handling, vision analysis, image recall via sub-agent, and AI image generation via ComfyUI
- Quiz/trivia system with Wikipedia integration
- Event management with multi-step workflows
- TTS (Text-to-Speech) via voice clone
- Profanity moderation
- Context-aware conversation capabilities
- User profiling/memory system
- Automated cron tasks: daily recap, DJ/news, press digest

## Architecture

### Core Components

1. **Entry Point**: `bot.php` - Main webhook handler that processes Telegram updates
2. **Configuration**: `config.php` - Contains bot token, API endpoints, model settings (DO NOT deploy - contains credentials)
3. **Database**: SQLite database (`telegram_bot.sqlite`)
4. **Dispatcher**: `include/dispatcher.php` - Modular intent classifier and sub-agent router

### Module Structure (`include/` directory)

- `api.php`: Telegram Bot API wrapper functions
- `database.php`: Database initialization, schema management, migrations
- `ai.php`: Ollama AI integration — `rootbotPersona()` (shared personality), `_ai_core()` (main response), `analyzeImage()` (vision), `_saluto()` (evening recap), `_dj()` (spontaneous commentary), Wikipedia functions
- `dispatcher.php`: Modular intent classifier — loads sub-agents, builds dynamic classifier prompt, routes to handlers
- `message.php`: Message processing, command routing (elseif chain), `sendPrivateResponse()`
- `orders.php`: Food ordering system ("pappatoie") management
- `events.php`: Event creation, participation, multi-step workflows (`handleEventInput()`)
- `quiz.php`: Quiz generation, Wikipedia search, LLM generator + reviewer, `listQuizTopics()`
- `moderation.php`: Profanity detection and user moderation
- `image.php`: Image download and storage handling
- `help.php`: Help command responses
- `user_memory.php`: User profile memory system (extractors, aggregators)
- `QBertClient.php`: LLM queue client (ticket-based polling, priority levels)

### Sub-Agent System (`include/agents/` directory)

The dispatcher uses a **modular sub-agent architecture**. Each agent is a PHP file that returns a declaration array. The classifier dynamically builds its prompt from all registered agents — adding a new agent requires only creating a file in `include/agents/`, no changes to the dispatcher.

#### Agent Declaration Format

```php
return [
    'id' => 'agent_name',
    'description' => "When this agent should be triggered (in Italian, used by classifier)",
    'parameters' => [
        'param_name' => "Description of what to extract",
    ],
    'handler' => function(array $ctx, array $params): ?array { ... },
    // Optional fields:
    'default' => true,              // Fallback agent (only one)
    'enriches' => 'other_agent_id', // Enrichment pattern (runs before target agent)
    'sends_own_response' => true,   // Agent sends its own Telegram messages
];
```

#### Current Agents

- **`chat.php`** (default): Normal conversation — calls `_ai_core()` with optional enrichments
- **`wikipedia.php`** (enriches `chat`): Factual/encyclopedic questions — searches Wikipedia, injects context into chat prompt
- **`quiz.php`**: Quiz/trivia requests — calls `generateAndSendQuiz()`, sends its own poll
- **`image_query.php`**: Questions about previously shared images — matches reference via LLM light, re-analyzes with `analyzeImage()`
- **`image_gen.php`**: Image generation via ComfyUI — translates Italian prompt to English via LLM, submits to z-image turbo workflow, rate limited (6/hour per user)
- **`audio_gen.php`**: Music/song generation via ComfyUI (ACE-Step 1.5 XL Turbo) — LLM (full model) composes CAPTION/LYRICS/BPM/KEYSCALE/LANGUAGE/DURATION from the ACE-Step guide, submits workflow, sends MP3 as `sendAudio`, rate limited (6/hour per user)

#### Classifier Flow

1. User mentions bot or writes in private chat
2. `dispatchIntent()` calls `classifyIntent()` with the message + last 3 chat messages as context
3. Classifier uses `OLLAMA_MODEL_LIGHT` to return JSON: `{"intent": "...", "params": {...}}`
4. Dispatcher routes to the matching agent handler
5. Enrichment agents (e.g., wikipedia) run first, then pass data to the target agent
6. Fallback to default agent (chat) if classification fails

### Bot Personality

Shared personality defined in `rootbotPersona()` (ai.php), used across all prompts:
- Observer of humanity, curious and benevolent
- Mix of Bender (Futurama) and Sheldon (Big Bang Theory)
- Sarcastic but never rude, affectionate teasing
- Each prompt adds context-specific instructions on top of the shared persona

### Message Flow

1. Telegram sends webhook updates to `bot.php`
2. `bot.php` responds 200 immediately, then processes in background
3. **Anti-spam**: `isDuplicateMessage()` checks for duplicate messages within 60s window
4. Image analysis happens BEFORE `processMessage()` — descriptions saved to context + `image_log`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rootclub/telegram-bot](https://github.com/rootclub/telegram-bot) — distributed by [TomeVault](https://tomevault.io/claim/rootclub).
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->
