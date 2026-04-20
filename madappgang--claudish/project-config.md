---
trigger: always_on
description: Handles auth, endpoints, headers, rate limiting. Optionally overrides stream format for aggregators.
---

# Claudish - Development Notes

## Release Process

**Releases are handled by CI/CD** - do NOT manually run `npm publish`.

1. Bump version in `package.json`
2. Commit with conventional commit message (e.g., `feat!: v3.0.0 - description`)
3. Create annotated tag: `git tag -a v3.0.0 -m "message"`
4. Push with tags: `git push origin main --tags`
5. CI/CD will automatically publish to npm

## Build Commands

- `bun run build` - Full build (extracts models + bundles)
- `bun run build:ci` - CI build (bundles only, no model extraction)
- `bun run dev` - Development mode

## Model Routing (v4.0+)

### New Syntax: `provider@model[:concurrency]`

```bash
# Explicit provider routing
claudish --model google@gemini-2.0-flash "task"
claudish --model openrouter@deepseek/deepseek-r1 "task"

# Native auto-detection (no prefix needed)
claudish --model gpt-4o "task"          # → OpenAI
claudish --model gemini-2.0-flash "task" # → Google
claudish --model llama-3.1-70b "task"   # → OllamaCloud

# Local models with concurrency
claudish --model ollama@llama3.2:3 "task"  # 3 concurrent requests
```

### Provider Shortcuts
- `g@`, `google@` → Google Gemini
- `oai@` → OpenAI Direct
- `cx@`, `codex@` → OpenAI Codex (Responses API)
- `or@`, `openrouter@` → OpenRouter
- `mm@`, `mmax@` → MiniMax
- `mmc@` → MiniMax Coding Plan
- `kimi@`, `moon@` → Kimi
- `glm@`, `zhipu@` → GLM
- `gc@` → GLM Coding Plan
- `llama@`, `oc@` → OllamaCloud
- `litellm@`, `ll@` → LiteLLM (requires LITELLM_BASE_URL)
- `ollama@` → Ollama (local)
- `lmstudio@` → LM Studio (local)
- Custom endpoint names also work as provider prefixes (e.g., `my-vllm@model-name`) — see "Custom Endpoints" below

### Default Provider Configuration (v7.0.0+)

The default provider for auto-routing is configurable. Set it via:

- **Config file**: `"defaultProvider": "openrouter"` in `~/.claudish/config.json`
- **Env var**: `CLAUDISH_DEFAULT_PROVIDER=litellm`
- **CLI flag**: `claudish --default-provider google "task"`

**Precedence** (highest to lowest):
1. CLI flag `--default-provider`
2. `CLAUDISH_DEFAULT_PROVIDER` env var
3. `defaultProvider` in config file
4. Legacy LITELLM auto-promotion (if `LITELLM_BASE_URL` + `LITELLM_API_KEY` set without explicit `defaultProvider`)
5. `OPENROUTER_API_KEY` present → OpenRouter
6. Hardcoded `"openrouter"`

**Example config**:
```json
{
  "defaultProvider": "litellm",
  "customEndpoints": { ... }
}
```

Valid values: any built-in provider name (`"openrouter"`, `"litellm"`, `"openai"`, `"anthropic"`, `"google"`) or a custom endpoint name defined in `customEndpoints`.

**Interaction with routing rules**: When `defaultProvider` is set and no explicit `routing["*"]` catch-all exists, Claudish synthesizes `routing["*"] = [defaultProvider]` at config load time. An explicit `routing["*"]` always wins.

**Legacy behavior**: If `LITELLM_BASE_URL` and `LITELLM_API_KEY` are set but `defaultProvider` is absent, LiteLLM is still promoted to first in the fallback chain. Claudish emits a one-shot stderr hint suggesting you set `defaultProvider` explicitly.

### Vendor Prefix Auto-Resolution (ModelCatalogResolver)

API aggregators (OpenRouter, LiteLLM) require vendor-prefixed model names that users shouldn't need to know. The `ModelCatalogResolver` interface searches each aggregator's dynamic model catalog to find the correct prefix automatically.

**How it works**: User types bare model name → resolver searches the provider's already-fetched model list → finds the exact match with vendor prefix → sends the prefixed name to the API.

**Current resolvers**:
- **OpenRouter**: `or@qwen3-coder-next` → searches catalog → sends `qwen/qwen3-coder-next`
- **LiteLLM**: `ll@gpt-4o` → searches model groups → finds `openai/gpt-4o` (prefix-strip match)
- **Static fallback**: `OPENROUTER_VENDOR_MAP` for cold starts when catalog isn't loaded yet

**Key design rules**:
- Exact match only — no fuzzy/normalized matching. Find the right prefix, don't guess the model.
- Dynamic catalogs (from provider APIs) are PRIMARY. Static map is cold-start fallback only.
- Resolution happens BEFORE handler construction (in `proxy-server.ts`), not inside adapters.
- Sync entry point (`resolveModelNameSync()`) — uses in-memory caches + `readFileSync`, no async propagation.

**Firebase slim catalog** (v7.0.0+): The `aggregators[]` field on model documents provides a typed multi-provider routing index. Each entry is `{ provider, externalId, confidence }`. CLI consumers can look up `provider → externalId` directly instead of walking the `sources` array. The catalog backend lives in the [models-index](https://github.com/MadAppGang/models-index) repo.

**Adding a new aggregator resolver**: Implement `ModelCatalogResolver` interface in `providers/catalog-resolvers/`, register in `model-catalog-resolver.ts`. No changes to proxy-server or provider-resolver needed.

**Architecture doc**: `ai-docs/sessions/dev-arch-20260305-104836-a48a463d/architecture.md`

## Local Model Support

Claudish supports local models via:
- **Ollama**: `claudish --model ollama@llama3.2` (or `ollama@llama3.2:3` for concurrency)
- **LM Studio**: `claudish --model lmstudio@model-name`
- **Custom URLs**: `claudish --model http://localhost:11434/model`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MadAppGang/claudish](https://github.com/MadAppGang/claudish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
