---
trigger: always_on
description: This repository is an evidence-preserving workspace for the Platform Engineering
---

# Repository Agent Contract

This repository is an evidence-preserving workspace for the Platform Engineering
Kaigi 2026 session described in `README.md`.

## Language

- Write machine-only instructions and contracts in English. This includes this
  file, `00_meta/`, source code, schema descriptions, and validation messages.
- Write repository content in Japanese. This includes Raw Notes, Observations,
  Hypothesis Episodes, Patterns, adopted Artifacts, and their human-readable
  templates.
- Write participant- and contributor-facing repository guidance in Japanese.
- Keep identifiers, frontmatter keys, enum values, paths, and relation types in
  English. Every governed content node must declare `content_language: ja`.
- A verbatim quotation may preserve its source language, but its surrounding
  description and interpretation must be Japanese.
- Do not mix English prose into Japanese content merely because an upstream
  template or source uses English. Proper nouns and established technical terms
  are allowed.
- Do not translate canonical identifiers, keys, enum values, or paths.

## Required reading order

Before creating, interpreting, promoting, or editing repository content, read:

1. `00_meta/repository-contract.md`
2. `00_meta/provenance-schema.md`
3. `00_meta/promotion-policy.md`
4. `00_meta/analysis-lenses.md`
5. `00_meta/relation-schema.md`
6. `00_meta/naming-conventions.md`

The files above define how truth is handled. They do not define the truth about
the session.

## Non-negotiable behavior

- Treat `01_working/raw-notes/` as immutable source material.
- Apply the draft-finalization and confidentiality exceptions defined in
  `00_meta/repository-contract.md` before applying Raw Note immutability.
- Interpret `Raw` as an epistemic position, not as a requirement for rough,
  short, unstructured, or human-written prose. Do not promote a Raw Note solely
  because GenAI organized it or because it is highly polished.
- Never move or delete a Raw Note because a derived node was created.
- Preserve incorrect source statements and append a correction instead of
  silently rewriting history.
- Never preserve customer, project, personal, commercial, internal-system, or
  credential information merely for provenance. Sanitize it before commit and
  never repeat removed values in corrections, filenames, logs, or summaries.
- Do not turn a short note into a confident claim without recording the
  interpretation and its uncertainty.
- Every derived claim must cite one or more repository node IDs through typed
  relations.
- Keep observation, interpretation, hypothesis, decision, and current artifact
  distinct. Do not collapse them into one document.
- Use lightweight Hypothesis validation by default. Choose one primary learning
  approach for the current step: `experiment`, `research`, or `interview`.
- Treat `research` as an approach, not an Evidence grade. A recognized
  reference may resolve a bounded terminology, conceptual-model, or published-
  guidance uncertainty, but its reputation alone does not establish causal
  effect, prevalence, audience demand, or target-context applicability.
- Close or route each new lightweight Episode with `proceed`, `revise`,
  `validate_further`, `stop_for_current_scope`, or `not_decided`. An
  `inconclusive` result is completed learning and remains active only when the
  recorded disposition requires further validation or remains undecided.
- Add Validation Components, Evidence Coverage, Finding, and Applicability only
  when a Hypothesis contains multiple decision-relevant uncertainties or a
  formal residual-risk decision needs a stable component target.
- Keep lightweight or extended validation results, residual uncertainty, Human
  Risk Decisions, and Artifact adoption distinct. A decision to proceed with
  risk changes none of the hypothesis result, evidence basis, or adoption state.
- Do not infer that a placeholder, directory name, template, or planned
  artifact is an adopted conclusion.
- Do not invent missing provenance, validation results, participant behavior,
  metrics, or evidence. Use `unknown`, `unverified`, or an explicit limitation.
- Preserve `knowledge_basis` as an axis separate from human review, confidence,
  validation result, and adoption. In particular, do not interpret
  `practitioner_experience` with `not_tested` as unsupported, independently
  validated, or universally true.
- Do not infer practitioner experience, case counts, or expertise from
  seniority, framework vocabulary, prose quality, or GenAI plausibility.
- Never set a derived node to `status: reviewed` without explicit human
  confirmation that the node represents the human's intended meaning. Do not
  treat agent self-review, validation, or publication-safety review as that
  confirmation. Record the human reviewer, confirmation timestamp, and
  `review_scope: intent_alignment`.
- Authorization to create, update, sanitize, or proceed is not review of
  persisted content. Follow the applicable finalization skill's persisted-node
  handoff and later-human-message requirement before recording human review.
- Never set an analysis node to `status: accepted`. Record human adoption only
  in `03_artifacts/` with an explicit, traceable adoption decision.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skijima404/pek2026-value-stream-to-practice](https://github.com/skijima404/pek2026-value-stream-to-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
