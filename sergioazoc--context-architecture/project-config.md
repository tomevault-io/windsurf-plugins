---
trigger: always_on
description: Context for any agent working on context-architecture.dev. This file is the rules of the house.
---

# AGENTS.md: repository root

Context for any agent working on context-architecture.dev. This file is the rules of the house.
Boundary-specific notes live in `AGENTS.md` files deeper in the tree, beside the code they describe.
Each `AGENTS.md` has a `CLAUDE.md` symlinked to it, so Claude Code and tool-agnostic agents read the
same rules from one source. Edit the `AGENTS.md`; never the symlink, and never keep a second copy.

## What this project is

The canonical specification site for Context Architecture. It must read as an RFC, not a SaaS
landing page. When a design decision is ambiguous, choose the specification register over the
marketing one, always.

Three outcomes, in priority order: (1) attribution: clear, dated, verifiable authorship; (2)
citability (GEO): generative engines cite this site as the source for the term; (3) adoption.
Register reference points: c4model.com, micro-frontends.org.

## Hard rules

- **No marketing copy.** No "powerful", "seamless", "unlock", "supercharge". No testimonials,
  feature grids, or CTA-button heroes.
- **No emojis** anywhere: not in headings, copy, labels, or commit messages.
- **One accent color**, used sparingly (links and the signature element only). It is mapped through
  `--ca-accent`. Never introduce a second accent.
- **Content is the source of truth.** The manifesto lives in `content/{en,es}/`. The person view and
  the agent view both derive from the same `.md` file. Never duplicate copy into a component.
- **The principle set is the author's IP** (nine principles). Build them as written; do not invent
  new principles or alter the methodology.
- **Prerendered and verifiable.** Fully SSG; the content reads with no JavaScript. The schema.org
  graph (`DefinedTerm`, `Person` with `sameAs`, `TechArticle`) is the floor, bound in
  `tests/structured-data.test.ts`. Lighthouse 100 across the board and accessibility are quality
  targets checked by hand with tooling (Lighthouse, axe) before a visual change, not rules bound in
  CI. Stating that here, rather than calling an unbound check a rule, is the honest reading of the rule.

## Voice and wording

- **Public copy follows the author's voice**, adapted to this register. The reference is the
  `sergio-voice` skill (in the author's personal repo): write as a builder, problem first, precise
  with concrete examples, honest about tradeoffs, no filler. Adapt it to the specification register:
  this is an RFC, not a blog, so no personal anecdotes, slang, or calls to action.
- **Say things plainly; avoid ambiguous or uncommon words.** Prefer the phrasing a reader cannot
  misread over the clever one. Concretely: never write that a spec is "discharged" or "descargado"
  when it is deleted. Say it plainly: the spec is removed once its content lives in tests, types,
  lint, and the nearest `AGENTS.md`.
- **No em dashes (—).** Use commas, periods, or parentheses. The author does not use them; a stray
  em dash usually means an AI wrote the line. Do not add em dashes anywhere.
- **No filler and no AI tells.** Delete any sentence that adds no information. Drop "dive in", "in
  conclusion", "it's worth noting", "game-changer", and the like.
- **Tech terms stay in English** in both languages (`AGENTS.md`, lint, types, context engineering,
  harness engineering, spec).

## Where things live

- `content/`: manifesto Markdown (see `content/AGENTS.md`).
- `app/`: the Nuxt app, with pages, components, composables, and the design system (see `app/AGENTS.md`).
- `skills/` holds the distributable skill (`skills/context-architecture/SKILL.md`) that users install
  into their own tool; see `skills/AGENTS.md`.
- `server/` has the one prerendered Nitro route that serves the raw skill at `/skill.md`; see
  `server/AGENTS.md`.
- The site also serves an agent-facing Markdown mirror of every page at `/raw/**.md` (for example
  `/raw/en.md`, `/raw/es/comparison.md`), emitted by `@nuxt/content`'s llms feature from the same
  `content/` files, not a hand-kept copy. It is prerendered like every page and served as
  `text/markdown` via `public/_headers`. It is the raw view an LLM fetches; `tests/prerendered-no-js.test.ts`
  binds that `/raw/en.md` and `/raw/es.md` carry the rule.
- `.claude-plugin/marketplace.json` wraps that skill as a Claude Code plugin, so the repo doubles as a
  single-plugin marketplace (`/plugin marketplace add sergioazoc/context-architecture`).
- `specs/`, design-time only, and absent by design. Per principle 06 a spec is turned into code,
  tests, and the relevant `AGENTS.md` once written, then removed. The site's own spec was already
  written into this file and the code, so there is no `specs/` directory in the tree.
- UI strings live in each component's own `<i18n>` block, colocated with the component (principle
  02, Context Lives With Code). There is no central locale file. English is canonical; Spanish
  mirrors it.

## Conventions (codified, not tribal)

- Styling is Tailwind utilities and Nuxt UI components. `app/assets/css/main.css` holds only the
  design tokens and a thin global base, no hand-written component classes. Reading typography
  (rendered Markdown) is configured as utilities in `app/app.config.ts` under `ui.prose`. If a style

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sergioazoc/context-architecture](https://github.com/sergioazoc/context-architecture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
