---
trigger: always_on
description: - Always prefer the **simplest solution**. If something feels complex, refactor
---

## Principles

- Always prefer the **simplest solution**. If something feels complex, refactor
- **Simplicity ≠ laziness.** Creating a reusable component for repeated patterns IS the simple solution—it maintains consistency and quality. Leaving duplication "because it's only N files" leads to inconsistency (bugs). DRY is about having a single source of truth, not just reducing typing. When you see the same pattern repeated, extract it
- Favor **clarity and minimalism** in both code and UI
- **Do not create formatting-only diffs.** Preserve existing formatting unless a line needs a semantic change. Formatting is handled by `oxfmt`, so never manually reflow unrelated code, expand one-line calls/types/objects into multiline, or collapse multiline code into one line unless that exact code is being changed for behavior.
- Follow design inspirations from Apple, Linear, Vercel
- Code must be modular, following SOLID and DRY principles
- Avoid nested conditionals and complex logic
- Prefer short and composable functions
- Avoid nested business logic inside `map`, `filter`, `flatMap`, and `reduce`. If an array callback needs branching, multiple steps, or non-trivial normalization, extract it into a named helper
- Prefer top-level functions that read like pipelines of named domain operations. The main function should describe the flow; helpers should describe the rules
- If a condition or transformation is important enough to think about, it is important enough to name
- Keep inline callbacks trivial. Simple property access or a one-line predicate is fine; anything more should become a helper
- When logic feels nested, split it by responsibility: matching, normalization, filtering, transformation, and merging should usually be separate functions
- **Split files with multiple concerns.** If a file has distinct responsibilities (e.g., utils, validation, parsing, main logic), extract them into a `_utils/` folder (if internal) or separate files. A file should have one clear purpose, avoid doing too many things in a single file
- Prefer functional programming over OOP
- Avoid mutations: return new values instead of modifying existing data or state
- Use `[condition && value, ...].filter(Boolean)` instead of `let` + `.push()` for conditional arrays
- **Never use `let` + reassignment to compute a value.** Extract a helper function with early returns instead (e.g., `function getLabel() { if (x) return a; return b; }`). For objects, use helper functions that return the result (e.g., `const { a, b } = await getOrCreate(...)`). See `getComparisonLabel` in `metric-comparison.tsx` for the pattern
- Use meaningful variable names and avoid abbreviations
- When defining functions with two or more parameters, use a single object parameter with named fields instead of positional arguments. Named fields are self-documenting at the call site, order-independent, and safer against accidental swaps. Positional params are fine for single-argument functions, framework callbacks with well-known signatures (e.g., `map(item, index)`), and functions wrapped with React `cache` (object params create a new identity on every call, breaking memoization)
- For workflow orchestration, prefer linear wave-based flows (core-workflow style) with `Promise.allSettled` over branching orchestration unless branching is strictly required
- Use linear, declarative code over nested conditionals and imperative code
- Don't be afraid to refactor existing code to improve quality, clarity, or simplicity. Always leave the codebase better than you found it
- Never cut corners or do hacks. Aim for maintainable, clean code
- Think about the big picture—how your changes fit into the overall architecture and future growth
- **Think from first principles.** Don't accept patterns just because they're common. For every piece of code, ask: "What is this actually doing? Is there a simpler, more declarative way?" For every test, ask: "Am I testing my business logic or just testing that React/the browser works?" Before finishing any change, review everything again: "Is this the best way to implement this? Is this the best way to test this? Am I missing anything?" Think like a top 0.1% engineer who deeply cares about quality and details, not just getting things working. Think like the best engineer in the world
- Preserve behavior, not implementation details
- For compatibility issues, test simpler equivalent shapes before building workarounds
- **Verify before fixing.** When evaluating review comments (AI or human), trace the actual code path to determine if the reported issue can happen in practice. Answer "can this actually happen?" before "should we fix it?" Distinguish real bugs from theoretical issues from code style — and say which one it is upfront. Don't default to agreement; apply the same rigor you'd apply to your own code
- **Treat specs and GitHub issues as guidance, not truth.** If the ticket pushes the code toward a worse design or a wrong assumption, call it out and propose the better approach instead of implementing it blindly.

## Engineering Mindset


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zoonk/zoonk](https://github.com/zoonk/zoonk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
