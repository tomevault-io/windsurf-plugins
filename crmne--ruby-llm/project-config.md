---
trigger: always_on
description: Define reusable AI assistants with class-based configuration, runtime context, and prompt conventions
---


# {{ page.title }}
{: .no_toc }

{{ page.description }}
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

After reading this guide, you will know:

* How to define agents with a class-based DSL
* How to use agents with plain Ruby chats and Rails-backed chats
* How runtime context works (`chat`, `inputs`, and lazy evaluation)
* How prompt conventions work in `app/prompts`
* Which methods are available on agent instances

## What Are Agents?

Agents are a class-based way to define a chat setup once and reuse it everywhere.

For example, instead of re-adding the same instructions and tools in every controller, job, or service, you define them once in an agent class and call that agent wherever you need it.

```ruby
class SupportAgent < RubyLLM::Agent
  model "{{ site.models.default_chat }}"
  instructions "You are a concise support assistant."
  tools SearchDocs, LookupAccount
end

response = SupportAgent.new.ask "How do I reset my API key?"
```

In other words, an agent is a named wrapper around the same configuration you would otherwise apply progressively with `chat.with_*` calls (`with_instructions`, `with_tools`, `with_params`, and so on).

Agents work in two modes:

* Plain Ruby mode via `.chat` (returns `RubyLLM::Chat`)
* Rails mode via `.create/.create!/.find` when `chat_model` is configured (returns your ActiveRecord chat model)

Example of Rails mode:

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat  # this activates the Rails integration
  model "{{ site.models.default_chat }}"
  instructions "You are a helpful assistant."
  tools SearchDocs, LookupAccount
end

chat = WorkAssistant.create!(user: current_user)
same_chat = WorkAssistant.find(chat.id)
```

## Defining an Agent

Create a class that inherits from `RubyLLM::Agent` and declare its configuration:

```ruby
# app/agents/work_assistant.rb
class WorkAssistant < RubyLLM::Agent
  model "{{ site.models.default_chat }}"
  instructions "You are a helpful assistant."
  tools SearchDocs, LookupAccount
  temperature 0.2
  params max_output_tokens: 256
end
```

Supported class macros:

These macros use the same arguments you already know from `RubyLLM.chat(...)` and `Chat#with_*` methods.
For example, `model` maps to `RubyLLM.chat(model:, provider:, ...)`, `tools` maps to `with_tools`, `instructions` maps to `with_instructions`, and so on.

* `model` (see [Chat Basics]({% link _core_features/chat.md %}))
* `tools` (see [Tools]({% link _core_features/tools.md %}))
* `instructions` (see [Chat Basics]({% link _core_features/chat.md %}))
* `temperature` (see [Chat Basics]({% link _core_features/chat.md %}))
* `thinking` (see [Thinking]({% link _core_features/thinking.md %}))
* `citations` (see [Citations]({% link _core_features/citations.md %}))
* `params` (see [Chat Basics]({% link _core_features/chat.md %}))
* `headers` (see [Chat Basics]({% link _core_features/chat.md %}))
* `schema` (see [Chat Basics]({% link _core_features/chat.md %}))
* `context` (see [Configuration]({% link _getting_started/configuration.md %}))
* `chat_model` (Rails-backed mode)
* `inputs` (declared runtime inputs)

`schema` supports:

* A schema class (for example `PersonSchema`) - same as `with_schema`
* A JSON schema hash - same as `with_schema`
* An inline DSL block with `schema do ... end` - agent-specific convenience

Inline DSL example:

```ruby
class CriticAgent < RubyLLM::Agent
  schema do
    string :verdict, enum: ["pass", "revise"]
    string :feedback
  end
end
```

## Runtime Context and Inputs

Agents support runtime-evaluated values using blocks and lambdas.

Declare additional runtime inputs with `inputs`:

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat
  inputs :workspace

  instructions { "You are helping #{workspace.name}" }
end
```

`chat` is always available in execution context:

* In `.chat` mode, `chat` is a `RubyLLM::Chat`
* In `.create/.create!/.find` mode, `chat` is your `chat_model` record

This enables Rails-style usage:

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat

  instructions current_date_time: -> { Time.current.strftime("%B %d, %Y") },
    display_name: -> { chat.user.display_name_or_email },
    full_name: -> { chat.user.full_name.presence || chat.user.display_name_or_email }

  tools do
    [
      TodoTool.new(chat: chat),
      GoogleDriveListTool.new(user: chat.user),
      GoogleDriveSearchTool.new(user: chat.user),
      GoogleDriveReadTool.new(user: chat.user)
    ]
  end
end
```

Important: values that depend on runtime `chat` must be lazy (blocks/lambdas), not eager class-load expressions.

## Prompt Management and Conventions

Agents have prompt conventions built in.

### Default instructions prompt

Calling `instructions` with no arguments enables default prompt lookup:

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat
  instructions
end
```

RubyLLM looks for:

* `app/prompts/work_assistant/instructions.txt.erb`

If the file exists, it is rendered and used as instructions. If it does not exist, RubyLLM raises `RubyLLM::PromptNotFoundError`.

### Prompt shorthand with locals

You can pass locals directly:

```ruby
class WorkAssistant < RubyLLM::Agent
  chat_model Chat

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crmne/ruby_llm](https://github.com/crmne/ruby_llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
