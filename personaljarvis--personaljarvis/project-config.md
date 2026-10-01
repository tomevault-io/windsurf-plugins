---
trigger: always_on
description: Recursive, decentralized learning + memory layer for AI agents. Implements the **Agentic Context Engine (ACE)** loop with diagnostic guardrails (AgentDoG + LATS) and CRDT-based peer-to-peer sync between instances. Design source: `../Self-learning/KI-Fehlervermeidung und Wissensaustausch (1).md` (treated as research survey, not literal spec — engineering decisions live in `docs/adr/`).
---

# Skillbook

Recursive, decentralized learning + memory layer for AI agents. Implements the **Agentic Context Engine (ACE)** loop with diagnostic guardrails (AgentDoG + LATS) and CRDT-based peer-to-peer sync between instances. Design source: `../Self-learning/KI-Fehlervermeidung und Wissensaustausch (1).md` (treated as research survey, not literal spec — engineering decisions live in `docs/adr/`).

## Hard Constraints

1. **All code lives under `skillbook/`.** Anything outside this directory is read-only — including `wiki/obsidian-vault/`, which is fully off-limits.
2. **Memory-layer storage is schema-isolated.** The skillbook database file and all tables/keys are namespaced and never reuse a user-data schema.
3. **No production stubs, TODOs, NotImplementedError, or "fix later" placeholders** in any path the capstone test exercises. Test mocks are allowed and expected.

## Module Boundaries

```
┌──────────────────────────────────────────────────────────────────┐
│ tests/test_capstone.py     (end-to-end ACE loop oracle)          │
└──────────────────────────────────────────────────────────────────┘
                                 │
   ┌─────────────────────────────┼─────────────────────────────┐
   ▼                             ▼                             ▼
┌──────────────┐         ┌──────────────────┐         ┌──────────────────┐
│ p2p_sync     │         │ ace_core         │         │ symcon_bridge    │
│ (CRDT delta) │◄────────│ Generator        │◄────────│ MQTT + JSON-RPC  │
└──────────────┘         │ Reflector(REPL)  │         └──────────────────┘
                         │ Curator          │
                         └────────┬─────────┘
                                  │
                  ┌───────────────┴──────────────┐
                  ▼                              ▼
           ┌──────────────┐              ┌──────────────────┐
           │ guardrails   │              │ memory_layer     │
           │ AgentDoG     │              │ Temporal KG +    │
           │ LATS         │              │ Skillbook store  │
           └──────────────┘              └──────────────────┘
```

**Dependency direction (strict, enforced by package layout):**

- `memory_layer` depends on stdlib + numpy only.
- `guardrails` depends on `memory_layer` (reads rules) and stdlib.
- `ace_core` depends on `memory_layer` + `guardrails` + stdlib.
- `symcon_bridge` depends on stdlib (and optionally `aiomqtt` for real broker use).
- `p2p_sync` depends on `memory_layer` (reads/writes deltas) + stdlib.
- Tests live in `tests/`; production modules never import from `tests/`.
- Circular imports are a build failure.

## Interface Types

Public surface uses `Protocol` (PEP 544) for swap-ability and `pydantic.BaseModel` for wire-format data. Internal helpers use `@dataclass(slots=True, frozen=True)`. No abstract base classes (`ABC`) unless inheritance is required.

Key interfaces:

- `MemoryStore` — `Protocol`, async; `put_fact`, `query_facts`, `put_rule`, `query_rules`, `delete_rule`.
- `Embedder` — `Protocol`, sync; `embed(text: str) -> np.ndarray`.
- `Actor` — `Protocol`, async; `call(name, params, timeout) -> ActorResult`. Mocked in tests.
- `LLM` — `Protocol`, async; `complete(prompt) -> str`. Deterministic mock when `ANTHROPIC_API_KEY` is unset.
- `Transport` — `Protocol`, async; `gossip(peer_id, payload)`, `subscribe(handler)`. Used by `p2p_sync`.

Concrete data models live in `skillbook.{module}.models` modules (pydantic v2).

## Tech-Stack Choices (one ADR per choice)

| Concern | Choice | ADR |
|---|---|---|
| Python runtime | 3.11+ (3.12 preferred, uv-managed) | ADR-0001 |
| Dependency manager | uv | ADR-0001 |
| Repo layout | `src/skillbook/{module}/`, `tests/`, `data/` | ADR-0001 |
| Persistent storage | SQLite under `data/`, file-per-instance | ADR-0002 |
| Embeddings | sentence-transformers if installed + cached, deterministic hash fallback | ADR-0003 |
| LLM provider | Anthropic SDK if `ANTHROPIC_API_KEY` present, deterministic mock otherwise | ADR-0004 |
| MQTT stack | `aiomqtt` for production; tests use injected mock bridge | ADR-0005 |
| P2P transport | OR-Set CRDT over async Transport protocol; default in-process queue, asyncio-TCP available | ADR-0006 |
| Reflector sandbox | Subprocess REPL with restricted globals + result-via-stdout JSON | ADR-0007 |
| Guardrails taxonomy | Enum-driven AgentDoG (Source/FailureMode/Consequence) | ADR-0008 |
| Survey deviations | See ADR-0009 | ADR-0009 |

## Test Pyramid

- **Unit tests** (`tests/unit/{module}/`): one behavior per file, mock at module boundary.
- **Integration tests** (`tests/integration/`): two-module flows (e.g. Reflector → Curator → memory).
- **Capstone** (`tests/test_capstone.py`): full 7-step scenario from the goal.
- Every test is deterministic under `--seeds=N`. A `seed` fixture parametrizes over `range(N)`. Tests that consume `seed` run N times; tests that don't run once. Failure under any seed = failure.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PersonalJarvis/PersonalJarvis](https://github.com/PersonalJarvis/PersonalJarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
