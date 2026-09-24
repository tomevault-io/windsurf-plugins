---
trigger: always_on
description: Guidance and repository standards for LLM coding agents working on `@rahularya01/pi-cursor`.
---

# AGENTS.md

Guidance and repository standards for LLM coding agents working on `@rahularya01/pi-cursor`.

---

## 1. Project Overview

`@rahularya01/pi-cursor` is a native Cursor provider extension for the **Pi Coding Agent**. It enables direct streaming communication with Cursor models (Claude, GPT-5.5, Composer, Grok, Gemini, Kimi) over HTTP/2 using Connect RPC and Protobuf binary framing.

### Core Stack

- **Runtime:** Node.js >= 22 (ESM native `"type": "module"`)
- **Peer Dependencies:** `@earendil-works/pi-ai` (>=0.80.0), `@earendil-works/pi-coding-agent` (>=0.80.0)
- **Protobuf / RPC:** `@bufbuild/protobuf` v2, `@bufbuild/buf` for schema compilation (`proto/agent.proto`)
- **Transport:** In-process HTTP/2 (`src/client/h2-session.ts` for streaming, `src/client/h2-unary.ts` for unary RPCs) — no subprocess
- **Toolchain:** Yarn 4 (package manager), Node.js (script runner)
- **Build & Test:** `tsup`, `typescript` (strict ESM, `tsc` still does the typechecking), `vitest`, `eslint`, `prettier`

---

## 2. Key Commands & Validation Workflows

Run `yarn check` before committing or completing any task. It runs the full validation suite:

```bash
# Run complete validation suite (Typecheck, Lint, Format check, Security check, Proto check, Tests)
yarn check
```

### Individual Development Commands

| Command               | Purpose                                                                                               |
| :-------------------- | :---------------------------------------------------------------------------------------------------- |
| `yarn typecheck`      | Run TypeScript compiler check without emitting (`tsc --noEmit`)                                       |
| `yarn test`           | Run unit tests (`vitest run`)                                                                         |
| `yarn test:watch`     | Run tests in interactive watch mode                                                                   |
| `yarn test:legacy`    | Run legacy standalone test scripts (routing, thinking levels, usage, context normalization, CLI auth) |
| `yarn lint`           | Run ESLint across `src/` and `tests/`                                                                 |
| `yarn lint:fix`       | Automatically fix ESLint errors                                                                       |
| `yarn format`         | Format codebase using Prettier                                                                        |
| `yarn format:check`   | Verify Prettier formatting compliance                                                                 |
| `yarn build`          | Bundle TypeScript sources with `tsup` into `dist/`                                                    |
| `yarn security-check` | Audit source for credential/token exposure leaks                                                      |
| `yarn proto:gen`      | Compile `proto/agent.proto` into `src/proto/agent_pb.ts` using `buf`                                  |
| `yarn proto:check`    | Verify `src/proto/agent_pb.ts` is up-to-date with `proto/agent.proto`                                 |
| `yarn proto:sync`     | Fetch and update protobuf descriptors from upstream                                                   |

### Smoke Testing

```bash
yarn smoke:auth    # Smoke test authentication resolution
yarn smoke:models  # Smoke test model discovery RPC
yarn smoke:stream  # Smoke test HTTP/2 streaming
yarn smoke:wire    # Smoke test low-level wire protocol frames
```

---

## 3. Architecture & Code Structure

```text
src/
├── index.ts                # Pi extension entrypoint & provider registration
├── usage.ts                # Usage quota dashboard & /cursor.usage handler
├── auth/                   # 4-tier credential resolution cascade
│   ├── index.ts            # Auth exports & manager
│   ├── cli-credentials.ts  # macOS Keychain, Cursor IDE SQLite DB, WSL path resolution
│   ├── oauth.ts            # PKCE browser login flow & token refresh
│   └── consent.ts          # System credentials privacy consent policy
├── client/                 # Low-level RPC & HTTP/2 transport
│   ├── h2-session.ts       # In-process HTTP/2 session for Connect RPC (streaming, bidirectional)
│   ├── h2-unary.ts         # In-process HTTP/2 client for unary RPCs (model discovery, usage)
│   ├── cursor-wire.ts      # Binary framing & Protobuf message encoders/decoders
│   └── bridge.ts           # BridgeHandle protocol framing + transport-agnostic entry point
├── stream/                 # Stream Simple adapter & session handling
│   ├── native-core.ts      # Core stream Simple interface for Pi AI
│   ├── root-prompt.ts      # Model-facing prompt messages (system prompt + replayed history)
│   ├── context-normalize.ts# Folds side-channel injections into system prompt
│   ├── thinking-filter.ts  # Reasoning effort mapping (off, minimal, low, medium, high, xhigh, max)
│   ├── model-discovery.ts  # GetUsableModels RPC & catalog mapping
│   ├── model-routing.ts    # Model alias & parameter routing
│   ├── drift.ts            # Wire protocol drift detection
│   └── recovery.ts         # Handled interaction queries & stream recovery

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rahularya01/pi-cursor](https://github.com/Rahularya01/pi-cursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
