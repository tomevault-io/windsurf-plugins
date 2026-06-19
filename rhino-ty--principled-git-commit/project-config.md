---
trigger: always_on
description: >
---


# Commit Conventions

> Universal commit conventions derived from Conventional Commits 1.0.0 + Tim Pope + `awesome-copilot/git-commit` + a 200-commit empirical study. Project-specific dialect (domain proper nouns, custom trailers, workflow integrations like PDCA) lives in the project's `docs/references/COMMIT.md` — see §15 Dialect Loading.
>
> **TL;DR**: `type(scope): summary` (≤100 chars, lowercase, imperative) + 1-2 line context + `- ` bullets (avg 16 lines). No `##` headers. English body by default. Atomic + leaves-repo-green + why-over-what.

---

## 0. Principles

A commit must satisfy four readers — humans **and** AI:

1. **`git log --oneline` scanner** — wants context from a single line (most common reader)
2. **`git blame <file>` tracer** — wants to know "why is this line here?" while debugging
3. **`git bisect` hunter** — wants to isolate the exact commit that broke production
4. **AI agent** — rebuilds context after `/clear`, reviews PRs, generates changelogs, answers natural-language history queries. Depends on `grep` + embedding search + consistent structure. Atomic units, domain keywords, English body, and explicit trailers are decisive.

The five principles below are the minimum that satisfies all four readers simultaneously. Return here when in doubt.

### 0.1 Atomic — one commit, one intent

Bundle exactly one logical change. Never mix "fix + style + docs" in one commit.

→ **Bisect, revert, and cherry-pick all assume atomic units.** Mixed commits also confuse AI summarization — the LLM grabs one intent and hallucinates the rest as a single coherent change.

> Test: "If I had to revert this commit, would I drag along unrelated changes?" If yes, split.

### 0.2 Leaves repo green — every commit builds clean

Each commit, taken on its own, must pass `tsc --noEmit` + `lint` + `build` (or your project's equivalent green-build set). No "WIP, next commit will fix this."

→ **Bisect signal integrity.** A broken-state commit in the middle of history poisons `git bisect` results — the bisect ends up blaming the wrong commit. AI-driven regression analysis suffers the same way.

### 0.3 Why over what — body explains motivation, not changes

The diff already shows **what** changed. The body's job is **why** — the motivation, the trade-off, the alternative you rejected, the constraint that forced this approach.

→ **Answers the 6-month-later blame question directly.** AI can extract intent from the body without re-deriving it from the diff (diff inference is expensive and prone to misinterpretation).

```
❌ Body: "modify foo.ts to add bar method"        (diff already shows this)
✅ Body: "extract bar() out of inline closure —    (the WHY)
        prevents re-allocation on every render
        causing useMemo deps invalidation"
```

### 0.4 Imperative mood — "If applied, this commit will..."

Summary is in imperative mood. Matches git's own internal messages (`Merge`, `Revert`, `Initial commit`).

→ **Consistent verb pattern.** Auto-changelog tooling and AI classification (type inference, change summarization) both depend on stable verb forms.

```
✅ add idempotency-key support to /v1/payments     (If applied, this commit will add ...)
✅ fix double-render in <Modal> on focus return
✅ migrate auth hashing from bcrypt to argon2id
❌ added idempotency-key support                   (past tense)
❌ adding idempotency-key support                  (gerund)
❌ idempotency-key support added                   (passive)
```

### 0.5 Searchable — keyword-rich summary and body

Name domain nouns, function names, file paths, SoT names, and component names explicitly. Vague verbs (`improve`, `update`, `enhance`, `cleanup`) must be paired with a concrete noun.

→ **Targets `git log --grep` and AI embedding search.** Six months later, a query like "where did we add idempotency keys to the payments endpoint?" should hit exactly one commit. Vague keywords cause AI hallucination; concrete keywords get pinpoint accuracy.

```
❌ feat(api): improve checkout
✅ feat(api/payments): add Idempotency-Key header support with 24h replay window

❌ refactor: cleanup auth helpers
✅ refactor(auth): replace bcrypt with argon2id (memory-hard against GPU attackers)
```

Keyword checklist: scope path / function or component name / pattern name / SoT name / domain enum.

---

## 1. Format

```
type(scope): summary

[optional 1-2 line context — why]

- bullet: concept- or file-level change
- bullet: ...

[optional Verification: block]
[optional trailers — Refs / BREAKING CHANGE / project-dialect trailers]
```

### 1.1 Summary line

| Property | Rule | Note |
|---|---|---|
| Length | **≤100 chars recommended** (empirical avg 71, max 116) | Strict 50/72 not enforced — clarity beats brevity rule |
| Form | `type(scope): summary` | Scope is optional but almost always specified |
| Subject case | **lowercase** (`feat(...): add ...`) | 200/200 in study were lowercase. Proper nouns retain their case |
| Language | **English by default**. Project-specific proper nouns (e.g., Korean product names) keep their original script — see project dialect | |
| Trailing punctuation | none | |
| Mood | imperative — see §0.4 | |

### 1.2 Body

- **`- ` bullets dominate**. In the 200-commit study: 981 bullets vs 5 `##` headers — headers are an outlier and therefore noise.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhino-ty/principled-git-commit](https://github.com/rhino-ty/principled-git-commit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
