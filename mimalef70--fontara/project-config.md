---
trigger: always_on
description: When the user asks to commit changes, first inspect the worktree carefully
---

# FontARA Agent Rules

## Git commit workflow

When the user asks to commit changes, first inspect the worktree carefully
before staging anything:

1. Run `git status --short` to list modified, deleted, and untracked files.
2. Run `git diff` and review the actual unstaged changes with care. Do not rely
   only on file names or `git status`.
3. Identify which changes belong to the requested task and which changes are
   unrelated or user-owned. Never stage unrelated changes just because they are
   present in the worktree.
4. Stage files explicitly by path. Avoid broad staging commands such as
   `git add .` unless every changed file has been reviewed and belongs in the
   commit.
5. After staging, run `git diff --cached --stat`,
   `git diff --cached --name-status`, and inspect any important staged hunks
   with `git diff --cached` before committing.
6. Run the relevant verification commands before the commit when practical. For
   broad extension changes, prefer `pnpm check`, `pnpm build:all`, and
   `pnpm lint:extension`. For narrow docs-only changes, a focused review is
   enough.
7. Write a clear, standard commit message that describes the user-facing or
   architectural intent, not just the files touched. Prefer concise imperative
   phrasing, for example `Improve font unicode and RTL handling`.
8. After committing, run `git status --short` and `git log -1 --oneline` to
   confirm the worktree state and the exact commit created.
9. If the user also asks to push, check `git status --branch --short` and
   `git log --oneline <upstream>..HEAD` before pushing so it is clear which
   commits will be sent.

## UI components and shadcn/ui

For extension UI work, prefer shadcn/ui primitives over one-off custom
components when a matching primitive exists. Add only the components that are
needed for the current change. For new shadcn/ui components, use the official
CLI as the default source of truth:

```sh
pnpm dlx shadcn@latest add <component>
```

Keep `components.json` aliases aligned with the real project layout before
running the CLI. For this extension the shadcn aliases point to `src` through
`@/*`, with `ui` at `@/ui/components/ui`, `hooks` at `@/ui/hooks`, and `utils`
at `@/utils/cn`.

When the repo's aliases or output paths do not match the extension layout, first
run the CLI with `--dry-run`, `--diff`, or `--view`, then adapt the generated
component into the project structure. Do not hand-write a shadcn component from
memory when the CLI can provide the current implementation.

Project convention:

1. shadcn/ui components live in `src/ui/components/ui`.
2. Import them from that local component module, matching the existing project
   style. For example, from `src/ui/options/index.tsx` use:

   ```ts
   import { Button } from "../components/ui/button"
   import { SidebarProvider } from "../components/ui/sidebar"
   ```

3. Keep generated shadcn component APIs recognizable and close to the upstream
   component names, so future `shadcn add <component>` output can be compared
   or merged without guesswork.
4. When adding a new shadcn component, run the CLI first, then use the official
   shadcn docs and CLI output as the source of truth. If the component behavior,
   composition, or API is unclear, ask DeepWiki about the shadcn/ui repository
   before inventing a local pattern.
5. Do not introduce unrelated UI libraries for primitives already covered by
   shadcn/ui.
6. For Persian or RTL extension surfaces, apply the shadcn RTL guidance before
   shipping: pass `dir="rtl"` where the component accepts direction, keep
   controls on the visual start side, flip directional icons with RTL variants,
   and prefer logical Tailwind utilities such as `text-start`, `start/end`,
   `ps/pe`, and `border-s/e` over physical left/right utilities unless the
   component documentation explicitly requires physical positioning.

## Extension i18n

User-facing extension UI text must go through the shared i18n catalog in
`src/i18n/messages.json` and the local React i18n layer in `src/ui/i18n`. Do not
add hard-coded Persian, English, or Arabic strings inside React components
unless the value is product data such as a font name, author name, URL, or brand
name.

When adding a visible UI string:

1. Add the key to `src/i18n/messages.json` for `en`, `fa`, and `ar`.
2. Use `useI18n().t(...)` in React components, with interpolation for dynamic
   values instead of string concatenation.
3. Use `useI18n().formatNumber(...)` and `formatVersion(...)` for visible
   numbers and versions.
4. Keep document `lang`/`dir` controlled by `src/ui/i18n/bootstrap.ts` and
   `I18nProvider`; do not hard-code page-level `dir="rtl"` in React roots.
5. Store interface language preference through `STORAGE_KEYS.UI_LANGUAGE`.
6. For manifest/store-facing text, add entries under the `extension` section of
   `src/i18n/messages.json`. Build output generates `_locales/*/messages.json`
   from this catalog; do not edit `_locales` files by hand.

## Site CSS from matched-selector JSON

When a site CSS file is generated from inspected JSON, use the captured JSON as
the source of truth for font stacks and text targets. In the default mode, use
the JSON `matchedSelector` values after the required normalizations below. Do

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mimalef70/fontara](https://github.com/mimalef70/fontara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
