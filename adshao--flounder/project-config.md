---
trigger: always_on
description: - Treat this repository as a public open-source project by default.
---

# AGENTS.md

## Project Defaults

- Treat this repository as a public open-source project by default.
- External-facing content must be written in English. This includes README files, docs, CLI help, security policy, contribution notes, prompts, package metadata, and generated public reports.
- Use TypeScript as the default implementation language.
- Use pi-mono primitives by default for agent/runtime integration, especially `pi-ai` and `pi-coding-agent`. Choose a different framework only when there is a concrete technical reason and document that reason.
- Keep model/provider selection runtime-configured. Do not assume every model family is available through every pi provider; use `codex-cli` only as an explicit local fallback when the user selects it.
- Keep the architecture ready for future coding-agent use cases. Separate ingestion, source indexing, agent tools, verification, reporting, and security policy guardrails.
- Prefer typed interfaces, schema validation, deterministic tests, and small extension points over ad hoc agent logic.
- Treat deterministic project profiles, source indexes, checklist seeders, and lens packs as planning aids only. They may route attention, but they must not produce vulnerability findings.
- When new learning materials appear during a specialized audit, study the target domain first so the audit has the required protocol, cryptography, proof-system, financial, or application-specific expertise before running or finalizing the audit.
- In live audits, prefer `flounder run`: give the model a thin capability surface and let it decide how to inspect the target's assets, trust boundaries, invariants, and attacker model.
- Treat `rounds` and `trials` as different mechanisms. Rounds must generate novel checklist coverage from prior observations; trials independently audit one item for stochastic agreement.
- Later exploration rounds must use duplicate filtering and coverage deltas. Do not call repeated single-pass audits "multi-round" unless they add new source-grounded audit items.
- Let project-specific configuration add context, lens packs, failure modes, and auditor agents without modifying core code.
- For blind proof runs, disable deterministic checklist seeders so the model must enumerate the relevant audit item itself before any audit trial can produce a finding.

## Thin-Layer Agentic Mode

- The framework's default and public driver is `flounder run` (thin agentic per phase). With a clue, `run` is the product pipeline: prepare -> sealed map/dig -> confirm -> report. With explicit source paths, `run --source` is the sealed map/dig audit. Keep each phase thin and agentic: the framework may orchestrate phase boundaries and durable state, but it must not inject a framework-owned audit strategy.
- In agentic mode the framework provides capabilities and guarantees, not strategy. A new component is justified only if it gives the model an affordance it lacks (read/search source, run an isolated local test, recall prior runs) or a guarantee the model cannot self-provide (execution confirmation, sandbox isolation, command safety, durable replayable state). Do not add taxonomy, domain playbooks, or search schedules to the audit path; if a human prior is useful, expose it as an optional model-callable tool, not as injected prompt preamble.
- Keep the one hard opinion: a claim is not a finding until a local test confirms it. `report_finding` may only reach `confirmed-executable` by citing a `run_test` that actually passed. Never let the model upgrade confirmation by assertion.
- All generated-test execution must route through the shared sandbox module and the command-safety policy. Verification stays local-only.
- Prefer making audit mode benefit from stronger models without framework changes. Resist re-introducing framework-side direction of how the model should reason.

## Map → Dig Deep Coverage

- The sealed discovery surfaces `run --source` / `map` / `audit` share the tools, the confirmation gate, and the network-sealed boundary; they differ only in coverage. `flounder run --source` = map → audit for source you already have (`--quick` runs a single breadth pass instead). `flounder map` = enumerate + persist the scope inventory only (no dig). `flounder audit <region>` = deep-audit one named region (skip the map). `flounder audit --scope <id>` = dig inventory items (after a `flounder map`). `flounder audit --verify <file>` = confirm-or-refute given suspected findings. (Internally these select the same breadth / map / dig / verify phases as before — the verb is just the CLI surface.)
- Map → dig: MAP enumerates a complete scope inventory (the model applies general lenses — spec conditions, value/asset flow, trusted-but-unbound inputs — and scores each by exposure × difficulty); DIG deep-audits the highest-scored scopes obligation-by-obligation. The framework encodes no domain analysis — both the enumeration and the scoring are the model's; the framework only parses the inventory the model wrote, sorts by the model's score, and pins each dig to a scope's region.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adshao/flounder](https://github.com/adshao/flounder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
