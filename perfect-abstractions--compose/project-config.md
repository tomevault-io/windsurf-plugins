---
trigger: always_on
description: > This file provides repository-scoped guidance for GitHub Copilot, AI code assistants, automation, and all contributors to [Compose](https://compose.diamonds/docs/).
---

# Repository Assistant / Copilot Instructions

> This file provides repository-scoped guidance for GitHub Copilot, AI code assistants, automation, and all contributors to [Compose](https://compose.diamonds/docs/).  
> It summarizes the most important principles, rules, and patterns. The canonical source of truth remains the public documentation and maintainers' decisions.

---

## 0. Quick Machine-Readable Checklist (Assistants & Bots)

When generating code, PRs, or issue actions:

- PRIORITY: Favor readability and self-contained facets over clever abstractions.
- FACETS: One file, no imports, no inheritance, no modifiers, no constructors.
- LIBRARIES: Self-contained, all functions `internal`, no `using X for Y`.
- VISIBILITY: Facet storage vars have no visibility specifier (except `internal` for `immutable`/`constant`).
- FUNCTION VISIBILITY (facets): Only `external` (entry points) or `internal` (rare helpers).
- STYLE: Parameters prefixed with `_`; camelCase names; braces required for every `if` / `else`.
- BANNED: In facets & libraries: inheritance, constructors, modifiers, public/private functions, external library functions, ternary operator, single-line ifs w/o braces, `selfdestruct`, `using` directives in libraries, visibility specifiers on storage variables.
- TESTS: Add/adjust tests for behavior changes; harness pattern respected.
- APPROVALS: At least 1 core maintainer approval for non-doc changes, 2 for breaking/critical.
- LABELS: Apply appropriate labels (bug, enhancement, docs, security, breaking-change, chore).
- COMPATIBILITY: Preserve externally observable standard behavior (events, errors, return values).
- STORAGE: Reuse structs by copying; only remove trailing variables; maintain original order.
- NO SURPRISES: Document deviations from standards and justify them.
- DO NOT AUTO-MERGE: Human approval required for on-chain logic.
- UPDATE DOCS: If behavior, interface, or rules change, update docs + this file in same PR.

---

## 1. Purpose

- Provide concise guidance to assistants, automation, and contributors.
- Act as the authoritative short-form of contribution, issue, and review policy.
- Prevent drift between code style, facet architecture, and documented design philosophy.

Canonical documentation:
- Main Docs: https://compose.diamonds/docs/
- (Add specific deep links as they are published: contributing, code-review, issues, security, design, testing.)

If a conflict arises: maintainers decide; update this file to match accepted decisions.

---

## 2. Scope

Applies to:
- Code suggestions (AI or human).
- PR authoring and review workflow.
- Issue creation, labeling, and triage.
- Facet & library architecture decisions.
- Automation enforcing style or rules.

Not granting:
- Automatic merge rights for on-chain code.
- Modification of security policy workflows without explicit approval.

---

## 3. Core Philosophy — Compose Is Written to Be Read

The highest priority: code must be easy to read and understand.

Why:
- Clarity builds confidence.
- Understandable code reduces bugs and accelerates correct extension.
- Consistency matters: the library should appear written by one careful author.

We invest in documentation + accessible code so users can confidently build and audit.

---

## 4. Facet Design Principles

### Facets Are Self-Contained
- One facet = one source file = one coherent functional unit.
- No inheritance, no cross-file dependencies, no imports.
- Each facet intended to be added as a complete unit (not partially sliced).

### Facets Are Read Top-to-Bottom
- Definitions precede use.
- Avoid forward mental jumps and indirection unless strictly improving clarity.

### Repeat Yourself (Selective Anti-DRY)
- Prefer inline clarity over indirection.
- Internal helpers are allowed only when they:
  - Encapsulate a fully self-contained action.
  - Remove large blocks of identical logic.
- Repetition is acceptable when it enhances immediate comprehension.
- Example: `internalTransferFrom` in ERC-721 where block duplication would harm clarity.

Guideline: Repeat yourself when it helps understanding; apply DRY only if it improves readability.

---

## 5. Banned Solidity Features — Facets & Libraries

Purpose: Keep deployed on-chain logic minimal, explicit, and consistent.

Exceptions: Tests, scripts, or external consumer projects may use these features.

Rules & Rationale:

1. No inheritance  
   - Replace with on-chain facet composition. Reduces coupling.

2. No constructors  
   - Use explicit initializer functions in deployment flows.

3. No modifiers  
   - Inline checks improve transparency and auditability.

4. No visibility specifiers on facet storage variables  
   - Diamond storage pattern (EIP-8042) makes visibility redundant.  
   - Exception: `immutable` and `constant` may be declared `internal`.

5. Facet functions: only `external` or `internal`  
   - No `public` or `private`—keeps external ABI explicit and internal use minimal.

6. Library functions: all `internal`  
   - No external library entry points; do not use `using Lib for X` inside Compose libraries.

7. No ternary operator  
   - Use explicit `if / else` blocks for clarity.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Perfect-Abstractions/Compose](https://github.com/Perfect-Abstractions/Compose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
