---
trigger: always_on
description: pnpm workspace. Never use npm.
---

# D&D 5e PHB — project notes

## Package manager

pnpm workspace. Never use npm.

## No external consumers (CRITICAL)

This is a greenfield project with no users, no published API, no downstream dependencies. **We own the entire stack — Quint spec, XState machine, TS features, MBT bridge, React UI.** Any layer can change to serve any other layer.

Do not treat internal boundaries as walls. When a lower layer needs a change to support a higher layer, change it — don't work around it with adapters, registries, or parallel data structures. The cost of changing `creature.qnt` and updating the MBT bridge is always less than the cost of maintaining a workaround that keeps layers "separate." Design for the system, not for the boundary.

Concretely: adding a field to `ActiveEffect`, renaming a type in the spec, restructuring `DndContext` — all fine. Update the bridge, run MBT, move on.

## No redundant state (CRITICAL)

Never duplicate data that already exists in another layer. Before adding a field to any type, **search for existing fields that carry the same data** across the entire codebase. If found: reference, project, or re-export — don't copy. The cost of threading existing data through a layer boundary is always less than the cost of maintaining two copies that can diverge.

This applies across all layers — Quint spec, XState context, TS types, React state. If a plan proposes adding fields, verify they don't already exist somewhere before implementing.

## Provenance and modeling discipline (CRITICAL)

When modeling content sources, distinguish three different concepts:

- **provenance** — the canonical rules source the shipped data claims to come from;
- **structured input** — machine-readable data used to help import, normalize, or cross-check;
- **runtime projection** — derived execution-facing facts used by the engine.

Do not collapse these into one field or one type.

For monster data in this repo:

- SRD is provenance for shipped SRD monsters.
- 5e-tools is valuable structured data and normalization inspiration, but it is **never** provenance.
- If a collection is supposed to be "the SRD catalog", model it so mixed-provenance or mixed-license states are unrepresentable at the collection boundary.

General design rule:

- **Make invalid states irrepresentable.** This is mandatory before proposing or implementing any data shape. If a proposed type can represent contradictory provenance, contradictory ownership, mismatched derived facts, support-status markers with no type/runtime consequence, or any field combination that is impossible in the code or rules domain, redesign the type before presenting it.
- Do not store derivable facts beside their source facts unless the duplication is executable at the boundary that matters. Prefer deriving labels, abbreviations, display names, option ids, and projections from one canonical value or table, so mismatches cannot be represented.
- Do not add status enums or metadata labels that neither affect the type system nor runtime behavior unless there is a specific, durable reason the repo needs them.

## Domain-language reflex (extends SRD-parity rules above)

When a union type feels off, the signal to refactor is **domain conflation**, not *just* "is this type-safe?" Type safety matters a lot; it is necessary but not sufficient. A mixed union whose name fits only half its members already lies about the world even if every variant typechecks. Justify splits/renames in domain terms first (e.g., "rest-triggered" vs "calendar-time-triggered" are distinct SRD triggers), and let type safety follow.

## Connascence discipline (CRITICAL)

When changing code, actively look for connascence: code facts that must change together for correctness.

This is mandatory before finalizing any change, especially when adding or preserving:

- string or numeric literals;
- tuple/array index assumptions;
- phase/order/count assumptions;
- support gates and downstream narrowed-type usage;
- duplicated validation/projection/execution logic;
- caller protocols that require a sequence of operations.

Required check:

1. Ask: "What must change together if this line changes?"
2. Classify the coupling:
   - name/type: usually acceptable if explicit and tool-visible;
   - meaning/value/position/algorithm: risky if duplicated or distant;
   - execution/timing/identity: high-risk unless type-enforced or tightly localized.
3. Evaluate locality and degree:
   - strong connascence is acceptable only when nearby and obvious;
   - distant or repeated connascence must be refactored.
4. Prefer refactors that weaken or localize connascence:
   - replace magic values with named constants or domain types;
   - replace positional conventions with named fields;
   - replace duplicated algorithms with one shared implementation;
   - replace caller sequencing requirements with one operation or state-typed APIs;
   - make support-gate facts flow through narrowed types instead of downstream memory.
5. If strong connascence must remain, colocate the coupled facts in one helper/module and name the helper after the domain invariant.

Do not rely on comments alone when code can encode the relationship.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dearlordylord/5e-quint](https://github.com/dearlordylord/5e-quint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
