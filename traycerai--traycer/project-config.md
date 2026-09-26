---
trigger: always_on
description: GUI renderer for Traycer. Read with repo-root `AGENTS.md`. Treat as a normal
---

# AGENTS.md — clients/gui-app

GUI renderer for Traycer. Read with repo-root `AGENTS.md`. Treat as a normal
browser React app unless the task needs native/desktop integration.

**Stack:** Vite, React, TS, TanStack Router (file-based) + Query, Zustand,
Tailwind v4, shadcn/ui, Vitest + Testing Library.

## Commands

```bash
# from clients/gui-app/
bun run dev
bun run lint:files <paths>   # the files you changed; CI runs the whole-project lint
bunx vitest run <path>       # one test file; CI runs the suite
```

After .ts/.tsx changes, run react-doctor on the changed files only (manual; not
in pre-commit): `npx -y react-doctor@latest . --verbose --diff <base> --offline
--no-score`. `bun run react-doctor` scans the whole project.

After making changes, lint the files you changed with `bun run lint:files
<paths>` and fix all errors. `@shadcn/lint` runs there and reads
`components.json` and `src/index.css`, so its errors name this app's real
variants, sizes and tokens — the fix is in the message. Don't run `bun run
lint`, `test` or `build` here, and run `compile` only to diagnose its failure:
each is a whole-project run (the lint needs about 9 GB whichever files you
touched). The commit hook lints and compiles, and CI runs all four (see the
root `AGENTS.md`).

**A `shadcn/no-restyle` error is answered in `src/components/ui/`, not in
`eslint.config.mjs`.** Every design-system component has a CONTRACT in that
config saying what a call site may still write on it — usually `layout` and
nothing else, because a call site PLACES a component and the component owns how
it looks. The fix for "`px-3` is not allowed on `<Button>`" is a size; for a
colour, a variant; for a header band, a `layout` prop. Add the variant, migrate
the sites that were hand-rolling it, and delete their classes. Widening a
contract is the last resort and has one test: the reason has to be a fact about
the SITE, not about the component — if it generalises, it should have been a
variant. A treatment that is genuinely one file's own goes in
`restyleExemptions`, one entry per file, `allow` keyed by the contract it opens
and the reason written above it.

**Commits:** nothing needs running by hand before a commit — repo-root
`pre-commit` already runs the affected checks (see root `AGENTS.md`), and
`lint:files` above is feedback while you work, not a gate. Tests are CI, not
the hook. `react-doctor` stays manual (not hooked).

## Map

| Path                          | Role                                                                   |
| ----------------------------- | ---------------------------------------------------------------------- |
| `src/routes/`                 | File-based routes                                                      |
| `src/components/`             | App UI; `components/ui/` = shadcn primitives (compose, don't rewrite)  |
| `src/stores/`                 | Zustand (UI/client state only)                                         |
| `src/hooks/`                  | App hooks (`hooks/<ns>/use-<verb>-<noun>-{mutation,query}.ts`)         |
| `src/lib/query-keys/`         | Central query/mutation key builders                                    |
| `src/lib/commands/`           | Command palette sources + `actions/` (palette and UI call the same fn) |
| `src/stores/epics/open-epic/` | Per-epic Y.Doc projector — read code/tests before changing             |
| `src/providers/`              | App-wide providers                                                     |

Generated — don't hand-edit: `src/routeTree.gen.ts`, `dist/`, `.tanstack/`.

## Non-negotiable rules

- **`cn(...)`** from `@/lib/utils` for all composed `className`s. No template
  literals / `+` / `.join(" ")`. Static single strings OK.
- **Fluid layout sizing** — `w-full`, `max-w-*`, viewport caps. No fixed px/rem
  for layout surfaces (icons / touch targets OK). One recorded exception:
  Settings ▸ Layout's status-bar preview frame
  (`panels/layout/status-bar-preview.tsx`) is a SIMULATED viewport whose width
  control names a pixel width, so it draws `w-[480px]` / `w-[880px]` /
  `w-[920px]` — always under `max-w-full`, since a frame wider than the
  ~944px Settings pane silently pushes the strip's right-hand cluster
  off-screen. A new fixed-px layout width needs the same kind of argument.
- **Safe area** — never write `env(safe-area-inset-*)`; `index.css` owns the
  only reads. `#root` reserves the top and both horizontal insets app-wide
  (landscape is supported, so the sensor housing can be on either side), which
  makes every in-flow surface safe with no code of its own. Those edges have no
  opt-out, for content and surface backgrounds alike. What is left to a
  surface: the bottom edge (`pb-safe-bottom`), and anything `fixed`.
  - Window-filling: `h-safe-dvh` / `min-h-safe-svh` / `w-safe-dvw`, never the
    raw `h-dvh` / `min-h-svh` / `w-screen`.
  - Floating over the viewport: the `*-safe-<edge>-gutter` tokens, which are
    the layout's own 1rem or the device inset, whichever is larger.
  - `fixed` overlays are portalled outside `#root` and inset themselves:
    `top-safe-center-y` + `left-safe-center-x` when centred (the horizontal
    centre is displaced by half the DIFFERENCE between the side insets, since

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [traycerai/traycer](https://github.com/traycerai/traycer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
