---
trigger: always_on
description: - **KISS — Keep It Simple & Stupid**
---

# Co-Pilot Instructions — Clean, Simple, Consistent

## Core Principles

- **KISS — Keep It Simple & Stupid**
  - Prefer the *simplest* working solution.
  - Remove optional parameters, layers, or abstractions unless proven necessary.

- **DRY — Don’t Repeat Yourself**
  - Extract small helpers only when repetition is real (≥2 places now, likely 3+).
  - Reuse existing utilities before creating new ones.

- **YAGNI — You Aren’t Gonna Need It**
  - No speculative features, flags, or extensibility points.
  - Implement only what today’s requirement needs.

- **Minimal Diff**
  - Change as little code as possible to solve the problem.
  - Prefer **surgical edits** over rewrites; keep file count stable.
  - Avoid introducing new dependencies and new files unless unavoidable.

- **Consistency Over Cleverness**
  - Match the project’s **naming, patterns, and structure**.
  - Prefer patterns already used in the codebase to new paradigms.

---

## Guardrails for Copilot

When proposing code, **adhere to all of the following**:

1. **Stay in Place**
   - Modify existing functions before adding new ones.
   - If a helper is needed, place it near its first use within the same file.

2. **Match Style**
   - Mirror existing naming, file layout, import style, and error handling patterns.
   - Keep public API shapes and function signatures stable unless a bug requires change.

3. **No New Files by Default**
   - Do not create new modules/components/hooks unless duplication or complexity becomes worse without them.
   - Never create standalone markdown documentation files (e.g., PR tickets, feature docs, summaries).
   - Embed all information directly into code comments or verbally respond to user.
   - **Exception:** Internal tickets can be created in `.github/INTERNAL-TICKETS/` for feature planning (gitignored, local only).

4. **Zero Surprises**
   - Avoid side effects, global state changes, or cross-cutting refactors.
   - Keep behavior backward-compatible unless the task explicitly requests otherwise.

5. **Only Change When Explicitly Told**
   - Do NOT make changes unless explicitly asked, given clear permission, or strongly hinted.
   - Do NOT assume what should be changed or refactored.
   - Ask for clarification if instructions are ambiguous.
   - Respect the current state of the code unless directed otherwise.
   - Do NOT "improve" or refactor code without being told to do so.

6. **Documentation Updates**
   - When making changes to **`forms-editor`** or **`forms-renderer`** packages, remind the user to update the documentation site (`apps/mieweb-forms-docs`) if:
     - Public API changes (props, methods, exports)
     - New features are added
     - Breaking changes occur
     - Usage examples need updating
   - **Note:** `forms-engine` is an internal package used by the other two - it does not require documentation updates unless the changes affect the public-facing packages
   - The documentation site is at `https://questionnaire-builder.opensource.mieweb.org/`

---

## Decision Checklist (run before editing)

- [ ] Is the change the **simplest** that works?  
- [ ] Does it **reuse** existing utilities/components?  
- [ ] Is the **diff minimal** (fewest lines/files touched)?  
- [ ] Does it **preserve names** and coding style?  
- [ ] Did I avoid **new files/deps/config**?  
- [ ] Are error cases handled like nearby code handles them?  
- [ ] Are performance and readability balanced (favor readability if perf is fine)?

If any box is unchecked, **simplify**.

---

## Preferred Patterns

- **Early returns** instead of nested `if`s.  
- **Small helpers** only when repeated or clearly improves clarity.  
- **Immutable updates** where the codebase already does so.  
- **Localize complexity**: keep tricky logic private/internal.  
- **Keep functions short** (< ~40 lines when possible).

### React & Tailwind Specific

**IMPORTANT: Always reference these copilot instructions when working with React and Tailwind in this project.**

- **CRITICAL: Use `mie:` Prefix ONLY in Package Components**: The `mie:` prefix is **ONLY** for components in:
  - `packages/forms-engine`
  - `packages/forms-editor`
  - `packages/forms-renderer`
  
  **DO NOT use `mie:` prefix in**:
  - `apps/web-demo-packages` (standalone Vite app - uses standard Tailwind)
  - `apps/mieweb-forms-docs` (Docusaurus site - uses standard Tailwind)
  
  These package components are consumed by external applications with global CSS resets, so they need the scoped `mie:` prefix to avoid conflicts.

- **CRITICAL: Always Use `mie:` Prefix for ALL Tailwind Classes in Packages**: When working in package components, **every single Tailwind utility class MUST have the `mie:` prefix**. This applies to:
  - Regular classes: `mie:flex`, `mie:p-4`, `mie:bg-miesurface`
  - Pseudo-state variants: `mie:hover:bg-mieprimary`, `mie:focus:ring-2`
  - Responsive variants: `mie:md:flex-row`, `mie:lg:hidden`
  - Combined variants: `mie:hover:bg-mieprimary` (NOT `mie:hover:mie:bg-mieprimary`)
  - Template literals: `` `mie:px-4 ${condition ? "mie:bg-mieprimary" : "mie:bg-miebackground"}` ``
  
  **CRITICAL: Prefix Placement** - The `mie:` prefix goes BEFORE variant modifiers:
  - ✅ `mie:hover:bg-mieprimary` (CORRECT)
  - ❌ `hover:mie:bg-mieprimary` (WRONG - will not work)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mieweb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
