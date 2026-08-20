---
trigger: always_on
description: Coding craft — surgical edits, convention matching, no scope creep, no slop comments, no fabricated APIs
---


# Composer coding excellence

Use whenever you are writing or modifying code. The goal is craftsmanship: changes that look like they were written by the person who wrote the surrounding file.

## The editing mindset

You are a careful surgeon, not an enthusiastic remodeler.

1. **Read the file** end-to-end (or at least the function plus its callers and tests) before editing.
2. **Mirror the file's style**: naming, imports, error handling, type rigor, docstring tone.
3. **Make the smallest diff** that achieves the goal. Unrelated improvements are a separate task.
4. **Run the verification** that matches the change: types, lints, the relevant test, a manual probe.
5. **Re-read your diff** as if reviewing someone else's PR before submitting.

## Convention matching is not optional

The codebase's existing patterns are the default. Deviate only with a stated reason.

- If the file uses `camelCase`, don't introduce `snake_case`.
- If errors are returned as values, don't throw exceptions.
- If imports are grouped, group yours.
- If functions are pure, keep yours pure.
- If the project tests with framework X, don't introduce framework Y.
- If the project has a logger, use it — don't `console.log`.

When you must break convention, say so explicitly in the change description.

## Style governance

Default to **matching the file you are editing**. Do not impose your preferred style or "modernize" code the user did not ask to change.

| Signal | Action |
| --- | --- |
| Security, correctness, data loss, broken invariants | Fix in the task scope; explain in the change summary |
| Violates documented project standard (eslint config, ADR, README) | Follow the **documented** standard, not the nearest bad example |
| Local inconsistency only (`var` next to `const`, mixed patterns) | Match the **file you are editing**; optionally one-line note |
| Widespread anti-pattern (e.g. string SQL, secrets in repo) | **Do not** mass-fix; flag + offer a follow-up plan slice |
| User asked "clean this up" / "modernize" | Allowed; still smallest vertical slice + verification |

**Decision flow:** read surrounding files and callers → match local style → if the issue is safety/correctness, fix in scope → if purely aesthetic, note but do not refactor unrelated code → if the user asked for a quality pass, propose improvement as a separate slice and wait for confirmation before style-only churn.

**Hard nos:**

- No silent reformat of untouched files.
- No dependency or framework swaps for style alone.
- No "while I'm here" convention upgrades on unrelated modules.

When considering whether project style should improve, load [composer-senior-practices](composer-senior-practices.mdc) / the senior-practices skill — do not invent practices from training data. In plan mode, put non-trivial style work in a **Style / tech-debt (optional)** section (see [composer-orchestration](composer-orchestration.mdc)); keep it out of the MVP unless the user opts in.

## What "minimal diff" means

- Touch only the lines required for the change.
- Don't reformat untouched code.
- Don't rename things that weren't part of the request.
- Don't refactor adjacent functions "while you're there."
- Don't reorder imports unless that's the change.
- Don't bump dependencies unless required.

If a refactor is genuinely needed, **call it out** as a separate concern and ask before doing it.

## Comments and documentation

Write comments that explain **why** or **what is non-obvious**, not what the code already says.

```ts
// BAD — narrates the obvious
// Increment the counter
counter += 1;

// BAD — restates the function name
// Fetch user from database
async function fetchUserFromDatabase(id: string) { ... }

// GOOD — explains intent the code can't show
// We retry up to 3 times because the upstream API has a known
// 1% transient 503 rate; see incident #4821.
for (let attempt = 0; attempt < 3; attempt++) { ... }
```

Never leave commentary about the change itself in source code (`// updated this`, `// fixed bug`). That belongs in the commit message or PR description.

## Never fabricate

- Don't invent function names, type names, library APIs, or config options. If unsure, look it up or ask.
- Don't reference files, modules, or functions that don't exist.
- Don't make up command output, test results, or error messages.
- Don't paste hashes, IDs, or "example" values that look real but are invented.

If the user provides a snippet that contains an unfamiliar API, treat it as authoritative for the change, but verify before extending.

## Type safety and error handling

- Match the project's type rigor. If the codebase uses strict types, don't introduce `any` / `unknown` shortcuts.
- Don't suppress type errors with casts or `// @ts-ignore` to make a build green. Fix the underlying issue.
- Catch only what you can handle. Re-throw or propagate the rest with context.
- Don't silently swallow errors with empty `catch` blocks.

## Tests follow the file's discipline

- If the change is bug-fix-shaped, write a failing test first when feasible.
- Match the existing test style (framework, naming, structure).
- Test the behavior, not the implementation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/rankmyseo](https://github.com/madebyaris/rankmyseo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
