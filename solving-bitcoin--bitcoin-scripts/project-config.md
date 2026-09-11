---
trigger: always_on
description: This repository is an evidence-backed knowledge base and reproducible laboratory
---

# Research agent guide

This repository is an evidence-backed knowledge base and reproducible laboratory
for Bitcoin Script primitives. `knowledge/` describes the field; `src/`
contains local reference implementations that support some of its claims.

## Start here

1. Read [`knowledge/index.md`](knowledge/index.md).
2. Read [`knowledge/bitcoin-script-reference.md`](knowledge/bitcoin-script-reference.md)
   for the execution model, opcodes, script types, signature hashing, resource
   limits, standardness policy, and historical quirks of Bitcoin Script.
3. Read [`knowledge/cost-model.md`](knowledge/cost-model.md) before comparing
   measurements.
4. Query [`knowledge/catalog.json`](knowledge/catalog.json) with
   `python3 tools/kb.py list`, `show`, `best`, or `search`.
5. Follow links from a catalog record to its knowledge page, implementation
   README, source, tests, and references.

Do not assume that absence from the catalog means that a construction does not
exist. It means only that it has not yet been recorded here.

## Evidence language

Use exactly these evidence levels, from weakest to strongest:

- `reported`: a primary source makes the claim; it has not been reproduced here.
- `inspected`: maintainers inspected the construction or source, without a local
  executable reproduction.
- `locally-reproduced`: a deterministic local test or metric reproduces it.
- `differentially-validated`: a local result was compared with an independent
  implementation or Bitcoin Core as appropriate.

Use exactly these deployment classes:

- `consensus-validated`: exercised under the applicable Bitcoin consensus rules.
- `policy-validated`: additionally accepted by the documented relay policy.
- `consensus-incompatible`: known to exceed or violate a consensus rule.
- `research-unlimited`: evaluated with one or more consensus checks disabled.
- `unclassified`: not yet established.

Never describe `research-unlimited` success as consensus validity or
deployability. The helper at `src/support/execution.rs` currently executes all
scripts in a tapscript context; `execute_raw_script_with_inputs` also disables
the stack limit. Record that distinction in every result that uses it.

## Research protocol

- State the question and comparison objective before optimizing.
- Prefer primary sources: specifications, papers, upstream repositories, and
  exact Bitcoin Core revisions.
- Record source URLs and immutable commits or document versions.
- Separate measured facts from inference. Label estimates explicitly.
- Use deterministic inputs and RNG seeds. Record configuration parameters.
- Report locking-script bytes, serialized witness bytes, stack peak, executed
  opcodes or validation budget when available, and the execution class.
- For every primitive that requires witness hints, report the exact number of
  hint stack items per invocation and for every measured repeated or batched
  configuration. Distinguish incremental hint items from the complete
  witness/data item count, state whether all hints coexist at script entry or
  describe the fragment boundary used, include them in the measured combined
  main-plus-alt-stack peak, and explain composition under Bitcoin's 1,000-item
  limit. Serialized hint bytes and total stack peak do not substitute for the
  explicit hint-item count.
- Compare like with like. Setup, cleanup, input pushes, output checks, and
  witness serialization must either be included on both sides or excluded on
  both sides.
- Record failed, dominated, and non-composable approaches in
  `knowledge/negative-results/`; they are part of the state of the art.
- Add unresolved questions to `knowledge/open-problems.md` with a falsifiable
  acceptance criterion.

## Adding or changing a primitive

1. Add or update its implementation README using
   `docs/primitive-readme-template.md`.
2. Add executable correctness tests, including malformed inputs and boundary
   cases.
3. Add metric markers to `tests/primitive_metrics.rs` when a representative
   configuration is stable.
4. Add or update the knowledge page and `knowledge/catalog.json`.
5. Update every affected comparison, technique, protocol, reference, negative
   result, and open problem page.
6. Run:

   ```sh
   python3 tools/kb.py validate
   cargo test --locked -- --skip fields::
   ```

Skip field-arithmetic tests by default, including in future work, unless the
user explicitly requests them. Keep the `--skip fields::` filter on broad test
runs and prefer targeted tests for the primitive being changed.

Use `UPDATE_PRIMITIVE_METRICS=1 cargo test --locked --test primitive_metrics`
only for an intentional metric change. Do not silently refresh measurements.

## Script compilation and optimization

- Compile repository scripts through
  `support::script::ScriptCompilation::compile_with_policy()`. Do not call
  upstream `Script::compile()`, `compile_optimized()`, or
  `compile_with_options()` directly outside the centralized policy.
- The policy applies `CompileOptions::ALL` to generated scripts whose
  unoptimized serialization is at most 32 KiB. Larger scripts use
  `CompileOptions::NONE` because the optimizer's fixpoint passes are too slow
  at that scale. Keep execution, byte metrics, opcode metrics, Tapleaf hashes,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solving-bitcoin/bitcoin-scripts](https://github.com/solving-bitcoin/bitcoin-scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
