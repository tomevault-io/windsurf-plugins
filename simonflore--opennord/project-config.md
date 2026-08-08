---
trigger: always_on
description: Guidance for Claude Code (and other AI agents) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (and other AI agents) working in this repository.

## What this is

**OpenNord** — an open, AI-native companion for **Nord® keyboards**, reading
program/preset files in the browser (no keyboard or Nord Sound Manager required)
and adding a community patch library, AI search/explanation, and direct USB
transfer to/from the instrument. The architecture is **family-wide**: one shared
CBIN container and one shared vendor-USB transport run the whole Nord line, and
`src/lib/clavia/` already carries a full-line model/partition registry
(`docs/NORD-PRODUCT-LINE.md`). **The Nord Stage line is what we cover first** —
Stage 2 / 3 / 4 files read through the shared model-codec seam
(`docs/MULTI-MODEL.md`); other families (Electro, Piano, Lead, …) are mapped and
next. Don't re-frame this as a Stage-4-only tool: Stage 4 is the *most complete*
model, not the only one.

Status: **early — RE done, product being built.** The Stage 4 `.ns4p` format is
decoded and validated (0-mismatch vs ns4decode); Stage 2/3 read via their own
codecs. The Stage 4's **USB transfer protocol is fully reverse-engineered and
hardware-validated** (enumerate/read/write — see `docs/PROTOCOL-RE.md`); a WebUSB
client + a read-only "Check my Nord" probe are built (`src/lib/device/`). The
remaining work is the product layer (Library UI, community library, AI). Read
`docs/ROADMAP.md` for what's built vs. planned.

Web PWA (React 19 + Vite + TypeScript), wrapped to iOS with Capacitor, so one
codebase runs in the browser and as a native app.

## Commands

```bash
npm install        # install deps (a SessionStart hook runs this on Claude Code web)
npm run dev        # Vite dev server
npm test           # vitest run (the parser/decoder tests)
npm run test:watch # vitest watch
npm run typecheck  # tsc --noEmit — CI GATE
npm run lint       # eslint + stylelint — CI GATE (stylelint enforces tokens.css: no hardcoded hex)
npm run lint:fix   # eslint --fix + stylelint --fix
npm run build      # tsc -b && vite build
npm run preview    # preview the production build
npm run fixtures:scan # auto-RE harness over the local (gitignored) corpus
npm run cap:sync   # Capacitor sync (after adding an ios/ platform)
```

**Before committing, run `npm run typecheck`, `npm run lint`, and `npm test`.** CI
(`.github/workflows/ci.yml`) runs `npm ci → typecheck → lint → test` on every PR;
all three are quality gates — keep them green.

Path alias: `@/` → `src/` (see `vite.config.ts`).

## Layout

| Path | What |
|---|---|
| `src/lib/clavia/` | the shared, model-agnostic container: CBIN header (`cbin.ts`), checksum, name/slot/category, `nord-file.ts` identifier, the `ModelCodec` registry (`model.ts`), and the capability/partition tables for the whole Nord line |
| `src/lib/ns4/` | the **Stage 4** `.ns4p` body codec — model + parser + bit/byte decode (the heart), plus the `.nsmp`/`.nsmp4` sample codec family (NW1 + OG encode, WAV import) |
| `src/lib/ns3/`, `src/lib/ns2/` | Stage 3 / Stage 2 body decoders + factory-library catalogs (Tier 2, `docs/MULTI-MODEL.md`) |
| `src/lib/migrate/` | cross-generation program migration (ns2/ns3 → ns4): CommonProgram + advisor seam + ns4 emitter (`docs/MIGRATION.md`) |
| `src/lib/device/` | WebUSB device layer: transport → session → transfer (read/write), backup, hardware probe/report |
| `src/lib/folder/`, `src/lib/library/` | local-folder scan/classify/index → the unified **Library** model |
| `src/lib/ai/` | AI-native search / explanation, provider-pluggable behind an interface |
| `src/lib/midi/` | **live CC/NRPN only** — *not* device transfer (that's vendor USB in `device/`) |
| `src/components/`, `src/routes/` | React UI + TanStack routes (`/library`, `/samples`, `/device`, `/compatibility`, `/dev/inspect`, `/dev/decode`). `DecodeInspector` is the in-browser RE tool (`/dev`) |
| `src/App.tsx`, `src/main.tsx`, `src/router.tsx` | App shell + entry + routes |
| `docs/` | architecture, roadmap, format notes, USB protocol, multi-model, legal stance |
| `scripts/`, `src/dev/` | RE/protocol tooling (`nord*.c`, `crack-checksum.py`) + the dev-only fixtures-corpus Vite plugin |

## How the decode layer works

A file is read **container-first, then body**. `src/lib/clavia/` owns the shared
CBIN envelope (magic/tag/header in `cbin.ts`, checksum, name/slot/category) and the
`ModelCodec` registry (`model.ts`); `parseClaviaFile()` reads the header once and
routes to the codec whose tags/version match — Stage 4 → `ns4/`, Stage 3 → `ns3/`,
Stage 2 → `ns2/`. Dependency direction is **model → clavia**, never the reverse.
Adding a model is one codec entry; the container layer doesn't change. See
`docs/MULTI-MODEL.md`.

### The Stage 4 body codec (`src/lib/ns4/`)

This is where most work happens. The format is *partially* known and is filled
in **incrementally, field by field, each traceable to a source.**

- **`bits.ts`** — low-level bit/byte reading of the Stage 4 program *body* (the
  shared CBIN magic/tag/header now lives in `clavia/cbin.ts`).
- **`maps.ts`** — `buildParamMap()` assembles the `Param[]` map from the generated
  offset/name data. A `Param` has a bit location, a group (`m`/`o`/`p`/`y` =
  master/organ/piano/synth), and layer info.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simonflore/opennord](https://github.com/simonflore/opennord) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
