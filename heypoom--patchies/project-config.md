---
trigger: always_on
description: **Patchies**: Visual programming environment for audio-visual patches. Connect nodes (P5.js, Hydra, Strudel, GLSL, JavaScript) to build creative projects with real-time collaboration and message passing.
---

# AGENTS.md

**Patchies**: Visual programming environment for audio-visual patches. Connect nodes (P5.js, Hydra, Strudel, GLSL, JavaScript) to build creative projects with real-time collaboration and message passing.

## Workflow Rules

- **CRITICAL**: Never start dev server manually. User will start if needed.
- **CRITICAL**: Never git commit or push for the user unless explicitly asked to do so. Wait for user review.
- **CRITICAL**: Never batch Read and Edit on the same file in parallel. Always Read a file first, wait for the result, then Edit it. The Edit tool requires the file to have been read in a prior step.
- Before implementing: update relevant spec files in `docs/design-docs/specs/`. Make sure specs are prefixed with numbers e.g. `50-foo-bar.md` and in the title too `# 50. Foo Bar`
- If asked explicitly to commit, write clear, short and concise commit messages following this format:

### Commit Message Format

```text
scope: description          # Most common — scope is the area of codebase
type(scope): description    # When type adds clarity (fix, feat, refactor, docs)
type(scope)!: description   # Breaking change
type: description           # No scope needed (docs, spec, chore)
```

**Common types**: `fix`, `feat`, `refactor`, `docs`, `spec`, `add`, `chore`
**Common scopes**: `transport`, `strudel`, `orca`, `clock`, `canvas`, `audio`, `bytebeat`, `csound`, `glsl`, `p5`, `hydra`.

- Use the object name as the scope when changes are object-specific.
- Use the module name as the scope when changes are module-specific.

Examples from this repo:

```text
transport: make transport panel beat indicator zero-indexed
feat(clock)!: use absolute time by default in parameter automation messages
fix(transport): reset lastPlayState on unsubscribe
refactor(orca): extract settings component into OrcaSettings
docs: shorten time signature docs
add beat object
```

Rules:

- Lowercase first word after colon
- No period at end
- Keep under ~72 characters
- Use imperative mood ("add" not "added")

## Core Stack

- **SvelteKit 5** + TypeScript
- **@xyflow/svelte** (node editor)
- **Bun** (package manager - use `bun install`)
- **Tailwind CSS 4** (Zinc/dark theme)
- **CodeMirror 6** (code editing)

## Development Commands

Run from `/ui` directory:

```bash
bun run dev              # Start dev server (USER starts this)
bun run build            # Production build
bun run check            # TypeScript & Svelte check
bun run lint             # Lint & format check
bun run test             # All tests
```

## Key Architectures

**Event Bus**: Type-safe system events (undo/redo, lifecycle, collaboration)

**Message System**: Max-style routing with `send()` / `recv()`, auto-cleanup on node deletion

**Rendering Pipeline**: FBO-based video chaining (P5 → Hydra → GLSL → Background). Topologically sorted render graphs.

**Audio System**: V2 AudioService (new) + V1 AudioSystem (legacy). Migrating nodes to V2 classes with async `create()` support.

**State**: Singletons (`MessageSystem`, `PatchiesEventBus`, `AudioSystem`) + Svelte stores + local storage auto-save

## Code Patterns

- **Always use `ts-pattern`**, never `switch` statements. This includes:
  - Conditional logic based on type/mode/state
  - Dynamic CSS class selection based on variants
  - Any branching on union types or enums

  ```ts
  // WRONG - never use switch
  switch (mode) {
    case "edit":
      return "bg-amber-600";
    case "multi":
      return "bg-blue-600";
    default:
      return "bg-purple-600";
  }

  // RIGHT - always use ts-pattern
  import { match } from "ts-pattern";
  match(mode)
    .with("edit", () => "bg-amber-600")
    .with("multi", () => "bg-blue-600")
    .otherwise(() => "bg-purple-600");
  ```

- Separate UI from business logic (manager pattern)
- TypeScript for all code
- Svelte 5: `$state`, `$props`, `$effect`, `$derived` (no `on:click`, use `onclick`)
- Prefer editing existing files
- **Shared functions over duplication**: When the same logic appears in multiple places (e.g., a message handler AND a context menu item), extract it into a named function and call it from both. Do NOT inline the same logic twice. If it's unclear whether a shared abstraction is appropriate, ask the user before duplicating.
- **Persistence**: Never store localStorage keys or persistence logic in components. Create a dedicated store in `src/stores/` (see `preset-library.store.ts` or `help-view.store.ts` for pattern)

## Styling

- Tailwind classes only (no custom CSS)
- Zinc palette, dark theme
- Support `class` prop for component extension
- Icons: `@lucide/svelte`

### Button Styling Rules

**MUST follow these rules for all buttons:**

1. **Always add `cursor-pointer`** - Buttons must show pointer cursor on hover
2. **Use `disabled:cursor-not-allowed`** - If a button has a disabled state, add this class
3. **Use shadcn-svelte Tooltip, NOT `title` attribute** - Native tooltips look inconsistent

```svelte
<!-- WRONG -->
<button title="Save changes" onclick={handleSave}>
  <Save class="h-4 w-4" />
</button>

<!-- RIGHT -->
<Tooltip.Root>
  <Tooltip.Trigger>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heypoom/patchies](https://github.com/heypoom/patchies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
