---
trigger: always_on
description: This document contains guidelines and rules for AI coding assistants working with this repository.
---

# AI Agent Rules for qubership-envgene Repository

This document contains guidelines and rules for AI coding assistants working with this repository.

## Documentation Structure (Diátaxis Framework)

This repository follows the [Diátaxis documentation framework](https://github.com/evildmp/diataxis-documentation-framework).

### Documentation Types

1. **How-to Guides** (`/docs/how-to/`)
   - Goal-oriented, practical steps
   - Solve specific problems
   - Minimal theory, maximum action
   - Target: ~200-400 lines

2. **Explanation** (`/docs/features/`)
   - Conceptual understanding
   - "Why" questions
   - Background and context
   - Design decisions and trade-offs

3. **Reference** (`/docs/`)
   - Technical specifications
   - Object schemas
   - API documentation
   - Factual, precise

4. **Tutorials** (`/docs/tutorials/`)
   - Learning-oriented
   - Step-by-step for beginners
   - Complete working example

### When Creating Documentation

**✅ DO:**

- Keep how-to guides focused and practical
- Separate theory into explanation documents
- Link between documentation types
- Use clear, descriptive titles
- Include realistic examples from the codebase

**❌ DON'T:**

- Mix how-to and explanation in one document
- Create long (>500 lines) how-to guides
- Include detailed theory in practical guides
- Use fantasy/made-up examples

---

## Always-loaded essentials

### Verify, don't fabricate

When a documentation statement names a specific identifier - a parameter, environment variable, file
path, library symbol - that identifier is confirmed in the source it describes. Unverifiable
identifiers are open questions, not statements of fact.

For object schemas and example fields, see also
[Object Examples in Documentation](#object-examples-in-documentation).

❌ **INCORRECT:**

- Naming a CI variable for a service by extending a pattern from a sibling service, without checking
  the implementation.
- Listing a config file path from memory without grepping the repository.
- Assuming a library exposes an env-var auth method by analogy with another component.

✅ **CORRECT:**

- Grep or read the source code to confirm the identifier before stating it.
- Mark the identifier as an open question until verifiable.

**Scope:** Applies to **new and modified content only**.

**Why:** Documentation is consumed as authoritative. A fabricated detail propagates into tickets,
validation rules, and tooling assumptions.

---

### Use existing vocabulary

If the document already defines terms, types, and notations for a domain, reuse them. Parallel
vocabulary - new section titles, column labels, role names - for concepts the document already covers
is avoided.

❌ **INCORRECT:**

- Inventing a column name that describes the same property an existing column already covers.
- Adding a structural subsection that duplicates an existing section type.
- Coining a new term when the document already names the same concept.

✅ **CORRECT:**

- Reuse the document's existing terms for the same concepts.
- If new vocabulary is genuinely needed, introduce it in a definitions section.

**Scope:** Applies to **new and modified content only**.

**Why:** Parallel vocabulary forces readers to maintain two mental glossaries and produces ambiguous
cross-references.

---

### Define every term

**Every domain term a document uses is defined. A term used in one document is defined in that document. A term
used across documents is defined once in the glossary, and each document links to it.**

Three rules govern terminology. This rule governs whether a definition exists and where it lives.
[Use existing vocabulary](#use-existing-vocabulary) governs which term to pick.
[Don't re-gloss established terms](#dont-re-gloss-established-terms) governs how often to restate it.

The glossary lives at [/docs/glossary.md](/docs/glossary.md). A term needs a definition when a competent reader
from outside this repository could misread it. This covers ordinary words used with a specific meaning here,
such as Environment or Effective Set.

- **Single-document term.** Define it on first use in that document, as a sentence, a parenthetical, or a short
  definitions list.
- **Cross-document term.** Add or reuse a glossary entry, then link to it from each document instead of
  restating the definition.
- **Promotion.** When a term defined in one document starts appearing in a second, write a glossary entry for
  it and replace the inline definition in both documents with a link.

❌ **INCORRECT:**

- Using a shared term such as Deploy Postfix with a fresh inline definition in each document that mentions it.
- Introducing a term with no definition in the document or the glossary, leaving the reader to infer it.

✅ **CORRECT:**

- A term local to one how-to guide is defined in that guide.
- A term shared by [calculator-cli.md](/docs/features/calculator-cli.md) and
  [envgene-objects.md](/docs/envgene-objects.md) has one glossary entry that both documents link to.

**Scope:** Applies to **new and modified content only**. Existing multi-document terms are back-filled into the
glossary only when the surrounding lines are edited for other reasons.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Netcracker/qubership-envgene](https://github.com/Netcracker/qubership-envgene) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
