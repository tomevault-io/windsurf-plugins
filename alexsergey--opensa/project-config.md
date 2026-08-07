---
trigger: always_on
description: - Write production-ready code
---

# Project Guidelines

## Important

- Write production-ready code
- Prefer small focused files
- Add comments only when necessary
- Prefer explicitness over magic
- Preserve existing architecture and conventions
- Follow existing project patterns before introducing new ones
- Prefer extending existing code over introducing new abstractions
- Reuse existing utilities before creating new ones
- Do not refactor unrelated code unless explicitly requested
- Split overly complex files into smaller modules
- Prefer minimal diffs
- Avoid reformatting unrelated code
- Prefer readable and maintainable code
- Prefer simplicity over clever abstractions
- Avoid unnecessary abstractions

---

## TypeScript Rules

- Use TypeScript strict mode
- Avoid `any`
- Prefer explicit types for public APIs
- Prefer `type` over `interface`
- Use discriminated unions when appropriate
- Prefer readonly where possible
- Avoid unnecessary generics

---

## React Rules

- Use functional components and hooks only
- Keep components small and composable
- Extract reusable logic into custom hooks
- Do not create components inside components
- Memoize expensive computations when needed
- Keep JSX clean and readable

---

## Styling

- Avoid inline styles
- Reuse utility classes when possible
- Keep styling readable
- Prefer composition over duplicated class lists

---

## API Rules

- Use async/await
- Handle loading states
- Handle error states
- Keep API logic outside React components
- Do not call APIs directly inside JSX

---

## Testing Rules

- Tests must be deterministic
- Keep tests readable and explicit
- Avoid duplicated setup logic
- Use descriptive test names

### Test Structure

- Negative test cases must go first
- Positive test cases must go after negative cases
- Negative and positive cases must be placed in separate `describe` blocks
- Separate describe blocks with an empty line
- Keep test structure consistent across the project

Example:

```ts
describe('formatValue', () => {
  describe('negative cases', () => {
    it('throws when value is invalid', () => {});
  });

  describe('positive cases', () => {
    it('returns formatted value', () => {});
  });
});
```

---

## Ignore Rules

Do not analyze or modify generated/dependency files.

Ignore:

- node_modules
- dist
- build
- coverage
- \*.generated.ts
- package-lock.json
- yarn.lock
- pnpm-lock.yaml

Never edit generated code manually.

---

## Cost Saving Rules

- Run only affected tests when possible
- Do not run the entire test suite for small isolated changes
- If only one test file was modified, run only that test file
- If only one component changed, avoid unrelated validations
- Read only files relevant to the current task
- Avoid scanning the entire repository unless necessary

---

## Standing Workflow Rules

- **CHECK `docs/project-goals.md` BEFORE writing an idea, a concept or a plan** — it is what the project is
  FOR, and it is directive, not aspirational. OpenSA is compatible with RenderWare; it is NOT a
  reimplementation of San Andreas. Two halves, and dropping either breaks it: **honour the authored DATA**
  (timecyc moods, handling rows, IDE flags, popcycle/cargrp — read them as the author meant, or the world
  stops behaving as designed and every mod written in those tables goes wrong in FEEL rather than in
  loading), and **do not port the LOGIC** — its execution, its data structures and its ceilings are one 2004
  machine's answer. We have our own engine and our own formats now, so a legacy limit is not our limit, and
  where we can beat the original we are REQUIRED to: matching a 2004 compromise is the choice that needs an
  argument, not improving on it. "That is what the original does" is the beginning of an argument, never the
  end of one. The goals also carry what keeps this honest — better must be DEMONSTRATED (measured or
  field-accepted, never assumed), performance is part of every feature's specification, and a mod author's
  data must keep working.
  **Read the goals first, then the restrictions**: one says what to aim at, the other what may not be done
- **CHECK `docs/restrictions/` BEFORE writing an idea, a concept or a plan** — it holds the rules a design
  has to satisfy (layer boundaries, format ceilings, engine splits, what is decided at build time and cannot
  be re-taken at runtime). A plan that violates one is not ambitious, it is a plan that gets rewritten after
  the first build. When a new restriction is discovered, it goes there in the SAME change, and every entry
  must say whether a violation is caught by a test/guard/lint or is SILENT — the silent ones are why the
  folder exists
- English only, repo-wide: no Cyrillic in any doc, comment, or committed file — paraphrase field verdicts
  in English (chat language stays whatever the user speaks)
- Record measured numbers into the plan doc after EVERY phase/step (before/after, representative log lines);
  a phase without its numbers is unfinished
- Every reported performance figure goes into `docs/benchmarks/` (per its readme's schema/index) BEFORE it
  is analysed — including numbers pasted in chat; always record which pak build a run read
- After a BIG rework (a migration, a subsystem rewrite, a major feature chain), run an audit AND a benchmark

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexSergey/opensa](https://github.com/AlexSergey/opensa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
