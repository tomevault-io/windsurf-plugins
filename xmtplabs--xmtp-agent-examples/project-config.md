---
trigger: always_on
description: Build event‑driven, middleware‑powered messaging agents on the XMTP network. 🚀
---


# XMTP Agent SDK

Build event‑driven, middleware‑powered messaging agents on the XMTP network. 🚀

## Documentation

Full agent building guide: **[Build an XMTP Agent](https://docs.xmtp.org/agents/get-started/build-an-agent)**

This SDK is based on familiar Node.js patterns: you register event listeners, compose middleware, and extend behavior just like you would in frameworks such as [Express](https://expressjs.com/). This makes it easy to bring existing JavaScript and TypeScript skills into building conversational agents.

## Installation

Choose your package manager:

```bash
npm install @xmtp/agent-sdk
# or
pnpm add @xmtp/agent-sdk
# or
yarn add @xmtp/agent-sdk
```

## Quick Start

```ts
import { Agent, createUser, createSigner, getTestUrl } from "@xmtp/agent-sdk";

// 1. Create a local user + signer (you can plug in your own wallet signer)
const user = createUser();
const signer = createSigner(user);

// 2. Spin up the agent
const agent = await Agent.create(signer, {
  env: "dev", // or 'production'
  dbPath: null, // in-memory store; provide a path to persist
});

// 3. Respond to text messages
agent.on("text", async (ctx) => {
  await ctx.sendText("Hello from my XMTP Agent! 👋");
});

// 4. Log when we're ready
agent.on("start", (ctx) => {
  console.log(`We are online: ${getTestUrl(ctx.client)}`);
});

await agent.start();
```

## Environment Variables

The XMTP Agent SDK supports configuration through environment variables (`process.env`), making it easy to configure your agent without code changes. Set the following variables and call `Agent.createFromEnv()` to automatically load them:

**Available Variables:**

| Variable                 | Purpose                                                                                                         | Example                                 |
| ------------------------ | --------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| `XMTP_DB_DIRECTORY`      | [Database directory](https://docs.xmtp.org/agents/build-agents/local-database#understand-local-database-files)  | `XMTP_DB_DIRECTORY=my/database/dir`     |
| `XMTP_DB_ENCRYPTION_KEY` | [Database encryption key](https://docs.xmtp.org/agents/concepts/identity#keep-the-database-encryption-key-safe) | `XMTP_DB_ENCRYPTION_KEY=0xabcd...1234`  |
| `XMTP_ENV`               | [Network environment](https://docs.xmtp.org/chat-apps/core-messaging/create-a-client#xmtp-network-environments) | `XMTP_ENV=dev` or `XMTP_ENV=production` |
| `XMTP_WALLET_KEY`        | [Private key for Ethereum wallet](https://docs.xmtp.org/chat-apps/core-messaging/create-a-signer)               | `XMTP_WALLET_KEY=0x1234...abcd`         |

Using the environment variables, you can setup your agent in just a few lines of code:

```ts
// Load variables from .env file
process.loadEnvFile(".env");

// Create agent using environment variables
const agent = await Agent.createFromEnv();
```

Agents can also recognize the following environment variables:

| Variable                 | Purpose                                                                                            | Example                        |
| ------------------------ | -------------------------------------------------------------------------------------------------- | ------------------------------ |
| `XMTP_FORCE_DEBUG_LEVEL` | [Activate debugging logs](https://docs.xmtp.org/agents/deploy/debug-agents) at the specified level | `XMTP_FORCE_DEBUG_LEVEL=Debug` |

## Core Concepts

### 1. Event‑Driven Architecture

Subscribe only to what you need using Node’s `EventEmitter` interface. Events you can listen for:

**Message Events**

- `attachment` – an incoming [remote attachment message](https://docs.xmtp.org/chat-apps/content-types/attachments)
- `markdown` – an incoming [markdown-formatted](https://docs.xmtp.org/agents/content-types/markdown) text message
- `message` – all messages that are not having a [custom content type](https://docs.xmtp.org/agents/content-types/content-types#custom-content-types)
- `group-update` – an incoming [group update](https://docs.xmtp.org/agents/content-types/group-updates#listen-for-group-updates) (like name change, member update, etc.)
- `reaction` – an incoming [reaction message](https://docs.xmtp.org/agents/content-types/reactions)
- `read-receipt` – an incoming [read receipt](https://docs.xmtp.org/chat-apps/content-types/read-receipts) notification
- `reply` – an incoming [reply message](https://docs.xmtp.org/agents/content-types/replies)
- `text` – an incoming [text message](https://docs.xmtp.org/agents/content-types/content-types#text-content-type)
- `transaction-reference` – an incoming onchain [transaction reference](https://docs.xmtp.org/agents/content-types/transaction-refs#receive-a-transaction-reference)
- `wallet-send-calls` – an incoming wallet [transaction request](https://docs.xmtp.org/agents/content-types/transactions#create-a-transaction-request) (batch calls)
- `unknownMessage` – a message event that does not correspond to any of the pre-implemented event types

**Conversation Events**

- `conversation` – a new group or DM conversation
- `dm` – a new DM conversation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xmtplabs/xmtp-agent-examples](https://github.com/xmtplabs/xmtp-agent-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
