---
trigger: always_on
description: |
---


# Walrus Memory — Portable Agent Memory

Walrus Memory enables AI agents to operate reliably across apps and sessions, without losing context. It stores memories on Walrus (decentralized storage), encrypts them with SEAL, enforces ownership onchain via Sui smart contracts, and retrieves them with semantic (vector) search. Memory is portable by design — not tied to a single runtime or provider — and scoped by `owner + namespace` for isolation and coordination.

---

## When to Use

Use Walrus Memory when your app or agent needs:

- **Portable memory** — persists outside prompts and context windows, moves across agents, apps, and workflows
- **Full owner control** — programmable permissions and explicit ownership define how memory is shared and accessed
- **Agent coordination** — shared memory spaces help agents coordinate across long-running and multi-step workflows
- **Semantic recall** — retrieve memories by meaning, not just keywords
- **Verifiable integrity** — memory integrity can be independently verified without centralized trust
- **Cross-app memory** — not tied to a single runtime or provider, share memory between apps via delegate keys

---

## When NOT to Use

- Temporary conversation context that only matters in the current session
- Large file storage (Walrus Memory is optimized for text memories)
- Use cases that don't need encryption or decentralization

---

## Installation

```bash
# Install the SDK
pnpm add @mysten-incubation/memwal

# Optional: for Vercel AI SDK integration
pnpm add ai zod

# Optional: for manual client (client-side SEAL encryption)
pnpm add @mysten/sui @mysten/seal @mysten/walrus
```

---

## Quick Start

### 1. Get Your Credentials

You need a **delegate key** (Ed25519 private key) and **account ID** (Walrus Memory account object ID on Sui).

Generate them at:
- Production: https://memory.walrus.xyz
- Staging: https://staging.memory.walrus.xyz

### 2. Initialize the SDK

```ts
import { MemWal } from "@mysten-incubation/memwal";

const memwal = MemWal.create({
  key: process.env.MEMWAL_PRIVATE_KEY!,
  accountId: process.env.MEMWAL_ACCOUNT_ID!,
  serverUrl: process.env.MEMWAL_SERVER_URL ?? "https://relayer.memory.walrus.xyz",
  namespace: "my-app",
});
```

### 3. Store and Recall Memories

```ts
// Store one already-distilled fact and wait until it is indexed.
await memwal.rememberAndWait(
  "User prefers dark mode and works in TypeScript.",
  undefined,
  { timeoutMs: 30_000 },
);

// Recall by meaning
const result = await memwal.recall({ query: "What are the user's preferences?" });
console.log(result.results);

// Extract facts from free-form text and wait until all accepted facts are indexed.
const analyzed = await memwal.analyzeAndWait(
  "I live in Hanoi and prefer dark mode.",
  undefined,
  { timeoutMs: 30_000 },
);
console.log(analyzed.facts.map((fact) => fact.text));

// Check relayer health
await memwal.health();
```

Use `*AndWait` when a workshop UI saves and then immediately recalls in the
same flow. Indexing can lag by a few seconds, so `remember()` / `analyze()`
may return before recall can find the new memory. Manual polling is still
available for advanced async UIs:

```ts
const accepted = await memwal.remember("User likes Sui.");
const stored = await memwal.waitForRememberJob(accepted.job_id, {
  pollIntervalMs: 750,
  timeoutMs: 30_000,
});
```

---

## SDK Entry Points

| Entry Point | Import | Description |
|---|---|---|
| `MemWal` | `@mysten-incubation/memwal` | **Default.** Relayer handles embedding, SEAL encryption, Walrus upload, vector search |
| `MemWalManual` | `@mysten-incubation/memwal/manual` | Manual flow — client handles embedding and SEAL encryption |
| `withMemWal` | `@mysten-incubation/memwal/ai` | Vercel AI SDK middleware — auto recall + save around AI conversations |
| Account utils | `@mysten-incubation/memwal/account` | Account creation, delegate key management |

---

## API Surface

### Walrus Memory Methods

| Method | Description | Returns |
|---|---|---|
| `remember(text, namespace?)` | Accept one memory job immediately | `{ job_id, status }` |
| `rememberAndWait(text, namespace?, opts?)` | Store one memory and wait for completion | `{ id, job_id, blob_id, owner, namespace }` |
| `recall({ query, limit?, namespace?, maxDistance? })` *(preferred)* or `recall(query, limit?, namespace?)` | Semantic search for memories | `{ results: [{ blob_id, text, distance }], total }` |
| `analyze(text, namespace?)` | Extract facts and accept one memory job per fact | `{ job_ids, facts, fact_count, status, owner }` |
| `analyzeAndWait(text, namespace?, opts?)` | Extract facts and wait for all fact jobs to complete | `{ results, facts, total, succeeded, failed, owner }` |
| `restore(namespace, limit?)` | Rebuild missing index entries from Walrus | `{ restored, skipped, total, namespace, owner }` |
| `health()` | Check relayer health | `{ status, version }` |
| `getPublicKeyHex()` | Get hex-encoded public key | `string` |

### Lower-Level Methods

| Method | Description |
|---|---|
| `rememberManual({ blobId, vector, namespace? })` | Register pre-uploaded blob with pre-computed vector |
| `recallManual({ vector, limit?, namespace? })` | Search with pre-computed vector (returns blob IDs only) |
| `embed(text)` | Generate embedding vector (no storage) |

### All Response Shapes

```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MystenLabs/MemWal](https://github.com/MystenLabs/MemWal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
