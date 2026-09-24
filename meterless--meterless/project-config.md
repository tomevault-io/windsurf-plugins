---
trigger: always_on
description: This document codifies a production implementation of Scout, the intent detection, risk guarding, and routing engine that runs BEFORE every meaningful action an agentic system takes. It is expressed in platform-neutral terms so any product can implement the same architecture as modular services.
---

# Scout Intent Agnostic Implementation Guide

## Purpose

This document codifies a production implementation of Scout, the intent detection, risk guarding, and routing engine that runs BEFORE every meaningful action an agentic system takes. It is expressed in platform-neutral terms so any product can implement the same architecture as modular services.

It covers:

- The five-stage decision pipeline (Sense, Interpret, Guard, Route, Recommend)
- The intent registry contract and versioning
- Two-stage scoring with the canonical confidence formula and bands
- The guard stack (injection, policy, scope, sensitivity, rate limits)
- The capability graph and tool-plan assembly
- Model profile routing
- The signed execution contract, the only artifact downstream engines accept
- Telemetry, the learning loop, and the eval gates that ship releases

Every number in this guide is traceable to a file in `docs/`. Where the docs leave a choice open, it is marked `(implementer's choice)`.

---

## 1) Core Design Principles

- **Nothing acts on a raw prompt.** Downstream engines (H-MEM, World Model, Markovian, Swarm) act only on signed execution contracts.
- **If an intent is not in the registry, it cannot be acted on.** The registry is the system's entire surface area; this is the property that prevents scope drift.
- **Deterministic floor, optional intelligence.** Stage-1 scoring is pure rules and always available; Stage-2 model rescoring is an upgrade, never a dependency.
- **Refuse at the boundary.** Scope violations are refused by the executor verifying the contract, not politely discouraged by a wrapper.
- **Clarify when unsure, but clarification is a cost.** The clarification rate has a hard ceiling in the eval gates; a system that always asks is failing, not being safe.
- **Every decision is replayable.** Contracts carry trace IDs; telemetry reconstructs any decision after the fact.

---

## 2) The Pipeline

Five stages, in order, per request:

```
Sense -> Interpret -> Guard -> Route -> Recommend -> ExecutionContract
```

1. **Sense** classifies intent from raw input. Output: top-k scored candidates, never a single label.
2. **Interpret** binds parameters and entity references from the prompt to the top intent, and decides whether clarification is needed.
3. **Guard** runs the policy stack. Output: risk level `low | medium | high | block` plus flags, redactions, and a reason on block.
4. **Route** walks the capability graph to assemble a tool plan.
5. **Recommend** picks the model profile and bundles everything into a signed contract.

The pipeline's public surface (see `docs/architecture.md` quickstart):

```ts
class Scout {
  constructor(options: {
    intentRegistry: string | IntentRegistry;   // file path or composed registry
    policyPack?: string;                        // named policy set, default "default"
    modelProfiles?: string;                     // profiles file
  });

  // Full pipeline: returns { intent, risk, executionContract, ... }
  decide(input: { prompt: string; user?: UserRef; surface: string; context?: unknown }): Promise<Decision>;

  // Sense stage only: returns { candidates: { intent, score }[] }
  classify(input: { prompt: string; surface: string }): Promise<{ candidates: IntentCandidate[] }>;

  // Re-planning with lineage (section 8)
  replan(input: { parent: string; reason: string; state?: unknown }): Promise<Decision>;
}
```

---

## 3) Intent Registry (the catalog)

Canonical record shape (`docs/intent-registry.md`):

```ts
type Intent = {
  id: string;                    // "deal.recover"; category.name grammar
  category: string;              // "deal", "email", "research"
  description: string;           // doubles as the classifier prompt
  parameters: ParameterSchema;   // JSON-schema-style, validated at Interpret
  capabilities: CapabilityRef[]; // abstract verbs, resolved by the graph
  riskClass: "low" | "medium" | "high";
  surfaces: string[];            // where this intent may be invoked
  roles: string[];               // who may invoke it
  examples: string[];            // classifier seeds AND eval fixtures; mandatory
  version: number;               // bump on ANY change; triggers re-eval
};
```

Rules:

- Examples are mandatory. An intent without examples cannot be classified or evaluated.
- Base `riskClass` sets the policy floor; risk is re-evaluated per request but never below the floor.
- `surfaces` and `roles` are enforced at Guard; a viewer-role invocation of an AE-only intent blocks before classification returns.
- The registry carries a `registryVersion`, stamped on every contract, so downstream engines detect drift.
- Loaders: single JSON file, directory of per-intent files, or composition of registries.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Meterless/Meterless](https://github.com/Meterless/Meterless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
