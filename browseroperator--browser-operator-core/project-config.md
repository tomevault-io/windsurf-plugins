---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Browser Operator** is an AI-native browser built on Chrome DevTools frontend. It adds a multi-agent AI framework to the DevTools panel, enabling intelligent automation and web interaction through specialized AI agents.

## Build & Development Commands

### Initial Setup

```bash
# Prerequisites: depot_tools in PATH (https://chromium.googlesource.com/chromium/tools/depot_tools.git)
gclient sync
npm install
cp .env.example .env  # Configure API keys
```

### Build

```bash
npm run build                    # Standard build (runs gn gen automatically)
npm run build -- --watch         # Watch mode for development
npm run build -- -t Debug        # Build to out/Debug instead of out/Default

# Fast build (skip type checking and bundling)
gn gen out/fast-build --args="devtools_skip_typecheck=true devtools_bundle=false"
npm run build -- -t fast-build
```

### Running DevTools with Custom Build

```bash
# Terminal 1: Build with watch
npm run build -- --watch

# Terminal 2: Serve the built files
cd out/Default/gen/front_end && python3 -m http.server 9000

# Terminal 3: Launch Browser Operator with custom DevTools
/Applications/Browser\ Operator.app/Contents/MacOS/Browser\ Operator \
  --disable-infobars \
  --custom-devtools-frontend=http://localhost:9000/ \
  --remote-debugging-port=9222
```

### Testing

```bash
npm run test                                      # Unit tests (Karma/Mocha)
npm run webtest                                   # E2E tests (Puppeteer)
npm run debug-webtest -- --spec=path/to/test     # Debug specific test
npm run lint                                      # ESLint
```

### Eval Runner (Agent Testing)

**Recommended: Use the eval-runner-analyst agent** to run evals and get detailed analysis:

```
# In Claude Code, use the Task tool with eval-runner-analyst agent:
"Run the action agent evals with cerebras gpt-oss-120b"
"Test action-agent-checkbox-001 and action-agent-form-001"
"Compare V0 and V1 action agents on iframe tests"
```

The eval-runner-analyst agent handles the complete workflow: running tests, collecting results, and providing detailed analysis of pass/fail patterns.

**Manual CLI usage** (if needed):

The eval runner automatically loads environment variables from `.env` in the project root.

```bash
# Run agent evaluations (launches headless Chrome by default)
npx tsx scripts/eval-runner/cli.ts --tool action_agent --verbose
npx tsx scripts/eval-runner/cli.ts --test action-agent-click-001 --verbose

# Use Cerebras for fast inference (preferred models: zai-glm-4.6, gpt-oss-120b)
npx tsx scripts/eval-runner/cli.ts --provider cerebras --model zai-glm-4.6 --tool action_agent
npx tsx scripts/eval-runner/cli.ts --provider cerebras --model gpt-oss-120b --tool action_agent

# Run V0 agent variant
npx tsx scripts/eval-runner/cli.ts --tool action_agent --tool-override action_agent_v0 --provider cerebras --model gpt-oss-120b

# Connect to running Browser Operator (bypasses bot detection, uses authenticated sessions)
npx tsx scripts/eval-runner/cli.ts --tool action_agent --remote-debugging-port 9222 --verbose

# Run with visible browser
npx tsx scripts/eval-runner/cli.ts --tool action_agent --no-headless
```

**Note:** The LLM judge defaults to OpenAI (`gpt-4o`) regardless of agent provider. Override with `--judge-provider` and `--judge-model`.

## Architecture

### DevTools Module Hierarchy

```
front_end/
├── core/           # Shared utilities, CDP backend integration
├── models/         # Business logic, data handling
├── panels/         # High-level panels (one per DevTools tab)
├── ui/components/  # Reusable UI components
└── entrypoints/    # Application entrypoints (devtools_app.ts)
```

Visibility rules: `core/` → `models/` → `panels/` → `entrypoints/` (enforced by GN build)

### AI Chat Panel (`front_end/panels/ai_chat/`)

```
ai_chat/
├── agent_framework/     # Agent execution engine
│   ├── AgentRunner.ts           # LLM loop, tool execution, handoffs
│   ├── ConfigurableAgentTool.ts # Agent definition via config objects
│   └── implementation/          # Concrete agent configs (ActionAgent, etc.)
├── LLM/                 # Provider integrations
│   ├── LLMClient.ts             # Client facade
│   ├── LLMProviderRegistry.ts   # Provider management
│   └── *Provider.ts             # OpenAI, Cerebras, Anthropic, Groq, etc.
├── cdp/                 # Chrome DevTools Protocol adapters
│   ├── CDPSessionAdapter.ts     # Abstract CDP interface
│   ├── DirectCDPAdapter.ts      # Direct CDP connection (eval runner)
│   └── SDKTargetAdapter.ts      # DevTools SDK integration
├── tools/               # Agent tools (~30 tools for browser actions)
├── dom/                 # Element resolution (shadow DOM, iframes)
├── common/              # Shared utilities (geometry, mouse, xpath)
├── core/                # Orchestration, LLMConfigurationManager
├── evaluation/          # Test case definitions
└── ui/                  # Chat panel UI components
```

### Key Concepts

**Agent Framework**
- `ConfigurableAgentTool`: Agents defined via config (name, prompt, tools, schema, handoffs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrowserOperator/browser-operator-core](https://github.com/BrowserOperator/browser-operator-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
