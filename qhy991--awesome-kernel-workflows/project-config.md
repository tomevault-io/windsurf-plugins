---
trigger: always_on
description: Instructions for any AI agent (Claude Code, Codex, …) changing
---

# AGENTS.md — working rules for agents in this repo

Instructions for any AI agent (Claude Code, Codex, …) changing
Awesome-Kernel-Workflows (AKW). Human contributors should follow them too.

When adding or modifying a kernel-optimization workflow, follow
[Agent.md](Agent.md) — the workflow maintenance guide. This file adds the
**runtime & consolidation rules** (below) and the **versioning & changelog
policy**. The common `AGENTS.md` filename serves as the entry point, so the
non-negotiable rules that keep 32 workflows consistent live here.

## Workflow code: the non-negotiable rules

A workflow is dispatched through Claude Code's `Workflow` tool, which loads the
`.js` as a **sandboxed script** (not a module) and injects
`agent/phase/parallel/pipeline/log/budget/args` as globals. Everything below
follows from that fact, and every rule is enforced by a CI guard — a PR that
violates one hard-fails. Getting it right by construction is cheaper than a
bounced PR.

### 1. Never hand-author the shared helper blocks — they have a single source

The mechanical helpers every workflow needs (`agentRetry`, `withTurnTimeout`,
`guard`, `expect`, `__attemptBlock`, `__experienceBlock`,
`normalizeSuitabilityValue`, `resolveBackendAxis`, `driverSh`, `driverPath`, the
arg-guard and typed-args blocks) are **single-sourced** in `_meta/scaffolding/`
and inlined into each workflow between sentinels:

```js
// --- BEGIN inlined backend-axis (resolve) scaffolding (from _meta/scaffolding/backend-axis.js) ---
...canonical block, byte-for-byte...
// --- END inlined backend-axis (resolve) scaffolding ---
```

Rules:

- **The region between `BEGIN inlined X` / `END inlined X` is generated. Do NOT
  edit it in a workflow `.js`.** A hand-edit to one copy is drift, and the
  matching guard test fails.
- To change a helper, edit the SSOT in `_meta/scaffolding/<X>.js`, then re-sync
  every workflow with that helper's codemod under `scripts/patch-<X>.js`
  (`patch-backend-axis.js`, `patch-arg-guard.js`, `patch-typed-args.js`,
  `patch-embedded-eval.js` take `--refresh`; `patch-turn-timeout.js` /
  `add-agent-retry-scaffolding.js` take a file list or `--all`). **The failure
  message of each guard test prints the exact command to run** — use that; don't
  guess the flag. Guards: `backend-axis-ssot-guard`, `scaffolding-ssot-debt-guard`,
  `agent-retry-guard-lint`, `turn-timeout-propagation-guard`, `typed-args-ssot`.
- When you write a **new** workflow, do not paraphrase these helpers from memory
  or from a nearby workflow — run the codemod so you get the current canonical
  block verbatim. A paraphrase that "looks equivalent" is the #1 source of drift.
- If a helper genuinely has no SSOT yet (currently `langToken` / `fenceToken`),
  say so in the PR and add the SSOT file — do not silently inline a fresh copy.

### 2. Eligibility lives in the manifest, not in the workflow body

Backend / language / problem-type eligibility is declared in `manifest.yaml`
`routing.accepts:` and enforced by the **KerSor selector** (issue #24). Therefore:

- **Do NOT emit `WORKFLOW_SUITABILITY` or `assertWorkflowSuitability()`.** They are
  retired; only two legacy workflows still carry them and both are being migrated.
  A new workflow with them fails the generator checklist.
- If a workflow resolves a backend from args, emit a small `resolveBackend()` that
  **normalizes/derives only — it must NOT throw on eligibility.** The selector
  owns rejection; the workflow does not re-litigate it.
- The manifest is the **contract SSOT**. Declare the contract there
  (`routing.accepts`, `backend.*`, args) and let the workflow read injected
  values — do not re-derive in JS what the manifest already states. Match the
  vocabulary of `docs/manifest-schema.yaml`.

### 3. Runtime sandbox constraints (all CI-enforced)

- **No line-leading ESM `import`.** The entrypoint is a script, not a module.
  There is no `require`/`import` of shared code — that is *why* the helpers in
  rule 1 are inlined rather than imported.
- **No `Date.now()`, `Math.random()`, or argless `new Date()`.** They break
  Workflow-tool resume (values differ across resumes → cached branches diverge).
  Derive any id from `args.run_index` / `args.round_index` / a filesystem counter.
  Enforced by the catalog forbidden-API scan (marks the workflow `known_broken`).
- **Wrap every `await agent()` in `agentRetry(fn, {retries, allowNull})`.** A bare
  `agent()` returning null on a transient 429 crashes the workflow via property
  deref. Enforced by `agent-retry-guard-lint` + `agent-retry-null-safety`.
- **Substrate CLI flags are `--artifact` / `--problem` / `--out`** — never
  `--kernel` / `--test` / `--result` (the run.sh parsers `exit 3` on those).
  Enforced by `substrate-flags-contract`.
- **All file writes go to `args.exp_dir`, never process CWD.** Writing to CWD
  pollutes the KerSor session tree. Enforced by `stray-files-static`.
- **`meta` stays a pure literal** — the Workflow tool extracts it statically.

### 4. Do the minimum, keep the method free

Consolidation removes duplication *below* the method (helpers, contract), never
standardizes the method itself. A workflow's phase spine and optimization logic
stay free-form — that is the point of having many workflows. Do not "standardize"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qhy991/Awesome-Kernel-Workflows](https://github.com/qhy991/Awesome-Kernel-Workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
