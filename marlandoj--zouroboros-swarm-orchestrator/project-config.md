---
trigger: always_on
description: **Incident:** Sprint 2 Deferred Remediation campaign (March 4, 2026) — 5 failure categories, ~50% efficiency.
---

# Swarm Orchestrator - Agent Memory

## March 2026 Failure Remediation (v4.0.0 → v4.1.0)

**Incident:** Sprint 2 Deferred Remediation campaign (March 4, 2026) — 5 failure categories, ~50% efficiency.

### Phase 1: Quick Wins (Implemented)
| Fix | Addresses | What it does |
|-----|-----------|-------------|
| **R1: Campaign Locking** | F1, F3 | File lock at `/dev/shm/{swarmId}.lock` prevents duplicate concurrent runs. 30-min stale threshold. |
| **R2: Per-Task Timeout** | F2 | `timeoutSeconds` field on each task. Tiers: quickEdit=120s, analysis=300s, heavyIO=900s, buildTest=600s. |
| **R6: Stderr Capture** | F6 | Bridge script falls back to stderr when stdout is empty. Eliminates zero-length output successes. |

### Phase 2: Reliability (Implemented)
| Fix | Addresses | What it does |
|-----|-----------|-------------|
| **R3: Post-Mutation Verification** | F5 | `expectedMutations` array on tasks. After task success, verifies files contain expected strings. Fails task if not. |
| **R4: Startup Preflight** | F1 | `preflight()` validates task structure, duplicate IDs, executor bridges, API creds, DAG deps, cycles, memory. Fails fast with diagnostics. |
| **R7: Prompt Reinforcement** | F5 | Appends "IMPORTANT: This task requires ACTUAL FILE CHANGES" + file list to prompts when `expectedMutations` is set. |

### Phase 3: UX (Implemented)
| Fix | Addresses | What it does |
|-----|-----------|-------------|
| **R5: Async Completion Notification** | F4 | Writes `/dev/shm/{swarmId}-complete.json` on finish (always). Optionally sends SMS or email via `--notify sms` or `--notify email`. User is informed regardless of chat session state. |

### Campaign JSON Example (post-remediation)
```json
{
  "id": "s2-compression-middleware",
  "persona": "claude-code",
  "task": "Add compression middleware to server/index.ts",
  "priority": "high",
  "timeoutSeconds": 120,
  "expectedMutations": [
    { "file": "/home/workspace/verdant-goods-store/server/index.ts", "contains": "compression()" },
    { "file": "/home/workspace/verdant-goods-store/package.json", "contains": "\"compression\"" }
  ]
}
```

## Lessons Learned from Production Failures

### February 2026 Incident Analysis

**Context:** Attempted to use swarm orchestrator for a multi-page website review with 5 agents simultaneously.

**What Failed:**
1. All parallel API calls timed out (120s insufficient)
2. No retry mechanism caused complete failure
3. Rate limiting from concurrent requests
4. Context window pressure with 256k tokens × 5 agents

**What Worked:**
1. Direct sequential API calls succeeded
2. Smaller, focused tasks completed successfully
3. Python subprocess approach was reliable
4. Bun/TypeScript orchestrator script logic was sound

**Root Causes Identified:**
- API calls had implicit rate limits per session (resolved: now local-only)
- Complex analysis requires >120s timeout
- No backoff strategy caused thundering herd
- Missing circuit breaker allowed cascading failures

## Memory Integration (v4.5.0)

As of v4.5.0, the orchestrator integrates with zo-memory-system for learning and adaptive routing:

| Feature | Implementation | Benefit |
|---------|---------------|---------|
| Auto-Episodes | `createSwarmEpisode()` at end of `run()` | Every swarm run becomes a queryable event |
| 6-Signal Routing | `compositeRoute()` adds procedure + temporal | Routes learn from past success/failure |
| Cognitive Profiles | Extended `executor-history.json` | Episode IDs, failure patterns, entity affinities |
| Error Classification | Auto-classify: timeout, mutation_failed, file_not_found | Failure patterns inform routing avoidance |
| Entity Affinities | Exponential moving average per entity per executor | Tracks which executors excel at which domains |

### Composite Router Signals (v4.5)

```
composite = (w.capability * capScore)     # Task-capability matching
           + (w.health * hlth)            # Circuit breaker health
           + (w.complexityFit * cfit)     # Complexity tier affinity
           + (w.history * hist)           # Historical success rate
           + (0.10 * (procScore - 0.5))   # Procedure success bonus (±0.05)
           + (0.05 * (tempScore - 0.5))   # Recent episodic bonus (±0.025)
```

### Querying Swarm History

```bash
# Recent swarm episodes
bun ~/Skills/zo-memory-system/scripts/memory.ts episodes --entity "swarm.ffb" --since "7 days ago"

# Executor cognitive profile
bun ~/Skills/zo-memory-system/scripts/memory.ts profile --executor gemini

# Procedure evolution
bun ~/Skills/zo-memory-system/scripts/memory.ts procedures --evolve "site-review"
```

## Architecture (v4.6.0 — Local Executors + OmniRoute Failover)

Primary path: all tasks execute via local executor bridges (claude-code, hermes, gemini, codex).
When all local executors exhaust retries, OmniRoute provides API-level failover through a
priority combo that chains multiple providers (paid → free tiers).

```
v4.4 (local only)              v4.6 (local + OmniRoute failover)
─────────────────              ─────────────────────────────────
1 execution path               2 execution paths
  - Local executor only          - Local executor (primary)
                                 - OmniRoute API failover (last resort)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marlandoj/zouroboros-swarm-orchestrator](https://github.com/marlandoj/zouroboros-swarm-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
