---
trigger: always_on
description: This repository contains `draft-ietf-acme-dns-persist`, an IETF Standards Track Internet-Draft specifying an ACME challenge type for persistent DNS-based domain control validation. The document is authored in kramdown-rfc markdown and processed by the `kramdown-rfc` toolchain into xml2rfc XML and then RFC-format text.
---

# Copilot Review Instructions

This repository contains `draft-ietf-acme-dns-persist`, an IETF Standards Track Internet-Draft specifying an ACME challenge type for persistent DNS-based domain control validation. The document is authored in kramdown-rfc markdown and processed by the `kramdown-rfc` toolchain into xml2rfc XML and then RFC-format text.

Review every pull request against the rules in this file. Flag violations with the relevant section reference from this file (not from the draft — the draft's section numbers change).

## Scope of These Rules

These rules apply to substantive content review — changes to normative text, examples, security considerations, or cross-references. Typo fixes, reference URL updates, CI-only changes, and presentation-deck updates do not require the full protocol and security analysis. Apply format checks (§1) to all PRs; apply the deeper review philosophy (§3–5) only where the PR changes protocol behavior.

## Review Comment Style

Consolidate observations per section rather than emitting one comment per line. Prefer a single top-level summary that names the issues, with specific inline comments only where they add precision (e.g., exact wording proposals, anchor references). Do not restate the checklist in each comment. If many issues cluster in one area, group them under a single heading in the summary.

---

## 1. Document Format: kramdown-rfc

Hard constraints on the source format:

- **Bibliographic references**: Normative references use `{{!RFCNNNN}}` (with `!`). Informative references use `{{?RFCNNNN}}` (with `?`). Bare `{{RFCNNNN}}` is a build error. Every RFC cited in normative text MUST use `{{!...}}`.
- **Section anchors**: `{#anchor-name}` after headings, lowercase-hyphenated. Cross-references use `{{anchor-name}}`. Verify all cross-references resolve.
- **Section markers**: `--- abstract`, `--- middle`, `--- back` delimit document structure. Content before `--- abstract` is frontmatter only.
- **BCP 14 boilerplate**: The directive `{::boilerplate bcp14-tagged}` must remain in the Conventions and Definitions section.
- **Figures**: Fenced code blocks use `~~~` (not backticks). Figure titles use `{: #fig-id title="..."}` on the line after the closing fence.
- **Definition lists**: Use kramdown `:` syntax.
- **Frontmatter**: YAML block between `---` delimiters. Do not modify frontmatter structure without understanding xml2rfc implications.

---

## 2. Normative Language (BCP 14)

This document uses BCP 14 keywords per RFC 2119 and RFC 8174.

- **MUST / MUST NOT**: Absolute. Use only when non-compliance breaks interoperability or security. Every MUST imposes an implementation burden — verify it is justified.
- **SHOULD / SHOULD NOT**: Expected behavior with permitted deviation. RFC 2119 requires the specification to state what happens when the requirement is not met. Flag any bare SHOULD without a deviation consequence.
- **MAY**: Truly optional. Verify interoperability is preserved whether or not the feature is implemented. If a MAY creates a dependency on other implementations, it may need to be SHOULD or MUST.
- **Capitalization**: BCP 14 keywords are normative only when fully capitalized. Lowercase "must", "should", "may" are ordinary English. Flag accidental capitalization in non-normative contexts.
- **Consistency**: Two statements about the same behavior must use the same keyword strength. Flag contradictions.

---

## 3. Protocol Review Philosophy

### Persistence Changes Everything

This protocol's defining property is that the validation signal persists in DNS. Every review question flows from this: persistent records have longer exposure windows, survive infrastructure changes, and remain valid even when the domain owner's intent has changed. Any new text must be evaluated against the question: "What is the impact of this being long-lived?"

### Cross-Flow Consistency

The draft defines multiple paths to validation. When a PR modifies one path, check all others:

- Some paths operate without a challenge object. Requirements that depend on challenge object fields must account for their absence.
- Some paths skip the challenge-response interaction. Requirements that assume client interaction must account for flows where the CA acts unilaterally.
- The challenge response payload is an extension point (RFC 8555 §7.5.1). New fields must degrade gracefully with servers that ignore them.

### Forward Compatibility

The draft requires unknown parameters to be ignored. This is a hard design constraint: any new parameter that requires enforcement cannot rely on existing implementations recognizing it. Flag PRs that introduce parameters assuming universal strict enforcement.

---

## 4. Account Identity Review Philosophy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ietf-wg-acme/draft-ietf-acme-dns-persist](https://github.com/ietf-wg-acme/draft-ietf-acme-dns-persist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
