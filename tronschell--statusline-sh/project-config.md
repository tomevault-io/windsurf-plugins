---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

This project uses **Bun** for the frontend bundler + test runner and **Wrangler** for the Cloudflare Worker. Do not introduce Node, npm, or Vite.

```bash
bun install

# Local dev — two processes:
bun dev                            # Bun SPA dev server (src/dev.ts) on :3001
bun --cwd worker dev               # Worker + D1 (wrangler --local) on :8787
bun run build                      # one-shot SPA build → ./dist (served by Vercel)

# Tests
bun test                           # root suite (frontend + shared + e2e)
bun --cwd worker test              # Worker suite
bun test test/compiler.test.ts     # single file
bun test -t "preserves settings.json" # single test by name pattern
```

The SPA reads `NEXT_PUBLIC_WORKER_URL` (build-time constant inlined into the bundle via `build.ts`'s `define` map; localhost falls back to `http://localhost:8787`, production falls back to `https://statusline-community.zoniixyt.workers.dev` — the Worker's `workers.dev` host, used because `api.statusline.sh` is not yet on Cloudflare DNS). Vercel serves `./dist`; the Worker handles all `/community`, `/designs`, `/install`, `/i/:id.{sh,ps1}`, `/robots.txt`, `/sitemap.xml`, and `/og/community/:slug.svg` routes cross-origin via CORS + Turnstile.

## Architecture invariants

### One IR, three backends — keep them in lockstep

`packages/shared/src/compiler/ir.ts` lowers a `Design` to a `RenderOp[]`. Three consumers must produce byte-equivalent output (after stripping ANSI) for the same fixture:

- `packages/shared/src/compiler/bash.ts` → bash script (jq → python3 → python fallback for JSON field reads)
- `packages/shared/src/compiler/powershell.ts` → PowerShell (uses plain `ConvertFrom-Json`; writes via `[Console]::Out.Write` to bypass PS color mangling)
- `packages/shared/src/compiler/interpret.ts` → ANSI string in pure JS (browser preview, hero animation, static cards)

`test/compiler.test.ts` enforces parity by spawning the compiled bash with mock JSON on stdin and diffing against the interpreter. **Any new `RenderOp` variant must be added to all three backends and the parity test in the same change.** Current ops: `literal`, `field`, `cond`, `progressBar`, `split`, `compute`, `rotator`. Computed expressions (`git_branch`, `git_dirty`, `cost_fmt`, `duration_hms`, `duration_human`) live in each backend's prelude.

### Adding a new element type

The discriminated union and its lowering touch seven places. Miss one and the preview will silently diverge from the installed script:

1. `packages/shared/src/types.ts` — add to the `Element` union
2. `packages/shared/src/schema.ts` — runtime validation + `ELEMENT_TYPES` array (hand-rolled, no Zod)
3. `packages/shared/src/compiler/ir.ts` — `elementToOps` switch case
4. `packages/shared/src/compiler/{bash,powershell,interpret}.ts` — handle any new `RenderOp` kinds
5. `src/frontend/store/designStore.ts` — `defaultsFor` factory in the `addElement` path
6. `src/frontend/components/Palette/ElementPalette.tsx` + `Inspector/fields/<Type>Fields.tsx` (or reuse an existing field component, as the `rateLimit*` elements do with `ContextBar`/`ContextPct` fields)
7. `test/compiler.test.ts` — parity coverage

### Installer safety rules

`worker/src/install/{bashTemplate,psTemplate}.ts` wrap the compiled statusline into a self-contained installer served at `/i/:id.{sh,ps1}` by the Worker. Two non-negotiable invariants:

- **Embed the compiled script via a quoted heredoc** (`<<'STATUSLINE_EOF'`) for bash, or a **single-quoted here-string** (`@' ... '@`) for PowerShell. No shell expansion of user content — paths and special chars must round-trip byte-for-byte. The templates pre-escape collisions (`STATUSLINE_EOF` → `STATUSLINE_EOF_X`, `'@` → `'@_X`) on the way in.
- **Merge `settings.json` structurally, never via string replacement.** Bash tries `jq` then `python3` then `python`; PowerShell uses `ConvertFrom-Json`. Every other top-level key (`model`, `permissions`, `mcpServers`, ...) must survive. A timestamped `.bak.<unix-s>` is written before the merge. `test/e2e.test.ts` enforces both behaviours end-to-end by running the real installer against a temp dir.

`STATUSLINE_SELFHEAL=1` opt-in: only if all JSON backends fail AND `claude` is on PATH, the installer shells out to `claude -p` with a 4 KiB settings snippet. Stays opt-in — never enable by default.

### State + persistence

- `designStore` (Zustand + persist, key `statusline-design-v1`, only `design` is persisted): all mutations go through `withHistory`, which `structuredClone`s the prior design onto `past` (capped at 50). Do not mutate `design` outside of these actions or undo/redo will desync. `updateElement` deliberately drops a stray `type` field in the patch to prevent accidental type rewrites.
- `uiStore` (key `statusline-ui-v1`): panel collapse, OS override, mock preset, mock stdin JSON, self-heal toggle.
- `useShareState` (sessionStorage, keys `statusline-design-id-v1` / `statusline-slug-v1`): `{designId, slug}` so refreshes keep the "Saved as <id>" indicator. Clearing `designId` also clears `slug`.

### Database


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tronschell/statusline.sh](https://github.com/tronschell/statusline.sh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
