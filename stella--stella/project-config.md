---
trigger: always_on
description: Stella is an open-source legal workspace.
---

# Stella Development Guidelines

Stella is an open-source legal workspace.

## Ideal Customer Profile (ICP)

**Current focus: mid-size law firms, 5–50 lawyers.**
Pragmatic, cost-conscious, not overly technical.

**Scale target: Magic Circle firms (2,000–5,000+ lawyers).**
The architecture must not block scaling to this level. See the Scalability section
below for decision guidelines.

International audience: do not assume English language or English typography
conventions are universal. Highlight competing standards (date formats, quotation
marks, citation styles, legal terminology) when relevant.

## Project Overview

**Monorepo:** `apps/api` (Elysia backend, Bun), `apps/web` (React + Vite frontend),
shared packages in `packages/`. Use Glob/Grep to explore.

## Workspace Layout

- `apps/*` contains runnable applications only.
- `packages/*` contains shared or publishable packages only.
- Every direct child of `apps/` and `packages/` must be a workspace package named
  `@stll/<directory>`.
- Use scoped workspace filters in commands, for example
  `bun --filter @stll/web dev`.

## Commands

`bun run dev` | `dev:web` (3000) | `dev:api` (3001) |
`build` | `lint` | `format` | `typecheck` | `test` |
`db:push`

## Documentation Access

The `stella-docs` MCP server provides on-demand access to library documentation via
`llms.txt`. When implementing features, fetch the relevant docs first using
`list_doc_sources` and `fetch_docs` tools.

**Not covered (no `llms.txt`):** Tailwind CSS, oxfmt. For these, use `WebFetch` or
`WebSearch` directly.

**Setup:** run `bun run setup:mcp` once after cloning.

## Meta Preferences

- Never manually reformat code you did not semantically change (auto-formatter
  output from `bun run format` is fine to include)
- Vary punctuation: prefer colons, semicolons, commas, and parentheses over em dashes
- Prefer explicit over implicit — when a backend endpoint accepts a discriminator
  (e.g., `?type=document|file`), thread it through the full stack (URL params,
  component props) instead of hardcoding a default on the frontend
- If TypeScript can make a class of bug structurally impossible (branded types,
  discriminated unions, exhaustive checks), prefer that over runtime validation or
  manual discipline
- Conventional Commits: `feat:`, `chore:`, `fix:`, `docs:`
- Rebase feature branches onto main (linear history)
- Fail fast — validate at boundaries, return/throw early
- Minimize brace nesting — invert conditions, early returns
- Use named constants, not string literals for domain values
- No direct `document.cookie` assignment
- Avoid spread in loop accumulators (use `.push()`)
- If you encounter a pre-existing bug or lint error while working on something else,
  fix it (separate commit)

## Regulated Industry

Stella handles privileged legal data. All code must meet **SOC 2 Type II** and
**ISO 27001** standards: least privilege, audit trails, encryption, workspace
isolation, ethical walls. Full checklist in `/conventions-security`.

## Design Principles

- No hidden complexity; code is the docs. Every operation must work for humans,
  scripts, and AI agents alike.
- No lock-in: standard formats, self-hosting is first-class.
- AI is a tool, not a persona. No anthropomorphizing.
- Performance is non-negotiable. Batch operations, minimize round-trips, lazy-load
  aggressively.
- **Vertical slices over horizontal layers.** Features are independent end-to-end
  slices (own routes, components, handlers). New capabilities land in their own slice;
  existing code stays untouched.

## Scalability

Never paint yourself into a corner. Architecture must support Magic Circle scale
without a rewrite. Never return unbounded result sets; keep the API stateless; filter
by tenant ID in the query. Full guidelines in `/conventions-scale`.

## UX & Brand

Use semantic tokens (`bg-muted`, `text-foreground`, `border`), not raw colour values.
Full brand deck, micro-interaction guidelines, and visual noise rules in
`/conventions-ux`.

## Coding Conventions

### TypeScript

- No enums — use `as const` objects or union types
- Model mutually exclusive internal states as discriminated unions with a stable
  `type`, `status`, or domain-specific discriminator. Avoid boolean flag sets plus
  optional payload fields when only some combinations are valid.
- When the linter blocks an `as` cast, restructure to narrow properly (type guards,
  `in` checks, records instead of arrays). If truly unavoidable, ask before adding and
  include a `// SAFETY:` comment explaining why the cast is sound.
- When a type mismatch appears, trace it to the source (e.g., the handler or query
  that produces the wrong type) rather than casting at the consumer. Check git to
  verify you didn't introduce the mismatch yourself before blaming the framework.
- Use `.at(0)` when the element may not exist (signals possible absence). Use `[0]`
  only when existence is already established (length check, or a `// SAFETY:` comment).
- Prefer arrow functions over function expressions
- Destructure in the parameter when the intermediate variable is not reused
  (e.g., `{ body: { file, name } }` not `body` then `const { file, name } = body`)
- Prefer discriminated union narrowing (`obj.type === "x"`) over `"key" in obj`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stella/stella](https://github.com/stella/stella) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
