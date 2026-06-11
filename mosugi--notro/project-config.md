---
trigger: always_on
description: - **User interactions (responses, explanations, questions)**: Reply in Japanese
---

# notro

## Language Settings

- **User interactions (responses, explanations, questions)**: Reply in Japanese
- **Documentation (README, CHANGELOG, comments, commit messages)**: Write in English
- **Code**: Use English including comments

---

## Claude Code Working Rules

### Styling Guidelines

- Use **TailwindCSS 4 utility classes only** for all styling
- Do not use inline styles (`style="..."` attributes)
- Do not use `<style>` tags inside Astro components — **exception**: notro-ui Notion block components (e.g. `Toggle.astro`, `TableBlock.astro`) may use `<style>` for complex CSS selectors (pseudo-elements, parent-context selectors) that cannot be expressed as Tailwind utilities
- Notion block component styles live in **`<style>` blocks within each component file** — do not add component-specific styles to `global.css`
- Design tokens (`--notro-*` variables), color utilities (`notro-text-*`, `notro-bg-*`), and markdown/mermaid wrapper styles (`.notro-markdown`) are defined in `global.css`
- `nt-*` classes in `global.css` are page layout design tokens (`nt-text-*`, `nt-bg-*`, `nt-border-*`) and are separate from Notion block styles
- Do not manipulate styles directly via `element.style.*` in client-side `<script>` tags
- Control visibility using **class manipulation** such as `element.classList.toggle("hidden")` — do not use `element.style.display`

### Build & Layout Verification

After changing code, always verify with the following steps:

```bash
# 1. Type-check + build (run from repo root)
pnpm run build

# 2. Visually check the build output in a browser for layout issues
pnpm --filter notro-blog run preview
```

- Do not push if `pnpm run build` fails
- Always run `pnpm --filter notro-blog run preview` to visually verify styling changes before committing

### notro-ui Component Management

Notion block components are managed via the `notro-ui` CLI. `packages/notro-ui/src/templates/` is the single source of truth; `templates/` holds a copy of it.

#### Basic Flow

```bash
# Add all components to a new template (skips existing files)
notro-ui add --all

# Pull updates from notro-ui into the template (overwrites local changes)
notro-ui update --all --yes
```

#### Commands

| Command | Behavior |
|---------|----------|
| `notro-ui init` | Generates `notro.json` |
| `notro-ui add [name...] [--all]` | Adds components (**skips existing files**) |
| `notro-ui update [name...] [--all] [--yes]` | Updates components (**overwrites local changes**) |
| `notro-ui remove [name...] [--all]` | Removes components |
| `notro-ui list [--installed]` | Lists available / installed components |

#### Guidelines

- **Protecting customized files**: `add` skips existing files by default. Re-running `add` will not overwrite components you have customized in the template
- **Intentional updates**: Use `update` only when pulling changes from notro-ui upstream. Running without `--yes` shows a confirmation prompt
- **`notro.json`**: Configuration file placed at the project root. Tracks `outDir`, `stylesDir`, and the list of installed components. Commit to git

---

### Sub-agent & Branch Management

- Branches worked on by sub-agents (Agent tool) must be **merged into the caller's branch** before pushing
- Do not push sub-agent branches directly
- Always verify the build passes after merging before pushing

### Branch Naming

- All branches Claude works on must use the `claude/` prefix
- Include the session ID at the end of the branch name (`claude/feature-name-XXXXX` format)
- **Pushes will fail with a 403 error without the `claude/` prefix and session ID**

### Multiple Implementation Options

- When multiple implementation approaches are possible, **always present the options to the user via `AskUserQuestion` before starting work**
- Do not decide on an approach and proceed on your own
- Present options with concrete trade-offs (performance, maintainability, implementation cost, etc.)

### Commit Message Guidelines

- Commit messages **must always be in English**
- Format: `<type>: <summary>` (e.g. `feat: add tag filter to blog list`)
- Type must be one of: `feat` / `fix` / `refactor` / `docs` / `chore` / `style` / `test`

### UI/UX Criteria

- Design with **internationalization in mind** as a baseline (text length, character width, RTL considerations)
- Clarity and usability for the user is the top priority
- Prioritize information clarity and accessibility over decoration

### Astro Implementation Best Practices

#### Extract Logic into `.ts` Files

- Keep **minimal code** in Astro file (`.astro`) frontmatter (`---` blocks)
- Extract data fetching, transformation, and business logic into functions in `.ts` files under `src/lib/`
- Write unit tests for all extracted functions (using `vitest`)
- Test scope:
  - Functions under `templates/blog/src/lib/` (when adding or changing)
  - Publicly-called functions under `packages/*/src/utils/`
  - Astro components (`.astro`) themselves do not need tests — extract logic to `.ts` and test those functions

```astro
// Good: extract logic to src/lib/posts.ts; frontmatter only imports and calls
---
import { getCollection } from "astro:content";
import { getSortedPosts, excludeFixedPages } from "@/lib/posts";

const allPosts = await getCollection("posts");

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mosugi/notro](https://github.com/mosugi/notro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
