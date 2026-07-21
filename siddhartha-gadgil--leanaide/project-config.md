---
trigger: always_on
description: This file documents the LLM-backed agents and agent-like LLM components used by
---

# mathdoc_agent Agents

This file documents the LLM-backed agents and agent-like LLM components used by
`mathdoc_agent`. The canonical definitions live in
`mathdoc_agent/mathagents/definitions.py`; prompt text lives in
`mathdoc_agent/mathagents/prompts.py`; schemas live in
`mathdoc_agent/models/refinement_specs.py` and
`mathdoc_agent/models/payloads.py`.

## Runtime Contract

All OpenAI Agents SDK agents are created by `mathdoc_agent.mathagents.definitions._agent`.
The default model is read from:

```text
MATHDOC_AGENT_MODEL
```

If the variable is unset, the package currently defaults to `gpt-5.5`.

Agents are executed through `mathdoc_agent.mathagents.runner.run_agent_typed`.
The runner accepts:

- OpenAI Agents SDK `Agent` objects;
- fake or test callables;
- objects exposing `.run(payload)`;
- already structured Pydantic-like outputs.

The runner converts the payload to JSON-compatible data, calls the agent, coerces
the result to the requested Pydantic output schema, logs start/completion/failure
events to stderr, and applies theorem-name enrichment to any
`deduced_from_theorem` objects in the returned model.

Agent calls are bounded by:

```text
MATHDOC_AGENT_AGENT_TIMEOUT_SECONDS
```

If unset, the timeout is 600 seconds. Set it to `0` or a negative value to
disable the timeout.

## Pipeline Placement

The default JSON generation path in `mathdoc_agent/pipeline.py` is:

1. Parse source text into a `MathDocument` using `document_parser_agent`.
2. Refine attached proofs using the proof classifier and proof refiners.
3. Resolve proof kinds that do not have direct Lean codegen handlers.
4. Record Mathlib definition reuse using local similarity search, an LLM exact
   match check, and LeanSearch fallback.
5. Export the document to PaperStructure JSON.
6. Rewrite `deduced_from_claim` dependencies into explicit Lean-friendly proof
   steps.
7. Audit public `claim` fields.
8. Repair informal local notation.
9. Audit risky proof-step assertions.
10. Repair proof-sanity issues.
11. Re-run residual `deduced_from_claim` materialization and informal notation
    repair.
12. Run deterministic Lean-facing JSON finalization.

When the caller supplies custom document or proof registries, the pipeline does
not automatically use the default post-processing agents unless they are also
passed explicitly.

## Shared Schema Fragments

Several agents share these nested schema fragments.

### ChildProofSpec

Used when an agent creates child proof nodes.

```json
{
  "id_suffix": "base",
  "kind": "simple",
  "text": "Proof text for this child.",
  "goal": "optional child goal",
  "hypotheses": ["optional local hypotheses"],
  "notes": ["optional notes"]
}
```

`kind` is usually one of the `ProofKind` values in
`mathdoc_agent/models/base.py`.

### LogicalProofStepData

Used for linear proof steps and inserted local assertions.

```json
{
  "type": "assert_statement",
  "name": "optional_local_name",
  "claim": "a proposition-shaped mathematical claim",
  "proof_method": "short justification",
  "lean_term": "optional Lean term proving this step",
  "source_claim": "optional general claim being instantiated",
  "arguments": ["optional local values or hypotheses"],
  "assumption": "for assume_statement steps",
  "variable_name": "for let/fix-style steps",
  "variable_type": "type of the introduced variable",
  "statement": "alternate statement field",
  "deduced_from_claim": ["local/contextual claims used"],
  "deduced_from_theorem": [
    {
      "claim": "general theorem statement",
      "name": "optional human theorem name",
      "description": "how it is used",
      "lean_name": "optional exact Lean/Mathlib declaration name",
      "lean_term": "optional instantiated theorem term"
    }
  ]
}
```

The pipeline should not emit final JSON with `results_used`; proof dependencies
should be represented as `deduced_from_theorem` or rewritten from
`deduced_from_claim` into explicit `have`/lemma steps.

### ParameterData

Used by structure and inductive declarations and by instance parameters.

```json
{
  "name": "G",
  "type": "Type u",
  "binder": "default"
}
```

`binder` may be `default`, `implicit`, or `typeclass`. If omitted, it is treated
as `default`.

### DeducedFromTheoremData

Used by proof agents to name external theorems.

```json
{
  "claim": "the general theorem statement",
  "name": "optional common name",
  "description": "optional usage note",
  "lean_name": "optional exact Lean declaration name",
  "lean_term": "optional Lean term for the instance used"
}
```

`lean_name` is enriched after agent execution when LeanSearch is enabled. The
agent should not invent Lean names. If a theorem is applied to particular local
variables or hypotheses, `lean_term` should contain the term for that instance.

## Main Document Agent

### `document_parser_agent`

Definition:

```python
document_parser_agent = _agent(
    "Document parser",
    prompts.DOCUMENT_PARSER_INSTRUCTIONS,
    DocumentRefinementSpec,
)
```

Used by:

- `UnknownDocumentHandler` in `mathdoc_agent/handlers/document_handlers.py`.
- The default document handler registry for `DocumentKind.unknown`.

Expected input payload:

```json
{
  "node": {
    "id": "doc.root",
    "kind": "unknown",
    "status": "raw",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [siddhartha-gadgil/LeanAide](https://github.com/siddhartha-gadgil/LeanAide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
