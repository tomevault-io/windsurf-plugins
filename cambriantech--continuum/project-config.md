---
trigger: always_on
description: **Lesson from the qwen3-coder-30b-a3b-compacted-19b-256k v1 publish (alloy hash `aa61c4bdf463847c`):** authoring per-artifact alloy files by hand is anti-architectural. Every successful forge requires the same set of fields — `name`, `userSummary`, `description`, `tags`, `source`, `stages[]` with notes, `results.benchmarks[]` with `samplesPath` + `baseSamplesPath`, `priorMetricBaselines[]`, `limitations[]`, `methodologyPaperUrl` — and we wrote them by hand into a `.alloy.json` for the v1 publish
---

# CLAUDE - ESSENTIAL DEVELOPMENT GUIDE

## 🏭 FORGE TEMPLATE ARCHITECTURE (the next sprint)

**Lesson from the qwen3-coder-30b-a3b-compacted-19b-256k v1 publish (alloy hash `aa61c4bdf463847c`):** authoring per-artifact alloy files by hand is anti-architectural. Every successful forge requires the same set of fields — `name`, `userSummary`, `description`, `tags`, `source`, `stages[]` with notes, `results.benchmarks[]` with `samplesPath` + `baseSamplesPath`, `priorMetricBaselines[]`, `limitations[]`, `methodologyPaperUrl` — and we wrote them by hand into a `.alloy.json` for the v1 publish. That's where they need to STOP being manually authored.

**The rule going forward:** all the fields a forge run needs to populate an alloy MUST live as Continuum entity data inside a `ForgeRecipe` entity (or equivalent), keyed by the artifact name. The forge pipeline takes the recipe entity as input, runs the prune / quant / eval stages, and emits the populated alloy as OUTPUT. The forge never consumes a hand-authored alloy; the foundry generates it.

**Recipe entity must carry, at minimum:**
- `name`, `description`, `userSummary`, `tags`, `methodologyPaperUrl`, `limitations[]` — **all the prose fields the model card renders**
- `source.baseModel` — what to forge from
- `stages[]` — the recipe steps with their per-stage `notes` (the methodology blockquotes)
- `calibrationCorpus` — pointer to the held-out corpus the importance profile and (eventual) compensation LoRA train against
- `quantTiers[]` — which GGUF tiers to ship
- `evaluationBenchmarks[]` — what to score against
- `priorMetricBaselines[]` — methodology negative-baselines to preserve in the publish for §4.1.3.4 falsifiability
- `hardware` — target VRAM tiers + device ladder

**Forge pipeline output entity (`ForgeArtifact`):**
- Inherits everything from the recipe
- Adds `results.benchmarks[]` (filled in from eval runs), `forgedParamsB`, `activeParamsB`, `hardwareVerified[]`, the alloy hash, the verify URL, the published HF repo URL
- This is what `publish_model.py` reads, NOT a hand-authored alloy file

**Why this matters for the second killer (and every killer after):** the qwen3-coder publish required ~6 manual edits to fix paper-speak hallucination, naming conventions, tag overflow, headline subtitle bugs, and benchmark renderer fallthrough. Every one of those was a manual touch on hand-authored prose. If the recipe entity had been the source of truth and the alloy had been the projection, none of those manual touches would have been needed. The architectural target is "author the recipe once in Continuum, run the foundry, ship the artifact, the card writes itself from the recipe + the eval results."

**Status:** the entity schema and the foundry executor that consumes it are NOT yet built. v1 of qwen3-coder shipped via hand-authored alloy. The next sprint (post-vision-support) is the foundry template architecture. Reference: `forge-alloy/python/forge_alloy/types.py` has the alloy types; the recipe-as-entity layer needs to wrap them with a Continuum entity that lives in the data layer and is editable through the standard `Commands.execute('data/...')` primitives.

## ⚡ THE TWO UNIVERSAL PRIMITIVES (E = mc²)

**Everything in this system is built on TWO primitives:**

### 1. `Commands.execute<T, U>(name, params)` - Request/Response
```typescript
import { Commands } from 'system/core/shared/Commands';

// Type-safe! params and result types inferred from command name
const users = await Commands.execute('data/list', { collection: 'users' });
const screenshot = await Commands.execute('screenshot', { querySelector: 'body' });
```

### 2. `Events.subscribe()|emit()` - Publish/Subscribe
```typescript
import { Events } from 'system/core/shared/Events';

Events.subscribe('data:users:created', (user) => { /* handle */ });
Events.emit('data:users:created', newUser);
```

**Key Properties:**
- Type-safe with full TypeScript inference
- Universal (works everywhere: browser, server, CLI, tests)
- Transparent (local = direct, remote = WebSocket)
- Auto-injected context and sessionId

**See detailed documentation:** [docs/UNIVERSAL-PRIMITIVES.md](docs/UNIVERSAL-PRIMITIVES.md)

---

## 🧬 THE COMPRESSION PRINCIPLE (Fundamental Law)

**One logical decision, one place. No exceptions.**

This applies to BOTH program memory and data memory:

| Type | Uncompressed (BAD) | Compressed (GOOD) |
|------|-------------------|-------------------|
| **Logic** | `findRoom()` in 5 files | `resolveRoomIdentifier()` in RoutingService |
| **Data** | `UUID_PATTERN` in 3 files | `isUUID()` exported from one place |
| **Constants** | Magic strings everywhere | `ROOM_UNIQUE_IDS.GENERAL` |

**The ideal codebase is maximally compressed:**
```
Root Primitives (minimal)    ←  Commands.execute(), Events.emit()
       ↓

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CambrianTech/continuum](https://github.com/CambrianTech/continuum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
