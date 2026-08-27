---
trigger: always_on
description: Developer and agent reference. For usage examples see README.md.
---

# AGENTS.md — Rasti::AI internals

Developer and agent reference. For usage examples see README.md.

## Architecture

### Provider

`Rasti::AI::Provider` is the entry point that makes provider and model a runtime value instead of a class reference. An instance holds the transport configuration (`api_key`, `usage_tracker`, `logger`, HTTP timeouts) plus the default `model`, and knows how to build the provider's `Client` and `Assistant`.

The set of providers is **fixed** — there is no registry and third-party providers are not supported. Two frozen constants in the base class drive resolution:

```ruby
MODULES = {open_ai: 'OpenAI', gemini: 'Gemini', anthropic: 'Anthropic', open_router: 'OpenRouter', huawei_maas: 'HuaweiMaaS'}.freeze
ALIASES = {openai: :open_ai, openrouter: :open_router, huawei: :huawei_maas}.freeze
```

`Provider.build(name, **options)` normalizes the name (`downcase.to_sym`), applies aliases, and resolves the class with `AI.const_get(MODULES[key])::Provider` — at call time, so there is no load-order constraint. Passing an existing `Provider` instance returns it untouched. Unknown names raise `Errors::UnknownProvider`.

`client_class` / `assistant_class` are derived from `name` through `provider_module`, so `OpenRouter::Provider` (which inherits from `OpenAI::Provider`) gets `OpenRouter::Client` and `OpenRouter::Assistant` without overriding anything.

#### Provider — methods to implement (7 total)

Public — also consumed by the `Client`:

```ruby
def name                    # :open_ai — also used by provider_module
def default_model           # from Rasti::AI config
def default_api_key         # from Rasti::AI config
def base_url                # e.g. 'https://api.anthropic.com/v1'
def parse_usage(response)   # raw response => Usage or nil
```

Private — the request/response translation:

```ruby
def request(client:, messages:, system:, model:, json_schema:, thinking:)  # calls the client, returns raw response
def encode_message(message) # {role:, content:} with generic roles => provider message
def parse_content(response) # raw response => String
```

Plus a `THINKING_LEVELS` constant (see [Thinking levels](#thinking-levels)). OpenAI-compatible providers only override `name`, `default_model` and `default_api_key`.

#### Public API

- **`Provider#generate_text(prompt:/messages:, system:, model:, json_schema:, thinking:, client:)`** — one request, no tools, no loop. Normalizes generic-role messages, extracts `system` messages into the provider's system field, and returns a `Result`.
- **`Provider#create_assistant(state:, system:, model:, tools:, mcp_servers:, json_schema:, thinking:, client:)`** — builds the provider-specific `Assistant`. `system:` is sugar for `state: AssistantState.new(context: ...)`; passing both raises `ArgumentError`.
- **`Provider#build_client`** — the provider's `Client` with the instance's transport options.
- **`Rasti::AI.provider`, `Rasti::AI.generate_text`, `Rasti::AI.create_assistant`** — thin module-level wrappers. They split transport options (used to build the provider) from call options (forwarded to the provider method).

`model:` accepts `'provider:model'` (split on the first `:`), but only when `provider:` is not given explicitly. The provider falls back to `Rasti::AI.default_provider` (`ENV['AI_DEFAULT_PROVIDER']`); nil raises `ArgumentError`. `model` on a `Provider` is only its default — every call accepts a `model:` override.

The private helpers `build_provider` and `split_provider_and_model` live in a `class << self` block with `private` inside it.

### Result

`Rasti::AI::Result` (a `Rasti::Model`) is the normalized output of `generate_text`:

| Attribute | Content |
|---|---|
| `content` | The text the model returned, or the JSON string when `json_schema` is used |
| `usage` | A `Usage` instance, from `Provider#parse_usage` |
| `raw` | The untouched provider payload |

`Provider#parse_usage` is public because the `Client` also calls it, from `track_usage`, on every response of the assistant's tool loop.

`Assistant#call` still returns a plain `String` — unchanged for compatibility.

### Generic roles

`Rasti::AI::Roles` (`USER`, `ASSISTANT`, `SYSTEM`) is the provider-agnostic vocabulary used by `generate_text(messages:)`. Each `Provider#encode_message` translates it (Gemini maps `assistant` to its `model` role; `system` messages are pulled out of the array by `split_system` and passed through the provider's system field).

The per-provider `Roles` modules still exist and are what the `Assistant` classes use. Inside `Rasti::AI::<Provider>::*`, a bare `Roles::USER` resolves to the provider's module by lexical scope — use the fully qualified `Rasti::AI::Roles::USER` for the generic one.

> ⚠️ Do not reference a provider's `Roles` constants in a constant definition inside `<provider>/provider.rb`: `multi_require` loads `provider.rb` **before** `roles.rb` (alphabetical order). Resolve them inside method bodies instead (see `Gemini::Provider#encode_role`).

#### Known duplication


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabynaiman/rasti-ai](https://github.com/gabynaiman/rasti-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
