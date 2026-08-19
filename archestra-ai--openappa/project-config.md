---
trigger: always_on
description: OpenAPPA (APPA = Agentic Permissions Policy Algebra)
---

# CLAUDE.md

OpenAPPA (APPA = Agentic Permissions Policy Algebra)
is a value-granular information-flow policy engine for LLM agents. It sits
between the agent and its tools/inference and answers one question before
every proposed flow: *can this value, derived from these sources, legally flow
into this sink?* It is declarative and algebraic — no guardrails, no prompt
filtering, no bespoke `if`s; any imperative judgment lives in registered
external authorities and transformers, never in the engine.

## IMPORTANT
The golden set is `website/content/docs/how-it-works.md`,
`website/content/docs/contracts.md` and `website/lib/terms.ts`. Golden
files must agree with each other in every commit: a change that alters
what another golden file also states lands together with the matching
update, and a commit that leaves two golden files contradicting each
other is not allowed. Non-golden files — the code and the rest of the
website, for now — can be harshly outdated.

The normative specification is not in this repository. Where the spec
and this code disagree, the spec is right and the code has drift to
close; do not cite rule ids here.

## Naming

- Use the `appa` prefix for new OpenAPPA-owned crates, binaries, environment
  variables, and protocol identifiers. Existing unprefixed names are
  deliberate, not violations: core's internal module names (`engine`, `plan`,
  `turn`, …) and the reserved `assistant.response` sink. Never introduce new
  `baton`-named
  identifiers: `baton` was the earlier name and can happen only in stale spots.
- "Engine", "Trajectory", "Value", "Label", "Dimension", "Authority",
  "Transformer", "Remedy plan" are defined terms — use them as
  `appa-engine/src/lib.rs` defines them, not colloquially.
- **Agentic terminology first.** In comments, docs, and identifiers, lead with
  the agentic vocabulary: *trajectory* (not execution trace or session
  history), *flow* (not information transfer or operation), *turn*, *tool
  call*, *emission*, *actor/agent*, and *harness*. Use the classical IFC or
  security term when it adds precision or establishes lineage, and gloss it
  for public readers at first use. Never let it displace the agentic term as
  the primary name for a concept that has one.
- Do not invent new terms, especially when working with spec. Try to use 
  existing definitions. If you want to introduce a new one - ask a user and 
  explain why.

## Document precedence

1. `appa-engine/src/lib.rs` — concepts and semantics of the engine as
   implemented, and the reference for what a term means.
2. `website/content/docs/how-it-works.md` — the reader-facing
   introduction. `website/content/docs/contracts.md` is the
   policy-review guide, and `website/lib/terms.ts` restates the
   vocabulary as the website's term-popover definitions.

Non-normative still means consistent: a change to one golden file lands
with the matching update to the others.

## No history, no compatibility

APPA owes nothing to its own past. Docs describe the current model only
— no retired rules, no "formerly", no migration notes. Config and wire
surfaces may break without shims or deprecation paths.

## Collaboration

Applies to discussion and work in this repository.

- Lead with the result, finding, or decision the user needs. Report progress
  when it exposes a discovery, tradeoff, or blocker rather than narrating
  routine tool use.
- Assume the reader writes software but may not know this codebase or have a
  complete AI or security background. Explain repository and domain context
  needed to evaluate a decision; do not explain standard programming concepts
  unless asked.
- Surface decisions when reasonable interpretations would produce materially
  different work. Recommend one option and state the tradeoff; decide routine
  implementation details without asking.
- Match depth to the task. Keep routine implementation updates concise, but
  show the reasoning behind changes to the model, spec, architecture, or
  security guarantees.
- Use concrete references such as paths, types, and commands. Mark
  uncertainty as uncertainty rather than smoothing it into confident prose.

## Writing (public docs)

Applies to the website docs and other material written for readers
outside the project.
Except for correctness, defined terminology, and claim scope, these are
defaults rather than lint rules; depart from them when the document reads
better as a result.

**Audience and purpose**

- Write for senior engineers and technology leaders. Assume technical
  fluency, but not complete or current knowledge of both AI and security.
- Do not require prior training in information-flow control. Introduce the
  minimum specialized vocabulary needed to state the idea accurately.
- Match the presentation to the document. The spec is normative and precise;
  guides build a usable mental model; glossaries and references optimize for
  lookup. Product framing belongs in introductions and guides, not in
  normative or reference material.
- In guides, show how remedy plans and narrowing keep an agent productive when
  those behaviors are relevant. Do not force the value proposition into every
  section.

**Vocabulary**

- Use APPA's defined terms consistently. Terms that readers must type in TOML

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [archestra-ai/OpenAPPA](https://github.com/archestra-ai/OpenAPPA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
