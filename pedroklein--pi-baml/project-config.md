---
trigger: always_on
description: Pi extension that bridges [BAML](https://github.com/BoundaryML/baml) (a structured output DSL for LLMs) with Pi's provider system. Enables typed LLM function calls via 3 model tiers (light/standard/heavy).
---

# AGENTS.md — pi-baml Project Guide

## Overview

Pi extension that bridges [BAML](https://github.com/BoundaryML/baml) (a structured output DSL for LLMs) with Pi's provider system. Enables typed LLM function calls via 3 model tiers (light/standard/heavy).

## Quick Context

- **What:** npm package (`pi-baml`) installable via Pi's package system
- **Why:** Typed, reliable structured output from LLMs without manual JSON parsing
- **How:** 3 model tiers configured in settings.json, resolved via Pi's ModelRegistry at call time

## Configuration

```json
{
  "baml": {
    "models": {
      "light": "github-copilot/claude-haiku-4.5",
      "standard": "github-copilot/claude-sonnet-4.6",
      "heavy": "hai-proxy/anthropic--claude-4.6-opus"
    }
  }
}
```

## Repo Structure

```
pi-baml/
├── src/
│   ├── index.ts              ← Extension factory (entry point)
│   ├── eventbus.ts           ← createPiBamlLibrary (stateless, no session_start)
│   ├── lib/
│   │   ├── types.ts          ← All shared types (zero logic)
│   │   ├── config.ts         ← parseBamlSettings()
│   │   ├── bridge.ts         ← resolveModelTier() — single resolution function
│   │   ├── executor.ts       ← createBamlExecutor() → BamlExecutor
│   │   ├── registry.ts       ← FunctionsRegistry, parseFunctionDeclarations
│   │   ├── cache.ts          ← RuntimeCache<T> (SHA-256 content hash)
│   │   ├── readme-parser.ts  ← parseReadmeDescription(), parseReadmeBody()
│   │   ├── type-parser.ts    ← parseTypeDefinitions()
│   │   └── system-prompt.ts  ← renderBamlSystemPrompt()
│   └── tools/
│       ├── baml-list.ts      ← createBamlListTool(registry)
│       ├── baml-run.ts       ← createBamlRunTool(registry, factory, settings)
│       └── baml-exec.ts      ← createBamlExecTool(settings, factory)
├── skills/
│   └── baml/
│       └── SKILL.md          ← BAML authoring skill for the agent
├── examples/                 ← Teaching examples (.baml files)
├── tests/
│   ├── unit/                 ← Unit tests (no network)
│   └── integration/          ← Real BAML runtime tests
├── docs/
│   ├── configuration.md      ← Settings.json reference
│   └── adr/                  ← Architecture Decision Records
├── package.json
├── tsconfig.json
├── tsup.config.ts
├── vitest.config.ts
└── README.md
```

## Key Technical Details

### Model Tiers

The entire model resolution is one function:

```typescript
resolveModelTier(settings, modelRegistry, tier = "standard") → { clientRegistry, bamlProvider }
```

1. Read `settings.models[tier]` → `"github-copilot/claude-haiku-4.5"`
2. `modelRegistry.find("github-copilot", "claude-haiku-4.5")` → Model object
3. `modelRegistry.getApiKeyAndHeaders(model)` → auth
4. Build `ClientRegistry` with "PiClient" as primary

### GitHub Copilot Auth (ADR-013)

BAML 0.85.0's `anthropic` provider sends auth via `x-api-key`, but GitHub Copilot requires `Authorization: Bearer`. The bridge handles this:

- **Anthropic models**: Injects `Authorization: Bearer <token>` in headers, sets `api_key` to `"not-used"`
- **OpenAI models**: Passes real token as `api_key` (BAML's `openai-generic` natively uses Bearer)
- **Always injects**: `X-Initiator`, `Openai-Intent`, `anthropic-dangerous-direct-browser-access`, `accept`

### BAML Runtime API

```typescript
import { BamlRuntime, ClientRegistry, Collector } from "@boundaryml/baml";

const runtime = BamlRuntime.fromFiles("/", { "main.baml": source }, {});
const ctx = runtime.createContextManager();

const cr = new ClientRegistry();
cr.addLlmClient("PiClient", "anthropic", {
  model: "claude-haiku-4.5",
  api_key: "not-used",  // Copilot ignores x-api-key when Authorization is present
  base_url: "https://api.individual.githubcopilot.com",
  headers: {  // Must be object, NOT JSON.stringify()
    "Authorization": "Bearer <oauth-token>",
    "User-Agent": "GitHubCopilotChat/0.35.0",
    "X-Initiator": "user",
    "Openai-Intent": "conversation-edits",
    ...
  },
});
cr.setPrimary("PiClient");

const result = await runtime.callFunction("MyFunc", args, ctx, null, cr, [collector]);
const parsed = result.parsed(false);
```

### Pi Extension API (relevant subset)

```typescript
export default function(pi: ExtensionAPI) {
  pi.registerTool({ name, description, parameters, execute });
  pi.events.emit("pi-baml:ready", libraryObject);
  // No session_start handler needed — library is stateless.
  // Consumers pass ctx.modelRegistry on each library method call.
  pi.on("before_agent_start", (event) => {
    // Append <available_baml_functions> block to system prompt
  });
}
```

### Discovery Priority

Functions are discovered from these directories (lowest → highest priority):

| Priority | Path | Group prefix |
|----------|------|--------------|
| 1 (lowest) | Pi's resolved skill paths (lazy) | `skill:` |
| 2 | `~/.agents/baml/<group>/` | none |
| 3 | `~/.pi/baml/<group>/` | none |
| 4 | `[settings.functionsDirs]` | none |
| 5 | `<cwd>/.pi/baml/<group>/` | none |
| 6 (highest) | `<cwd>/.agents/baml/<group>/` | none |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PedroKlein/pi-baml](https://github.com/PedroKlein/pi-baml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
